# gkestorage

<!-- You have some errors, warnings, or alerts. If you are using reckless mode, turn it off to see inline alerts.
* ERRORs: 0
* WARNINGs: 0
* ALERTS: 1 -->
<p>
<a href="http://go/pso-storage-sample-code">go/pso-storage-sample-code</a>
</p>
<h2>Storage Technologies sample command(s) or code around AI infrastructure
service</h2>
<h3></h3>
<h3>Google Cloud Storage </h3>
<h4>Command - gsutil </h4>
<p>
Copy a single 12GB file (using e2-medium instance)
</p>


<pre
class="prettyprint">gsutil cp gs://thomashk-gcsfuse/sample_data/file_2.txt .</pre>
<p>
thomashk_thomashk_joonix_net@deployment-8:/mnt/disks/ml-data/gcloudstorage$ time
gsutil cp gs://thomashk-gcsfuse/sample_data/file_2.txt .
</p>
<p>
Copying gs://thomashk-gcsfuse/sample_data/file_2.txt...
</p>
<p>
| [1 files][ 12.2 GiB/ 12.2 GiB]   26.0 MiB/s
</p>
<p>
Operation completed over 1 objects/12.2 GiB.
</p>
<p>
real    6m39.360s
</p>
<p>
user    5m52.924s
</p>
<p>
sys     3m54.710s
</p>
<p>
Result - ~50MiB/s
</p>
<h4>Command - gcloud storage cp</h4>
<p>
Copy a single 12GB file (using e2-medium instance)
</p>


<pre
class="prettyprint">time gcloud storage cp gs://thomashk-gcsfuse/sample_data/file_2.txt .</pre>
<p>
thomashk_thomashk_joonix_net@deployment-8:/mnt/disks/ml-data/gcloudstorage$ time
gcloud storage cp gs://thomashk-gcsfuse/sample_data/file_2.txt .
</p>
<p>
Copying gs://thomashk-gcsfuse/sample_data/file_2.txt to file://./file_2.txt
</p>
<p>
  Completed files 1/1 | 12.2GiB/12.2GiB | 191.9MiB/s
</p>
<p>
Average throughput: 203.1MiB/s
</p>
<p>
real    1m4.413s
</p>
<p>
user    0m52.969s
</p>
<p>
sys     0m47.303s
</p>
<p>
Result - ~203.1MiB/s
</p>
<h3></h3>
<h3>Google Cloud Storage (GCS) Fuse </h3>
<p>
<a
href="https://github.com/GoogleCloudPlatform/gcsfuse">https://github.com/GoogleCloudPlatform/gcsfuse</a>
</p>
<p>
<a
href="https://cloud.google.com/storage/docs/gcs-fuse">https://cloud.google.com/storage/docs/gcs-fuse</a>
</p>
<p>
Use case:
</p>
<p>
Architecture - We suggest using GCS as the source of truth. It is the most
secure, affordable, scalable with high availability and reliability overall. At
the same time, it is the storage solution that allows Tb level of bandwidth and
high IOPS.
</p>
<h4>GKE - cluster setup with GCSfuse</h4>
<p>
<a
href="https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver">https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver</a>
</p>
<ol>
<li>GKE Cluster creation using GCSFuse: 


<pre
class="prettyprint">gcloud container clusters create gcsfuse-test1     --addons GcsFuseCsiDriver     --cluster-version=1.29.7-gke.1008000     --location=us-central1        --machine-type=n1-standard-96 --enable-autoscaling --num-nodes 3 --min-nodes 3 --max-nodes 9 --workload-pool=thomashk-mig.svc.id.goog</pre>
</li>
<li>Set up the credentials: 


<pre
class="prettyprint">gcloud container clusters get-credentials gcsfuse-test1 \
    --location=us-central1</pre>
</li>
</ol>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ gcloud container clusters
get-credentials gcsfuse-test \
</p>
<p>
    --location=us-central1
</p>
<p>
Fetching cluster endpoint and auth data.
</p>
<p>
kubeconfig entry generated for gcsfuse-test
</p>
<ol>
<li>Create namespace: 


<pre
class="prettyprint">kubectl create namespace ns-fusetest</pre>
</li>
</ol>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ kubectl create namespace
ns-fusetest
</p>
<p>
namespace/ns-fusetest created
</p>
<ol>
<li>Create service account ksa: 


<pre
class="prettyprint">kubectl create serviceaccount ksa \
    --namespace ns-fusetest</pre>
</li>
</ol>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ kubectl create serviceaccount ksa \
</p>
<p>
    --namespace ns-fusetest
</p>
<p>
serviceaccount/ksa created
</p>
<ol>
<li>Assign IAM permission: 


<pre
class="prettyprint">gcloud storage buckets add-iam-policy-binding gs://thomashk-gcsfuse \
    --member "principal://iam.googleapis.com/projects/594050280394/locations/global/workloadIdentityPools/thomashk-mig.svc.id.goog/subject/ns/ns-fusetest/sa/ksa" \
    --role "roles/storage.objectUser"</pre>
</li>
</ol>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ gcloud storage buckets
add-iam-policy-binding gs://thomashk-gcsfuse \
</p>
<p>
    --member
"principal://iam.googleapis.com/projects/594050280394/locations/global/workloadIdentityPools/thomashk-mig.svc.id.goog/subject/ns/ns-fusetest/sa/ksa"
\
</p>
<p>
    --role "roles/storage.objectUser"
