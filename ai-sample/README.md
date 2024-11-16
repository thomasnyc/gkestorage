<h3>AI storage combine test for inference sample :</h3>


<p>
We are building a single GKE cluster with nodepool for AI use case demoing the combination of storage options together for AI inference case: 
</p>
<ul>

<li>GCSfuse w mem cache  - this handles data from Google Cloud Storage using GCSfuse with MEM cache.  Data like job related data can live on GCS and be able to be leveraged per POD / node on the job bases. Of course, models can be . </li>

<li>Parallelstore - this handles data available required for high speed. Model files can live on Parallelstore and made available to GKE clusters and PODs</li>

<li>2nf boot disk - this handles the LARGE container needed for inference.  </li>
</ul>
<p>
Provided we already setup the following items:
</p>
<ul>

<li>GCS bucket for GCSfuse usage</li>

<li>2nd boot disk image creation with image needed</li>

<li>Parallelstore creation at the region</li>
</ul>
<p>
Create a GKE cluster - with image-streaming , GcsFuseCsiDriver and ParallelstoreCsiDriver
</p>



<pre class="prettyprint">gcloud container clusters create ai-testing-1    --addons GcsFuseCsiDriver     --addons=ParallelstoreCsiDriver    --cluster-version=1.30.5-gke.1443001     --location=us-central1   --workload-pool=thomashk-mig.svc.id.goog  --image-type="COS_CONTAINERD"  --enable-image-streaming  </pre>


<p>
Create a nodepool with scaling capability - w 2nd boot disk using container image cache:
</p>



<pre class="prettyprint">gcloud container node-pools create testpool     --cluster=ai-testing-1    --location=us-central1     --image-type="COS_CONTAINERD"     --enable-image-streaming --secondary-boot-disk=disk-image=projects/thomashk-mig/global/images/test-llama7b,mode=CONTAINER_IMAGE_CACHE  --machine-type=n1-standard-64 --enable-autoscaling --num-nodes 1 --min-nodes 1 --max-nodes 10 </pre>


<p>
Create the credential / service account - this example name = ksa.  This is required by the GCSfuse accessing the Google Cloud Storage bucket.
</p>



<pre class="prettyprint">gcloud container clusters get-credentials ai-testing-1  --location=us-central1
kubectl create serviceaccount ksa</pre>


<p>
Assign the read bucket IAM role to the service account ksa. 
</p>



<pre class="prettyprint">gcloud storage buckets add-iam-policy-binding gs://thomashk-gcsfuse \
    --member "principal://iam.googleapis.com/projects/594050280394/locations/global/workloadIdentityPools/thomashk-mig.svc.id.goog/subject/ns/default/sa/ksa" \
    --role "roles/storage.objectUser"</pre>


<p>
<strong>Check with the storage:</strong>
</p>



<pre class="prettyprint">gcloud beta parallelstore instances list     --project=thomashk-mig     --location=us-central1-a</pre>


<p>
10.113.134.4,10.113.134.2,10.113.134.3
</p>
<p>
Update the <a href="https://github.com/thomasnyc/gkestorage/blob/main/ai-sample/parallelstore-class-pv-pvc.yaml">parallelstore-class-pv-pvc.yaml</a> with the provided IP addresses
</p>
<p>
Create the storage class, persistent volume and persistent volume claim
</p>



<pre class="prettyprint">kubectl create -y parallelstore-class-pv-pvc.yaml</pre>


<p>
Job yaml sample:
</p>



<pre class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: gcs-100-fuse-singlefile
  namespace: ns-fusetest
spec:
  parallelism: 100
  template:
    metadata:
      annotations:
        gke-gcsfuse/volumes: "true"
        gke-gcsfuse/memory-request: 50Gi
        gke-parallelstore/volumes: "true"
        gke-parallelstore/cpu-request: 500m
        gke-parallelstore/memory-request: 1Gi
        gke-parallelstore/ephemeral-storage-request: 5Gi
        gke-parallelstore/ephemeral-storage-limit: 10Gi
    spec:
      containers:
      - name: largeimagetest
        image: us-central1-docker.pkg.dev/thomashk-mig/thomashk-aitest/llama7bmodel
        resources:
          limits:
            cpu: 1000m
            memory: 512Mi
          requests:
            cpu: 1000m
            memory: 512Mi
        command:
          - "/bin/sh"
          - "-c"
          - |
            echo "first read with cache missing"
            time cat /data/sample_data/file_2*  > /dev/null

            echo "2nd read with caahe"
            time cat /data/sample_data/file_2* > /dev/null

            echo "cp file to local"
            time cp /data/sample_data/file_2* /tmp/.
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
        volumeMounts:
        - name: gcs-fuse-csi-ephemeral
          mountPath: /data
          readOnly: true
        - mountPath: "/data"
          name: parallelstore-pv 
      serviceAccountName: ksa
      Volumes:
      - name: parallelstore-pv 
        persistentVolumeClaim:
          claimName: parallelstore-pvc
      - name: gcs-fuse-csi-ephemeral
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse
            mountOptions: "implicit-dirs"
            fileCacheCapacity: "30Gi"
      restartPolicy: Never
  backoffLimit: 1</pre>


<p>
Run the sample job: 
</p>



<pre class="prettyprint">kubectl create -f ai-serving-sample.yaml </pre>


<p>
Testing: 
</p>
<p>
(base) thomashk_thomashk_joonix_net@deployment-8:~/gke/parallelstore/gke-parallelstore$ kubectl exec --stdin --tty gcscombine1-9vcx5 -- bash
</p>
<p>
Defaulted container "largeimagetest" out of: largeimagetest, gke-parallelstore-sidecar (init), gke-gcsfuse-sidecar (init)
</p>
<p>
root@gcscombine1-9vcx5:/app# df -h
</p>
<p>
Filesystem                                                              Size  Used Avail Use% Mounted on
</p>
<p>
overlay                                                                  95G  4.6G   90G   5% /
</p>
<p>
tmpfs                                                                    64M     0   64M   0% /dev
</p>
<p>
thomashk-gcsfuse                                                        1.0P     0  1.0P   0% /data
</p>
<p>
thomashk-mig/us-central1/parallelstore1/default-pool/default-container   18T  100G   18T   1% /models
</p>
<p>
/dev/sda1                                                                95G  4.6G   90G   5% /etc/hosts
</p>
<p>
shm                                                                      64M     0   64M   0% /dev/shm
</p>
<p>
tmpfs                                                                    12G   12K   12G   1% /run/secrets/kubernetes.io/serviceaccount
</p>
<p>
tmpfs                                                                   118G     0  118G   0% /proc/acpi
</p>
<p>
tmpfs                                                                   118G     0  118G   0% /proc/scsi
</p>
<p>
tmpfs                                                                   118G     0  118G   0% /sys/firmware
</p>
<p>
root@gcscombine1-9vcx5:/app# 
</p>
<p>
