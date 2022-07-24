# Google Kubernetes Engine (GKE)

## Background

### Features of GKE

1. Load Balancing
2. Node Pool
3. Autoscale
4. Auto-upgrade (Node Pool)
5. Self Healing
6. Logging and Monitoring (via Cloud Monitoring)

### Zone
> Compute Engine resources are hosted in multiple locations worldwide. These locations are composed of regions and zones. A region is a specific geographical location where you can host your resources. Regions have three or more zones. For example, the us-west1 region denotes a region on the west coast of the United States that has three zones: us-west1-a, us-west1-b, and us-west1-c.
>
> - _Regions and zones - Google Cloud (https://cloud.google.com/compute/docs/regions-zones/#available)_

### Standard Cluster Architecture
> In GKE, a cluster consists of at least one control plane and multiple worker machines called nodes.
>
> ...
>
> Nodes
>
> The individual machines are Compute Engine VM instances that GKE creates on your behalf when you create a cluster.
>
> ...
>
> ![image](https://cloud.google.com/static/kubernetes-engine/images/cluster-architecture.svg)

## Commands

### Create GKE
```bash
gcloud config set compute/zone us-central1-a
gcloud config list compute/zone

gcloud container clusters create gke-study           # About 5 mins
gcloud container clusters get-credentials gke-study

kubectl config get-contexts
# CURRENT   NAME                                                       CLUSTER                                                    AUTHINFO                                                   NAMESPACE
# *         gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study   gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study   gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study

kubectl config view
# apiVersion: v1
# clusters:
# - cluster:
#     certificate-authority-data: DATA+OMITTED
#     server: https://35.202.186.28
#   name: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
# contexts:
# - context:
#     cluster: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
#     user: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
#   name: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
# current-context: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
# kind: Config
# preferences: {}
# users:
# - name: gke_qwiklabs-gcp-00-704bffb34b90_us-central1-a_gke-study
#   user:
#     exec:
#       apiVersion: client.authentication.k8s.io/v1beta1
#       args: null
#       command: gke-gcloud-auth-plugin
#       env: null
#       installHint: Install gke-gcloud-auth-plugin for use with kubectl by following
#         https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
#       interactiveMode: IfAvailable
#       provideClusterInfo: true
```

### Deploy Image
```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
kubectl get deploy,service
# kubectl get all

# Exposed URL
EXTERNAL_IP=$(kubectl get service hello-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$EXTERNAL_IP:8080
```

### Remove Kubernetes Cluster
```bash
gcloud container clusters delete
```