</p>
<p>
bindings:
</p>
<p>
- members:
</p>
<p>
  - projectEditor:thomashk-mig
</p>
<p>
  - projectOwner:thomashk-mig
</p>
<p>
  role: roles/storage.legacyBucketOwner
</p>
<p>
- members:
</p>
<p>
  - projectViewer:thomashk-mig
</p>
<p>
  role: roles/storage.legacyBucketReader
</p>
<p>
- members:
</p>
<p>
  - projectEditor:thomashk-mig
</p>
<p>
  - projectOwner:thomashk-mig
</p>
<p>
  role: roles/storage.legacyObjectOwner
</p>
<p>
<img src="images/image1.png" width="" alt="alt_text" title="image_tooltip">
</p>
<h4>GKE - GCSFuse with local SSD cache</h4>
<p>
GKE Cluster creation for GCSFuse with local ssd cache:
</p>


<pre
class="prettyprint">gcloud container clusters create gcsfuse-test1     --addons GcsFuseCsiDriver     --cluster-version=1.29.7-gke.1008000     --location=us-central1     --ephemeral-storage-local-ssd count=8     --machine-type=n1-standard-96 --enable-autoscaling --num-nodes 3 --min-nodes 3 --max-nodes 9 --workload-pool=thomashk-mig.svc.id.goog</pre>
<p>
Make sure there are local ssd as ephemeral storage with the container cluster
</p>
<p>
Job yaml sample:
</p>


<pre
class="prettyprint">apiVersion: batch/v1
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
        gke-gcsfuse/ephemeral-storage-request: 50Gi
    spec:
      containers:
      - name: reader
        image: busybox
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
      serviceAccountName: ksa
      volumes:
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
</p>
<h4>GKE - GCSFuse with Memory cache</h4>
<p>
GKE Cluster creation for GCSFuse:
</p>


<pre
class="prettyprint">gcloud container clusters create gcsfuse-test1     --addons GcsFuseCsiDriver     --cluster-version=1.30.5-gke.1014001     --location=us-central1         --machine-type=n1-standard-96 --enable-autoscaling --num-nodes 3 --min-nodes 3 --max-nodes 9 --workload-pool=thomashk-mig.svc.id.goog</pre>
<p>
Make sure there is enough memory as cache.
</p>
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
    spec:
      containers:
      - name: reader
        image: busybox
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
            echo "2nd read with cache"
            time cat /data/sample_data/file_2* > /dev/null
            echo "cp file to local"
            time cp /data/sample_data/file_2* /tmp/.
        volumeMounts:
        - name: gcs-fuse-csi-ephemeral
          mountPath: /data
          readOnly: true
      serviceAccountName: ksa
      volumes:
      - name: gke-gcsfuse-cache
        emptyDir:
          medium: Memory
      - name: gcs-fuse-csi-ephemeral
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "30Gi"
      restartPolicy: Never
  backoffLimit: 1</pre>
<h4></h4>
<h4>GKE - Using Multiple buckets within a single POD:</h4>
<p>
In this example - we used a memory cache as well. But memory cache is NOT a
requirement.
</p>
<p>
Job yaml sample:
</p>


<pre class="prettyprint">kind: Job
metadata:
  name: gcs-fuse-2-bucket
  namespace: ns-fusetest
