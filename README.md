# ScyllaDB Helm Chart on Google Kubernetes Engine (GKE)

 ## What is Helm Charts? Why use a Helm Chart? 
  
Helm uses a packaging format called charts. A chart is a collection of files that describe a related set of Kubernetes resources. A single chart might be used to deploy something simple, like a memcached pod, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on. 

Helm Charts deploys all the kubernetes entities in a ordered fashion wrapping them together in a RELEASE. In addition to that you also get versioning control allowing you to upgrade your release and rolling back changes.

A good introduction to Helm Charts by Amy Chen can be found [here](https://youtu.be/vQX5nokoqrQ)

## Running on GKE:
  
  * [Install Google Cloud SDK](https://cloud.google.com/sdk/)

  * Authenticate to your GCP
    
    `gcloud init`
  
  * Install kubectl
    
    `gcloud components install kubectl`

  * From your GCE  web console create a cluster on GKE. [How do I find my project id on GCP?](https://cloud.google.com/resource-manager/docs/creating-managing-projects?visit_id=1-636622601155195003-3404293793&rd=1#identifying_projects)
  
  * Export some environment variables
  
    `export PROJECT=<your-project-id> # see step below` 
  
    `export CLUSTER_NAME=scylla-helm` 
  
    `export CLUSTER_VERSION=1.9.7-gke.1` 
  
    `export ZONE=us-central1-a` 
  
  * Deploy a GKE cluster
  
   `gcloud beta container --project $PROJECT clusters create $CLUSTER_NAME --zone $ZONE --username "admin" --cluster-version $CLUSTER_VERSION --machine-type "n1-standard-8" --image-type "COS" --disk-size "100"`
    
  * Get credentials for your GKE cluster
    
    `gcloud container --project $PROJECT clusters get-credentials $CLUSTER_NAME --zone $ZONE`

  * Check your setup
    
    `kubectl config current-context`
    
     You should see something like: `gke_$PROJECT_$ZONE>_$CLUSTER_NAME` 
    
  * Clone our repository
    
    `git clone https://github.com/scylladb/scylla-code-samples.git`
    
    `cd scylla-code-samples/k8s-helm_chart`
    
  * [Install Helm](https://docs.helm.sh/using_helm/#installing-helm)
  
    Do not forget to run helm init 
    
    `helm init` 
    
 
  * Get your password
    
    `gcloud container clusters describe $CLUSTER_NAME --zone $ZONE | grep pass`
    
    take note of your password here to use on next steps
    
  * Setup RBAC
    
    `kubectl create serviceaccount --namespace kube-system tiller`
    
    `kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'`
    
    `kubectl --username=admin --password=<password> create -f scylladb-gke/tiller-clusterrolebinding.yaml`
    
    `kubectl --username=admin --password=<password> create -f scylladb-gke/cluster-admin.yaml`
    
  * Install the helm chart 
    
    `helm install scylladb` 
    
    This is going to install a new helm release with a random name. We will use the release name on the next steps.
    
  * Get the name of your helm release 
    
    `helm list` 
    
  * Export an environment variable for the release
  
    `export RELEASE=$(helm list | grep scylladb | tr -s \t | cut -f1)`
    
  * Check the status 
    
    `helm status $RELEASE` 
    
  * Check your scylla cluster 
    
    `kubectl exec -ti <some-pod-name> -- nodetool status # Check your cluster`
    
    `kubectl logs <some-pod-name> # Check the logs for some pod`
    
  * Grow your cluster by upgrading your Release - adding 2 more nodes. This will update the REVISION number on your release 
    
    `helm upgrade --set replicaCount=5, $RELEASE scylladb/` 
    
    `helm history $RELEASE`
    
    `kubectl exec -ti <some-pod-name> -- nodetool status`

  * Shrink your cluster by upgrading your Release - removing one node
    
    `helm upgrade --set replicaCount=4, $RELEASE scylladb/` 
    
    `helm history $RELEASE`
    
    `kubectl exec -ti <some-pod-name> -- nodetool status`
    
  * Shrink your cluster by rolling back to REVISION 1 - removing another node
    
    `helm rollback $RELEASE 1` 
    
    `helm history $RELEASE`
    
    `kubectl exec -ti <some-pod-name> -- nodetool status`
    
  * Delete your helm release
    
    `helm delete $RELEASE`
