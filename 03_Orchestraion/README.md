# Orchestrating the Cloud with Kubernetes

## Course
* https://www.cloudskillsboost.google/focuses/557?parent=catalog

## Background

### Pods
> Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.<br><br>A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.<br><br> _Pods - Kubernetes Guide (https://kubernetes.io/docs/concepts/workloads/pods/)_

### Service
> An abstract way to expose an application running on a set of Pods as a network service.
With Kubernetes you don't need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.<br>...<br>Publishing Services (ServiceTypes)<br><br>Type values and their behaviors are:<br>* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.<br>* NodePort: Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.<br>* LoadBalancer: Exposes the Service externally using a cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.<br>* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.<br><br>_Service - Kubernetes Guide (https://kubernetes.io/docs/concepts/services-networking/service/)_

### Deployments
_(TODO)_

## Commands

### Create GKE
```bash
gcloud config set compute/zone us-central1-b
gcloud container clusters create io

gsutil cp -r gs://spls/gsp021/* .
cd orchestrate-with-kubernetes/kubernetes
ls
```

### Deploy and Expose Nginx
```bash
kubectl create deployment nginx --image=nginx:1.10.0
kubectl get pods
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get services

EXTERNAL_IP=$(kubectl get service nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "http://$EXTERNAL_IP"
```

### Deploy Application (with Pods, Monolith)
```bash
cat pods/monolith.yaml
kubectl create -f pods/monolith.yaml
kubectl get pods
kubectl describe pods monolith

kubectl port-forward monolith 10080:80
curl http://127.0.0.1:10080

# Acess Monolith Application without AuthN 
curl http://127.0.0.1:10080/secure

# Acess Monolith Application with JWT Token
curl -u user http://127.0.0.1:10080/login
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure

# 2nd Cloud Shell - Print Application Logs
kubectl logs monolith
kubectl logs -f monolith
# 1st Cloud Shell
curl http://127.0.0.1:10080

# Access to Terminal of Monolith Application Container
kubectl exec monolith --stdin --tty -c monolith /bin/sh
ping -c 3 google.com
exit
```

### Expose Application (via Service)

```bash
cd ~/orchestrate-with-kubernetes/kubernetes
cat pods/secure-monolith.yaml
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml

cat services/monolith.yaml
kubectl create -f services/monolith.yaml

gcloud compute firewall-rules create allow-monolith-nodeport --allow=tcp:31000
gcloud compute instances list
EXTERNAL_IP=$(kubectl get service nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl -k https://$EXTERNAL_IP:31000
```

```bash
# Select Pods with Labels
kubectl get pods -l "app=monolith"
kubectl get pods -l "app=monolith,secure=enabled"

# Add Labels
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
kubectl describe services monolith | grep Endpoints

gcloud compute instances list
curl -k https://$EXTERNAL_IP:31000
```

### Deploy Application (with Deployments, Microservice)

```bash
cat deployments/auth.yaml

kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml

kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml

kubectl get services frontend
EXTERNAL_IP=$(kubectl get service frontend -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl -k https://$EXTERNAL_IP
# echo https://$EXTERNAL_IP
```