spec:
  parallelism: 1
  template:
    metadata:
      annotations:
        gke-gcsfuse/volumes: "true"
        gke-gcsfuse/memory-request: 50Gi
    spec:
      containers:
      - name: reader
        image: busybox
        resources:
          limits:
            memory: 120Gi
          requests:
            cpu: 1000m
            memory: 120Gi
        command:
          - "/bin/sh"
          - "-c"
          - |
            echo "first read with cache missing"
            time cat /data/dummy_files/*  > /dev/null
            echo "finished bucket1 first read"
            echo "2nd read with cache"
            time cat /data/dummy_files/* > /dev/null

            echo "first read with cache missing"
            time cat /data2/dummy_files/*  > /dev/null
            echo "finished bucket1 first read"
            echo "2nd read with cache"
            time cat /data2/dummy_files/* > /dev/null
            echo "finished bucket1 first read"
           
        volumeMounts:
        - name: gcs-fuse-csi-ephemeral-1
          mountPath: /data
          readOnly: true
        - name: gcs-fuse-csi-ephemeral-2
          mountPath: /data2
          readOnly: true
      serviceAccountName: ksa
      volumes:
      - name: gke-gcsfuse-cache
        emptyDir:
          medium: Memory
      - name: gcs-fuse-csi-ephemeral-1
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "50Gi"
      - name: gcs-fuse-csi-ephemeral-2
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse2
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "50Gi"
      restartPolicy: Never
  backoffLimit: 1</pre>
<p>
</p>
<p>
2nd version: using the node with TPU for testing :
</p>


<pre class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: gcs-fuse-2-bucket
  namespace: ns-fusetest
spec:
  parallelism: 1
  template:
    metadata:
      annotations:
        gke-gcsfuse/volumes: "true"
        gke-gcsfuse/memory-request: 100Gi
        gke-gcsfuse/memory-limit: 120Gi
    spec:
      nodeSelector:
        cloud.google.com/gke-tpu-accelerator: tpu-v5-lite-podslice
        cloud.google.com/gke-tpu-topology: 2x2
      containers:
      - name: reader
        image: busybox
        resources:
          limits:
            memory: 90Gi
            google.com/tpu: 4
          requests:
            cpu: 2000m
            memory: 10Gi
            google.com/tpu: 4
        command:
          - "/bin/sh"
          - "-c"
          - |
            echo "first bucket read with cache missing"
            time cat /data/dummy_files/*  > /dev/null
            echo "first bucket read completed"
            echo "first bucket 2nd read with cache"
            time cat /data/dummy_files/* > /dev/null
            echo "first bucket read with cache 2nd read finished"
            echo "2nd bucket first read with cache missing"
            time cat /data2/dummy_file1/*  > /dev/null
            echo "2nd bucket read completed"
            echo "2nd bucket 2nd read with cache"
            time cat /data2/dummy_file1/* > /dev/null
            echo "2nd bucket read with cache 2nd read finished"
        volumeMounts:
        - name: gcs-fuse-csi-ephemeral-1
          mountPath: /data
          readOnly: true
        - name: gcs-fuse-csi-ephemeral-2
          mountPath: /data2
          readOnly: true
      serviceAccountName: ksa
      volumes:
      - name: gke-gcsfuse-cache
        emptyDir:
          medium: Memory
      - name: gcs-fuse-csi-ephemeral-1
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "50Gi"
      - name: gcs-fuse-csi-ephemeral-2
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse2
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "50Gi"
      restartPolicy: Never
  backoffLimit: 1</pre>
<p>
Couple important point:
</p>
<ol>
<li><code>gke-gcsfuse/memory-limit: must be larger than the sum of
fileCacheCapacity add up. If they are the same or equal. Error occurr:
</code>


<pre class="prettyprint">
OOMKilled / Transport endpoint is not connected. 
</pre>
</li>
</ol>
<p>
FYI - this is the command creating the TPU nodepool:
</p>


<pre
class="prettyprint">gcloud container node-pools create tpu1-pool     --location=us-west1-a     --cluster=tpuv5-cluster2     --node-locations=us-west1-c     --machine-type=ct5lp-hightpu-4t     --num-nodes=1   --tpu-topology=2x2</pre>
<h3></h3>
<h3>Using 2nd boot disk with “HUGE” container(s)</h3>
<p>
Use case:
</p>
<p>
Reference:
</p>
<p>
<a
href="https://github.com/GoogleCloudPlatform/ai-on-gke/blob/2c2fec4067e1d7cc7acd01f99ccef85fe1240ea7/tools/gke-disk-image-builder/README.md?plain=1#L2">https://github.com/GoogleCloudPlatform/ai-on-gke/blob/2c2fec4067e1d7cc7acd01f99ccef85fe1240ea7/tools/gke-disk-image-builder/README.md?plain=1#L2</a>
</p>
<p>
With a 29GB image at the GCP Artifact Registry:
</p>
<p>
Docker Image:
</p>
<p>
IMAGE ID       CREATED             SIZE
</p>
<p>
llama-2-7b-chat-hf                                                     latest
d7d9d7f54433   About an hour ago   29.6GB
</p>
<p>
Tag the image and move the image to the GCP Artifact Registry:
</p>


<pre
class="prettyprint">sudo docker tag llama-2-7b-chat-hf us-central1-docker.pkg.dev/thomashk-mig/thomashk-aitest/llama7bmodel</pre>
<p>
Optional - Install go
</p>


<pre
class="prettyprint">rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz</pre>
<p>
Do this as Root.
</p>
<p>
As user: export PATH=$PATH:/usr/local/go/bin
</p>
<p>
Gives service account access right to the Artifact Registry
</p>


<pre
class="prettyprint">gcloud artifacts repositories add-iam-policy-binding thomashk-aitest \
  --location=us-central1 \
  --member=serviceAccount:594050280394-compute@developer.gserviceaccount.com \
  --role="roles/artifactregistry.reader"</pre>
<p>
<a
href="https://github.com/GoogleCloudPlatform/ai-on-gke/tree/main/tools/gke-disk-image-builder">https://github.com/GoogleCloudPlatform/ai-on-gke/tree/main/tools/gke-disk-image-builder</a>
</p>
<p>
Use the provided gke-disk-image-builder script
</p>


<pre
class="prettyprint">go run ./cli     --project-name=thomashk-mig     --image-name=test-llama7b     --zone=us-central1-a     --gcs-path=gs://thomashk-ml-test/2nddisk     --disk-size-gb=80  --image-pull-auth=ServiceAccountToken   --container-image=us-central1-docker.pkg.dev/thomashk-mig/thomashk-aitest/llama7bmodel:latest</pre>
<p>
Make sure you use the complete container address:
</p>
<p>
Missing :latest will not work.:
--container-image=us-central1-docker.pkg.dev/thomashk-mig/thomashk-aitest/llama7bmodel
</p>
<p>
Must use --image-pull-auth=ServiceAccountToken if pulling from Artifact Registry
</p>
<p>
Output:
</p>
<p>
Image has successfully been created at:
projects/thomashk-mig/global/images/test-llama7b
</p>
<p>
Create the cluster
</p>


<pre
class="prettyprint">
gcloud container clusters create test-2nd-cluster \
    --zone=us-central1-a \
    --image-type="COS_CONTAINERD" \
    --enable-image-streaming
</pre>
<p>
Must turn on the “enable-image-streaming”
</p>
<p>
When creating the nodepool. Need to make sure there is 2nd boot disk option:
</p>


<pre
class="prettyprint">gcloud container node-pools create test2nddisknodepool2     --cluster=test-2nd-cluster     --zone=us-central1-a     --image-type="COS_CONTAINERD"     --enable-image-streaming --secondary-boot-disk=disk-image=projects/thomashk-mig/global/images/test-llama7b,mode=CONTAINER_IMAGE_CACHE
</pre>
<p>
Job:
</p>


<pre
class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: largecontainertest10
spec:
  parallelism: 10
  template:
    metadata:
      annotations:
    spec:
      containers:
      - name: largeimagetest
        image: us-central1-docker.pkg.dev/thomashk-mig/thomashk-aitest/llama7bmodel
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
        command:
          - "/bin/sh"
          - "-c"
          - |  
            echo "test on the image "
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
      restartPolicy: Never
  backoffLimit: 1
</pre>
<p>
Job completed in 4 sec means that it is not pulling anything from the registry
</p>
<p>
Start time
</p>
<p>
Sep 5, 2024, 9:38:44 AM
</p>
<p>
Completion time
</p>
<p>
Sep 5, 2024, 9:39:48 AM
</p>
<h3></h3>
<h3>Filestore - CSI driver example - using Persistent Volume and PVC </h3>
<p>
Prep create filestore:  10.7.183.186:/nfsshare
</p>
<p>
GKE cluster name: tpuv5-cluster
</p>
<p>
Region: us-west1-a
</p>
<p>
Enable the Filestore CSI driver with the existing GKE cluster:
</p>


<pre
class="prettyprint">gcloud container clusters update tpuv5-cluster    --update-addons=GcpFilestoreCsiDriver=ENABLED  --region=us-west1-a</pre>
<p>
Sample output:
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ gcloud container clusters update
tpuv5-cluster    --update-addons=GcpFilestoreCsiDriver=ENABLED
--region=us-west1-a
</p>
<p>
Updating tpuv5-cluster...done.
</p>
<p>
Updated
[https://container.googleapis.com/v1/projects/thomashk-mig/zones/us-west1-a/clusters/tpuv5-cluster].
</p>
<p>
To inspect the contents of your cluster, go to:
https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/tpuv5-cluster?project=thomashk-mig
</p>
<p>
Setup the PresistentVolume and the PresistentVolumeClaim with yaml -
filestorecsi.yaml
</p>


<pre
class="prettyprint">
apiVersion: v1
kind: PersistentVolume
metadata:
  name: filestorenfs
spec:
  storageClassName: ""
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: "modeInstance/us-west1/gketest/nfsshare"
    volumeAttributes:
      ip: 10.7.183.186:
      volume: nfsshare
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: podpvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  volumeName: filestorenfs
  resources:
    requests:
      storage: 1Ti
</pre>
<p>
Sample output:
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~/gke/tpu/filestorecsi$ kubectl create
-f filestorecsi.yaml
</p>
<p>
persistentvolume/filestorenfs created
</p>
<p>
persistentvolumeclaim/podpvc created
</p>
<p>
Test with pod: testpod.yaml using the presistentVolumeClaim
</p>


<pre class="prettyprint">apiVersion: v1
kind: Pod
metadata:
  name: interactive-test-pod
spec:
  containers:
  - name: test-container
    image: ubuntu:latest # Or any other image you prefer
    command: ["sleep", "infinity"] # Keep the pod running
    tty: true # Enable interactive terminal
    stdin: true # Allow input to the container
    volumeMounts:
    - name: my-volume
      mountPath: /home
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: podpv</pre>
<p>
To create a pod for interactive use:
</p>


<pre class="prettyprint">kubectl create -f testpod.yaml</pre>
<p>
Interactive Command:
</p>


<pre
class="prettyprint">kubectl exec --stdin --tty interactive-test-pod -- bash</pre>
<p>
thomashk_thomashk_joonix_net@deployment-8:~/gke/tpu/filestorecsi$ kubectl exec
--stdin --tty interactive-test-pod -- bash
</p>
<p>
root@interactive-test-pod:/# df -h
</p>
<p>
Filesystem              Size  Used Avail Use% Mounted on
</p>
<p>
overlay                  95G  8.7G   86G  10% /
</p>
<p>
tmpfs                    64M     0   64M   0% /dev
</p>
<p>
<strong>10.7.183.186:/nfsshare 1007G  251M  956G   1% /home</strong>
</p>
<p>
/dev/sda1                95G  8.7G   86G  10% /etc/hosts
</p>
<p>
shm                      64M     0   64M   0% /dev/shm
</p>
<p>
tmpfs                   2.8G   12K  2.8G   1%
/run/secrets/kubernetes.io/serviceaccount
</p>
<p>
tmpfs                   2.0G     0  2.0G   0% /proc/acpi
</p>
<p>
tmpfs                   2.0G     0  2.0G   0% /proc/scsi
</p>
<p>
tmpfs                   2.0G     0  2.0G   0% /sys/firmware
</p>
<p>
root@interactive-test-pod:/# cd /home
</p>
<p>
root@interactive-test-pod:/home# ls
</p>
<p>
home  lost+found
</p>
<p>
##create the environment with poetry
</p>
<p>
root@interactive-test-pod:/home# .
/home/home/thomashk_thomashk_joonix_net/.cache/pypoetry/virtual
</p>
<p>
envs/poetry-demo-H1a7U4EZ-py3.11/bin/activate
</p>
<p>
(poetry-demo-py3.11) root@interactive-test-pod:/home# python -c "import numpy;
print(numpy.version.version)"
</p>
<p>
</p>
<h3>Hyperdisk ML - CSI driver example - using Persistent Volume and PVC</h3>
<p>
Reference:<a
href="https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/hyperdisk-ml#model-access">https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/hyperdisk-ml#model-access</a>
</p>
<p>
Prepare for the Hyperdisk ML - Have a 1TB dataset VM image and create the
readonly hyperdisk ML disk . This is done in the background.
</p>
<h4>(optional) Single node mounting </h4>
<p>
Using a VM to validate the data:
</p>


<pre
class="prettyprint">gcloud compute instances create hyperdisktestvm1 \
   --project=thomashk-mig \
   --zone=us-central1-a \
   --machine-type=c3d-standard-16 \ --network-interface=network-tier=PREMIUM,nic-type=GVNIC,stack-type=IPV4_ONLY,subnet=default \
   --metadata=enable-oslogin=true \
   --maintenance-policy=MIGRATE \
   --provisioning-model=STANDARD \
   --service-account=594050280394-compute@developer.gserviceaccount.com \
   --scopes=https://www.googleapis.com/auth/cloud-platform \
--create-disk=auto-delete=yes,boot=yes,device-name=hyperdisktestvm1,image=projects/debian-cloud/global/images/debian-12-bookworm-v20240910,mode=rw,size=10,type=pd-balanced \
   --disk=boot=no,device-name=hyperdisktest1,mode=ro,name=hyperdisktest1 \
   --no-shielded-secure-boot \
   --shielded-vtpm \
   --shielded-integrity-monitoring \
   --labels=goog-ec-src=vm_add-gcloud \
   --reservation-affinity=any</pre>
<p>
Mounting command:
</p>


<pre
class="prettyprint">sudo mkdir /mnt/disk
sudo mount -o noload,ro /dev/nvme0n2 /mnt/disk</pre>
<h4>GKE setup and testing:</h4>
<p>
Create a new cluster -enable-autoscaling
</p>


<pre
class="prettyprint">gcloud container clusters create hyperd-test1  --addons=GcePersistentDiskCsiDriver --cluster-version=1.30.3-gke.1969001     --location=us-central1       --machine-type=c3d-standard-16 --enable-autoscaling --num-nodes 1 --min-nodes 1 --max-nodes 12 --workload-pool=thomashk-mig.svc.id.goog </pre>
<p>
Hyper disk ML  supported VM families are: A2, A3, G2, C3 and C3d . using C3D
</p>
<p>
Cluster needs GKE Version 1.30.2 or above
</p>
<p>
Create a new PersistentVolume and PersistentVolumeClaim:
</p>


<pre class="prettyprint">apiVersion: v1
kind: PersistentVolume
metadata:
  name: hdml-static-pv
spec:
  storageClassName: "hyperdisk-ml"
  capacity:
    storage: 1024Gi
  accessModes:
    - ReadOnlyMany
  claimRef:
    namespace: default
    name: hdml-static-pvc
  csi:
    driver: pd.csi.storage.gke.io
    volumeHandle: projects/thomashk-mig/zones/us-central1-a/disks/hyperdisktest1
    fsType: ext4
    readOnly: true
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: topology.gke.io/zone
          operator: In
          values:
          - us-central1-a
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  namespace: default
  name: hdml-static-pvc
spec:
  storageClassName: "hyperdisk-ml"
  volumeName: hdml-static-pv
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 1024Gi</pre>



<pre
class="prettyprint">kubectl create -f hyperdiskpvpvc.yaml</pre>
<p>
thomashk_thomashk_joonix_net@deployment-8:~/gke/hyperdisk$ kubectl create -f
hyperdiskpvpvc.yaml
</p>
<p>
persistentvolume/hdml-static-pv created
</p>
<p>
persistentvolumeclaim/hdml-static-pvc created
</p>
<p>
Test jobs - Job yaml sample:
</p>


<pre class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: hyperdisk-test-job
spec:
  parallelism: 100
  template:
    metadata:
      annotations:
    spec:
      containers:
      - name: test-container
        image: busybox
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
            echo "hyperdisk validation - ls"
            ls /hyperdiskmount
            echo "hyperdisk - cat files"
            time cat /hyperdiskmount/spilta* > /dev/null
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
        volumeMounts:
          - name: hyperdisk-volume
            mountPath: /hyperdiskmount
            readOnly: true  
      volumes:
        - name: hyperdisk-volume
          persistentVolumeClaim:
            claimName: hdml-static-pvc
      restartPolicy: Never
  backoffLimit: 1</pre>
<p>
Sample / initial test:  Not official test result to be shared.
<br>Reading 26 files with 5GB each = total 130GB:
</p>
<p>
Deployed the 100 concurrent jobs over 12 GKE nodes .
</p>
<p>
Result: Run 100 concurrent pods - avg: 22 - 23 mins to cat the 130GB data to
/dev/null
</p>
<ul>
<li>Use the basic bandwidth. </li>
</ul>
<p>
Single node - singe cat command:
</p>
<p>
Result: Run 1 read from single VM - 7 mins to cat the 130GB data to /dev/null
</p>
<p>
</p>
<h3>ParallelStore - CSI driver example - using Existing Parallelstore on
GKE</h3>
<p>
CSI driver Reference <a
href="https://cloud.google.com/kubernetes-engine/docs/concepts/parallelstore-for-gke">https://cloud.google.com/kubernetes-engine/docs/concepts/parallelstore-for-gke</a>
</p>
<p>
Parallelstore Reference:
</p>
<p>
Creation of new parallelstore - this is a common use case with static storage /
existing volume.
</p>
<p>
Need to get whitelisted for both:
</p>
<ul>
<li>Use of Parallelstore</li>
<li>Use of ParallelstoreCSIdriver</li>
</ul>
<p>
Do this to enable the beta features:
</p>


<pre class="prettyprint">sudo gcloud components update</pre>
<p>
Create a new VPC peering
</p>


<pre
class="prettyprint">gcloud compute addresses create pstoreaddress \
  --global \
  --purpose=VPC_PEERING \
  --prefix-length=24 \
  --description="Parallelstore VPC Peering" \
  --network=default</pre>
<p>
64 address shall be reserved for every Parallelstore instance.
</p>
<p>
Output:
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ gcloud compute addresses create
pstoreaddress \
</p>
<p>
  --global \
</p>
<p>
  --purpose=VPC_PEERING \
</p>
<p>
  --prefix-length=24 \
</p>
<p>
  --description="Parallelstore VPC Peering" \
</p>
<p>
  --network=default
</p>
<p>
Created
[https://www.googleapis.com/compute/v1/projects/thomashk-mig/global/addresses/pstoreaddress].
</p>
<p>
Get the CIDR range:
</p>


<pre
class="prettyprint">CIDR_RANGE=$(
  gcloud compute addresses describe pstoreaddress \
    --global  \
    --format="value[separator=/](address, prefixLength)"
)</pre>
<p>
Output:
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ CIDR_RANGE=$(
</p>
<p>
  gcloud compute addresses describe pstoreaddress \
</p>
<p>
    --global  \
</p>
<p>
    --format="value[separator=/](address, prefixLength)"
</p>
<p>
)
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ echo $CIDR_RANGE
</p>
<p>
10.113.134.0/24
</p>
<p>
Creating the firewall:
</p>


<pre
class="prettyprint"> gcloud compute firewall-rules create pstore-rule \
  --allow=tcp \
  --network=default \
  --source-ranges=$CIDR_RANGE</pre>
<p>
Output:
</p>
<p>
thomashk_thomashk_joonix_net@deployment-8:~$ gcloud compute firewall-rules
create pstore-rule \
</p>
<p>
  --allow=tcp \
</p>
<p>
  --network=default \
</p>
<p>
  --source-ranges=$CIDR_RANGE
</p>
<p>
Creating firewall...⠹Created
[https://www.googleapis.com/compute/v1/projects/thomashk-mig/global/firewalls/pstore-rule].
</p>
<p>
Creating firewall...done.
</p>
<p>
NAME         NETWORK  DIRECTION  PRIORITY  ALLOW  DENY  DISABLED
</p>
<p>
pstore-rule  default  INGRESS    1000      tcp          False
</p>
<p>
Vpc-peering connection:
</p>


<pre
class="prettyprint">gcloud services vpc-peerings connect \
  --network=default \
  --ranges=pstoreaddress \
  --service=servicenetworking.googleapis.com</pre>
<p>
Output:
</p>
<p>
ashk_thomashk_joonix_net@deployment-8:~$ gcloud services vpc-peerings connect \
</p>
<p>
  --network=default \
</p>
<p>
  --ranges=pstoreaddress \
</p>
<p>
  --service=servicenetworking.googleapis.com
</p>
<p>
</p>
<p>
Operation
"operations/pssn.p24-594050280394-9befaf46-7e3c-479c-b8e2-392f4818c097" finished
successfully.
</p>
<p>
Creation of Parallelstore instance.
</p>


<pre
class="prettyprint">gcloud beta parallelstore instances create parallelstore1 \
  --capacity-gib=12000 \
  --location=us-central1-a \
  --network=default \
  --project=thomashk-mig \
  --directory-stripe-level=directory-stripe-level-balanced \
  --file-stripe-level=file-stripe-level-balanced</pre>
<p>
<strong>Check with the storage:</strong>
</p>


<pre
class="prettyprint">gcloud beta parallelstore instances list     --project=thomashk-mig     --location=us-central1-a</pre>
<p>
thomashk_thomashk_joonix_net@deployment-8:~/gke/parallelstore$ gcloud beta
parallelstore instances list     --project=thomashk-mig
--location=us-central1-a
</p>
<p>
NAME                                                                    Capacity
 DESCRIPTION  CREATE_TIME                     UPDATE_TIME
STATE   NETWORK                                        RESERVED_IP_RANGE
ACCESS_POINTS
</p>
<p>
projects/thomashk-mig/locations/us-central1-a/instances/parallelstore1  12000
              2024-10-07T20:30:30.698217424Z  2024-10-07T20:40:40.596792117Z
ACTIVE  projects/thomashk-mig/global/networks/default
10.113.134.4,10.113.134.3,10.113.134.2
</p>
<p>
Creation of GKE:
</p>


<pre
class="prettyprint">gcloud container clusters create parallelst-test1 --cluster-version=1.30.3-gke.1969001     --location=us-central1       --machine-type=c3d-standard-8   --addons=ParallelstoreCsiDriver  --enable-autoscaling --num-nodes 1 --min-nodes 1 --max-nodes 25 --workload-pool=thomashk-mig.svc.id.goog </pre>
<p>
Storageclass creation:
</p>


<pre
class="prettyprint">apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: parallelstore-class
provisioner: parallelstore.csi.storage.gke.io
volumeBindingMode: Immediate
reclaimPolicy: Delete
allowedTopologies:
- matchLabelExpressions:
  - key: topology.gke.io/zone
    values:
    - us-central1-a
</pre>
<p>
PersistentVolume creation:
</p>


<pre
class="prettyprint">apiVersion: v1
kind: PersistentVolume
metadata:
  name: parallelstore-pv
spec:
  storageClassName: parallelstore-class
  capacity:
    storage: 12000Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  csi:
    driver: parallelstore.csi.storage.gke.io
    volumeHandle: "thomashk-mig/us-central1/parallelstore1/default-pool/default-container"
    volumeAttributes:
      accessPoints: 10.113.134.4,10.113.134.3,10.113.134.2
      network: default
</pre>
<p>
Be Careful with the volume handle:
</p>
<p>
<strong>"thomashk-mig/locations/us-central1/instances/parallelstore1/default-pool/default-container"</strong>
</p>
<p>
It should be
<strong>"thomashk-mig/us-central1/parallelstore1/default-pool/default-container"</strong>
</p>
<p>
PersistentVolumeClaim creation:
</p>


<pre
class="prettyprint">apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: parallelstore-pvc
spec:
  accessModes:
  - ReadWriteMany
  volumeName: parallelstore-pv
  resources:
    requests:
      storage: 12000Gi # Range from 12,000Gi to 100,000Gi, must be in multiples of 4,000 GiB
  storageClassName: parallelstore-class
</pre>
<p>
Run FIO job:
</p>
<p>
Creating the secret before running the download pod:
</p>


<pre
class="prettyprint">kubectl create secret generic hf-secret \
    --from-literal=hf_api_token=hf_svFxxxxxxxxxxxxxxxxxxxxxxj\
    --dry-run=client -o yaml | kubectl apply -f -</pre>
<p>
secret/hf-secret created
</p>
<p>
Download dataset test job with GKE :  This is a huggingface job where you must
store the huggingface key to the gke secret first.
</p>


<pre
class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: downloader-job1
spec:
  template:  # Template for the Pods the Job will create
    metadata:
      annotations:
        gke-parallelstore/volumes: "true"
        gke-parallelstore/cpu-request: 500m
        gke-parallelstore/memory-request: 1Gi
        gke-parallelstore/ephemeral-storage-request: 5Gi
        gke-parallelstore/ephemeral-storage-limit: 10Gi
    spec:
      containers:
      - name: copy
        resources:
          requests:
            cpu: "6"
          limits:
            cpu: "6"
        image: huggingface/downloader:0.17.3
        command: [ "huggingface-cli" ]
        args:
        - download
        - google/gemma-1.1-7b-it
        - --local-dir=/data/gemma-7b
        - --local-dir-use-symlinks=False
        env:
        - name: HUGGING_FACE_HUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: hf-secret
              key: hf_api_token
        volumeMounts:
        - mountPath: "/data"
          name: parallelstore-pv 
      restartPolicy: Never
      volumes:
      - name: parallelstore-pv 
        persistentVolumeClaim:
          claimName: parallelstore-pvc
  parallelism: 1         # Run 1 Pods concurrently
  completions: 1         # Once 1 Pods complete successfully, the Job is done
  backoffLimit: 1        # Max retries on failure
</pre>
<p>
Then run the FIO read test pod:
</p>


<pre
class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: 100-benchmark-job
spec:
  parallelism: 100
  template:  # Template for the Pods the Job will create
    metadata:
      annotations:
        gke-parallelstore/volumes: "true"
        gke-parallelstore/cpu-request: 500m
        gke-parallelstore/memory-request: 1Gi
        gke-parallelstore/ephemeral-storage-request: 500Mi
        gke-parallelstore/cpu-limit: 1000m
        gke-parallelstore/memory-limit: 2Gi
        gke-parallelstore/ephemeral-storage-limit: 1Gi
    spec:
      containers:
      - name: fio
        resources:
          requests:
            cpu: "1"
        image: litmuschaos/fio
        args:
        - fio
        - --filename
        - /models/gemma-7b/model-00001-of-00004.safetensors:/models/gemma-7b/model-00002-of-00004.safetensors:/models/gemma-7b/model-00003-of-00004.safetensors:/models/gemma-7b/model-00004-of-00004.safetensors:/models/gemma-7b/model-00004-of-00004.safetensors
        - --direct=1
        - --rw=read
        - --readonly
        - --bs=4096k
        - --ioengine=libaio
        - --iodepth=8
        - --runtime=60
        - --numjobs=1
        - --name=read_benchmark
        volumeMounts:
        - mountPath: "/models"
          name: parallelstore-pv 
      restartPolicy: Never
      volumes:
      - name: parallelstore-pv 
        persistentVolumeClaim:
          claimName: parallelstore-pvc
</pre>
<p>
</p>
<p>
Result:
</p>
<p>
</p>
<h3>Test - GCSfuse write vs gcloud storage cp </h3>
<p>
Use case: AI customers may want to write things back to the GCS bucket while
GCSfuse is already in place for READ actions. Our recommendation is to use
GCSfuse for READ and gcloud storage cp for WRITE.
</p>
<p>
This test is to show customer why we have this recommendation:
</p>
<p>
Summary:
</p>
<ul>
<li>Test environment: 10 Pods running on 10 nodes (n1-standard-16). Single GCS
bucket with 1 file 3.4GB. </li>
<li>Test steps (per pod): </li>
<ul>
<li>(cp) Copy the 3.4GB file from GCSfuse mount to /tmp</li>
<li>(cp) Copy the 3.4GB file from /tmp to a new directory at the same GCSFuse
mount</li>
<li>(gcloud storage cp) Copy the 3.4GB file to a new “folder” in the GCS
bucket.</li>
</ul>
<li>Result:</li>
<ul>
<li>cp - GCSFuse to /tmp  	~20sec</li>
<li>cp - /tmp to GCSfuse 		~50sec - 1min</li>
<li>Gcloud storage cp to GCS	~20sec</li>
</ul></li>
</ul>
<p>
Sample result:
</p>
<p>
Cp from GCSFuse to local:
</p>


<pre
class="prettyprint">0.06user 7.29system 0:20.65elapsed 35%CPU (0avgtext+0avgdata 1996maxresident)k
</pre>
<p>
Cp from local to GCSFuse:
</p>


<pre
class="prettyprint">0.06user 6.21system 0:59.61elapsed 10%CPU (0avgtext+0avgdata 1940maxresident)k
</pre>
<p>
Gcloud storage cp :
</p>
<p>
Average throughput: 225.0MiB/s
</p>
<p>
32.24user 17.74system 0:19.14elapsed 261%CPU (0avgtext+0avgdata
84652maxresident)k
</p>
<p>
Test commands:
</p>
<p>
Create the GKE cluster:
</p>


<pre
class="prettyprint">gcloud container clusters create fusewritetest    --cluster-version=1.30.5-gke.1014001    --addons GcsFuseCsiDriver --location=us-central1      --machine-type=n1-standard-16 --enable-autoscaling --num-nodes 1 --max-nodes 25 --workload-pool=thomashk-mig.svc.id.goog</pre>
<p>
Commands to get credential:
</p>


<pre
class="prettyprint">gcloud container clusters get-credentials fusewritetest     --location=us-central1</pre>
<p>
Create namespace:
</p>


<pre
class="prettyprint">kubectl create namespace ns-fusetest</pre>
<p>
Create service account:
</p>


<pre
class="prettyprint">kubectl create serviceaccount ksa     --namespace ns-fusetest</pre>
<p>
IAM binding:
</p>


<pre
class="prettyprint">gcloud storage buckets add-iam-policy-binding gs://thomashk-gcsfuse     --member "principal://iam.googleapis.com/projects/594050280394/locations/global/workloadIdentityPools/thomashk-mig.svc.id.goog/subject/ns/ns-fusetest/sa/ksa"     --role "roles/storage.admin"</pre>
<p>
We use storage admin to make sure we can create object back to the GCS bucket
with GCSfuse. <br><br>Sample job yaml:
</p>


<pre
class="prettyprint">apiVersion: batch/v1
kind: Job
metadata:
  name: gcs-10-fusewrite-test
  namespace: ns-fusetest
spec:
  parallelism: 10
  template:
    metadata:
      annotations:
        gke-gcsfuse/volumes: "true"
        gke-gcsfuse/memory-request: 10Gi
    spec:
      containers:
      - name: clicontainer
        image: gcr.io/google.com/cloudsdktool/google-cloud-cli:latest
        resources:
          limits:
            cpu: 12
            memory: 2048Mi
          requests:
            cpu: 12
            memory: 2048Mi
        command:
          - "/bin/sh"
          - "-c"
          - |
            apt install time
            echo "cp to local /tmp"
            time cp /data/dummy_files/dummy_file_02.bin /tmp/

            echo "rename the file with different name "
            mv /tmp/dummy_file_02.bin /tmp/dummy_file_02.bin_$(hostname -s)

            echo "cp from local to GCS bucket"
            time cp /tmp/dummy_file_02.bin_$(hostname -s) /data/write-tmp/

            echo "gcloud cp tp GCS bucket"
            time gcloud storage cp /tmp/dummy_file_02.bin_$(hostname -s) gs://thomashk-gcsfuse/storage-cp-tmp/
        volumeMounts:
        - name: gcs-fuse-csi-ephemeral
          mountPath: /data
          readOnly: false
      serviceAccountName: ksa
      volumes:
      - name: gke-gcsfuse-cache
        emptyDir:
          medium: Memory
      - name: gcs-fuse-csi-ephemeral
        csi:
          driver: gcsfuse.csi.storage.gke.io
          volumeAttributes:
            bucketName: thomashk-gcsfuse
            mountOptions: "implicit-dirs,file-cache:enable-parallel-downloads:true"
            fileCacheCapacity: "10Gi"
      restartPolicy: Never
  backoffLimit: 1
</pre>
<p>
Execute the workload:
</p>


<pre
class="prettyprint">kubectl create -f 10job-fuse-write-gcs.yaml</pre>




