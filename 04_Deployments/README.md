# Managing Deployments Using Kubernetes Engine

## Courses
* https://www.cloudskillsboost.google/focuses/639?parent=catalog

## Background

### Limitation of Single Region or Location
* Resources
* Latency
* Availability
* Lock-in
* ...
_(TODO: Explain Characteristics)_

### Managing Deployments

#### Scale-Out, Scale-In
_(TODO)_
#### Rolling-Update
_(TODO)_
#### Canary
_(TODO)_
#### Blue-Green
_(TODO)_

## Commands

### Prerequisites
```bash
# Create GKE with 5 nodes of n1-standard-1
gcloud config set compute/zone us-central1-a
gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"

# Download Source Codes
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
```
### Deployments
```bash
# Explore deployment resource
kubectl explain deployment
kubectl explain deployment --recursive
kubectl explain deployment.metadata.name

# Create Deployment (Auth Application)
vi deployments/auth.yaml
# "i"
# ...
# containers:
# - name: auth
#   image: kelseyhightower/auth:1.0.0
# ...
# <ESC> + ":wq"
cat deployments/auth.yaml

kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml
# kubectl get deployments
# kubectl get replicasets
# kubectl get pods
kubectl get deployments,replicasets,pods,service

# Create Deployment (Hello Application)
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

# Create Deployment (Frontend Application)
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml

kubectl get services frontend
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
```

### Scale-Out, Sacle-In
```bash
# Explore replicas in deployment resource
kubectl explain deployment.spec.replicas

# Change replicas
kubectl scale deployment hello --replicas=5
kubectl get pods | grep hello- | wc -l
kubectl scale deployment hello --replicas=3
kubectl get pods | grep hello- | wc -l
# on 2nd Cloud Shell
# kubectl get pods -w
# on 1st Cloud Shell
# kubectl scale deployment hello --replicas=5
# kubectl scale deployment hello --replicas=3
```

### Rolling-Update
```bash
# Explore strategy in deployment resource
# kubectl explain deployment.spec.strategy.rollingUpdate  # maxSure: 25%, maxUnavailable: 25%

# Trigger Rolling-Update
kubectl edit deployment hello
# ...
# containers:
#   image: kelseyhightower/hello:2.0.0
# ...
kubectl get replicaset
kubectl rollout history deployment/hello

# Pause Rolling-Update
kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello

# Resume Rolling-Update
kubectl rollout resume deployment/hello
kubectl rollout status deployment/hello

# Rollback Rolling-Update
kubectl rollout undo deployment/hello
kubectl rollout history deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

### Canary
```bash
# Deploy Canary Application
cat deployments/hello-canary.yaml
kubectl create -f deployments/hello-canary.yaml
kubectl get deployments
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

# Explore Session Affinity
# kubectl explain service.spec.sessionAffinity
# kind: Service
# ...
# spec:
#   sessionAffinity: ClientIP
# ...
```

### Blue-Green Deploy
```bash
# Deploy Blue and Green Application
kubectl apply -f services/hello-blue.yaml
kubectl create -f deployments/hello-green.yaml

# Update Service for Apply Green Application
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
kubectl apply -f services/hello-green.yaml
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

# Rollback to Blue
kubectl apply -f services/hello-blue.yaml
```
