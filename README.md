# Prometheus-k8s-Performance Clinic
Repository containing the files for the Performance Clinic related to Prometheus on K8s 


This repository showcase the usage of the Prometheus OpenMetrics Ingest by using GKE with :
- the HipsterShop
- Jenkins

## Prerequisite 
The following tools need to be install on your machine :
- jq
- kubectl
- git
- gcloud ( if you are using GKE)
- Helm
### 1.Create a Google Cloud Platform Project
```
PROJECT_ID="<your-project-id>"
gcloud services enable container.googleapis.com --project ${PROJECT_ID}
gcloud services enable monitoring.googleapis.com \
cloudtrace.googleapis.com \
clouddebugger.googleapis.com \
cloudprofiler.googleapis.com \
--project ${PROJECT_ID}
```
### 2.Create a GKE cluster
```
ZONE=us-central1-b
gcloud container clusters create onlineboutique \
--project=${PROJECT_ID} --zone=${ZONE} \
--machine-type=e2-standard-2 --num-nodes=4
```
### 3.Clone Github repo
```
git clone https://github.com/dynatrace-perfclinics/prometheus-k8s-pipeline
cd prometheus-k8s-pipeline
```
### Deploy the sample Application

#### 0. Istio
0. Create the various namespaces
For the hipsterShop :
```
   kubectl create namespace hipster-shop
   kubectl -n hipster-shop create rolebinding default-view --clusterrole=view --serviceaccount=hipster-shop:default
```

1. Download Istioctl
```
curl -L https://istio.io/downloadIstio | sh -
```
This command download the latest version of istio ( in our case iostio 1.10.2) compatible with our operating system.
2. Add istioctl to you PATH
```
cd istio-1.10.3
```
this directory contains samples with addons . We will refer to it later.
```
export PATH=$PWD/bin:$PATH
```
#### Install Istio
The installation of istio in your cluster will be done with the help of istioCtl
```
istionctl install  --set profile=demo -y
```
Then we want to instruct istio to automatically inject the envoy Proxy to all the pods of our Hipster-shop application
so we will label the namesapce : hipster-shoo
```
   kubectl label namespace hipster-shop istio-injection=enabled
```

#### 1.HipsterShop
```
cd hipstershop
./setup.sh
```

### 3. Expose the HipsterShop out of the cluster
```
kubectl apply -f /istio/hipstershop_gateway.yaml
```
### 4. Install Litmus Chaoas
```
helm repo add litmuschaos https://litmuschaos.github.io/litmus-helm/
kubectl create ns litmus
kubectl get pods -n litmus
kubectl apply -f /litmus_choas/scheduler.yaml -n litmus
```
Apply the experiments in the hipster-shop Namespace
```
kubectl apply -f /litmus_choas/experiment.yaml -n hipster-shop
```
Apply the Service account authorize to run the various experiments
```
kubectl apply -f /litmus_choas/rbac.yaml -n hipster-shop
```
Deploy the scheduled experiments :
```
kubectl apply -k /litmus_choas/schedule/ -n hipster-shop
```
### 5. Run a neoload test

#### uppdate the definition of your hipstershop url
TO get the ip adress of the istio gateway run the following command :
```
kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```
Update the following file loadtest/hipstershop/team/servers/34#2E89#2E214#2E38.xml
update the hostname value by replacing with your Ip adress.


