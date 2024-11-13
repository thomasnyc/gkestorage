<h3>GKE - GCSFuse with local SSD cache</h4>
<p>
GKE Cluster creation for GCSFuse with local ssd cache:
</p>


<pre
class="prettyprint">gcloud container clusters create gcsfuse-test1     --addons GcsFuseCsiDriver     --cluster-version=1.29.7-gke.1008000     --location=us-central1     --ephemeral-storage-local-ssd count=8     --machine-type=n1-standard-96 --enable-autoscaling --num-nodes 3 --min-nodes 3 --max-nodes 9 --workload-pool=thomashk-mig.svc.id.goog</pre>
<p>
Make sure there are local ssd as ephemeral storage with the container cluster
</p>

<li>Set up the credentials: 
<pre
class="prettyprint">gcloud container clusters get-credentials gcsfuse-test1 \
    --location=us-central1</pre>
</li>
</ol>

<li>Create namespace: 
<pre
class="prettyprint">kubectl create namespace ns-fusetest</pre>
</li>

<li>Create service account ksa: 
<pre
class="prettyprint">kubectl create serviceaccount ksa \
    --namespace ns-fusetest</pre>
</li>
<li>Assign IAM permission: 
<pre
class="prettyprint">gcloud storage buckets add-iam-policy-binding gs://thomashk-gcsfuse \
    --member "principal://iam.googleapis.com/projects/594050280394/locations/global/workloadIdentityPools/thomashk-mig.svc.id.goog/subject/ns/ns-fusetest/sa/ksa" \
    --role "roles/storage.objectUser"</pre>
</li>
</ol>
