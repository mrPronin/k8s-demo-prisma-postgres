# k8s-demo-prisma-postgres
Kubernetes demo config for prisma deployment with PostgreSQL on Google Cloud

# Parameters
PROJECT_ID: [my_project_id]

PROJECT_NUMBER: [my_project_number]

CLUSTER_NAME: [my_cluster_name]

ZONE: [my_zone], for example: us-central1-f


# Init shell & env
gcloud config set project [my_project_id]

PROJECT_ID=$(gcloud config get-value project)

PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"

CLUSTER_NAME=[my_cluster_name]

ZONE=[my_zone]

gcloud config set compute/zone ${ZONE}

gcloud container clusters get-credentials ${CLUSTER_NAME}

# Activate GC project
## Create project on GC (optional)

## Initialize Cloud SDK
https://cloud.google.com/sdk/docs/initializing

gcloud init


## Fetch configurations list
gcloud config configurations list

## Activate configuration
gcloud config configurations activate [CONFIGURATION_NAME]


## Check current gcloud settings
https://cloud.google.com/sdk/gcloud/reference/config/list

gcloud config list

## Activate API

gcloud services enable \\\
	cloudshell.googleapis.com \\\
	container.googleapis.com \\\
	containeranalysis.googleapis.com
  
# Kubernetes Cluster
## Create
gcloud beta container --project ${PROJECT_ID} \\\
clusters create ${CLUSTER_NAME} \\\
--zone ${ZONE} \\\
--machine-type "n1-standard-1" \\\
--num-nodes "3" \\\
--max-nodes=3 \\\
--min-nodes=1 \\\
--image-type "COS" \\\
--enable-stackdriver-kubernetes \\\
--enable-ip-alias \\\
--enable-autoupgrade \\\
--enable-autorepair \\\
--no-enable-basic-auth \\\
--disk-type "pd-standard" \\\
--disk-size "100" \\\
--default-max-pods-per-node "110" \\\
--addons HorizontalPodAutoscaling,HttpLoadBalancing \\\
--scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append"

## Verify cluster status
gcloud container clusters list

## Authenticate kubectl
gcloud container clusters get-credentials ${CLUSTER_NAME}

# Deploy prisma to GKE
## 'prisma' namespace
### Describe 'prisma-namespace.yaml'
### Apply namespace
kubectl apply -f k8s/prisma-namespace.yaml
### Check namespaces
kubectl get namespaces

## Disk provisioning
### Describe 'postgres-pvc.yaml'
### Apply PVC
kubectl apply -f k8s/postgres-pvc.yaml

## PostgreSQL
### Describe 'postgres-deployment.yaml'
### Apply PostgreSQL deployment
kubectl apply -f ./k8s/postgres-deployment.yaml
### Check PostgreSQL deployment
kubectl get pods -n prisma
kubectl get deployment -n prisma

## PostreSQL service
### Describe 'postgres-service.yaml'
### Apply PostgreSQL service
kubectl apply -f k8s/postgres-service.yaml
### Check PostgreSQL service
kubectl get services -n prisma

## prisma ConfigMap
### Describe 'prisma-configmap.yaml'
### Apply prisma ConfigMap
kubectl apply -f k8s/prisma-configmap.yaml
### Check prisma ConfigMap
kubectl get configmaps -n prisma

## prisma deployment
### Describe 'prisma-deployment.yaml'
### Apply prisma deployment
kubectl apply -f k8s/prisma-deployment.yaml
### Check prisma deployment
kubectl get pods -n prisma
kubectl get deployment -n prisma

## prisma service
### Describe 'prisma-service.yaml'
### Apply prisma service
kubectl apply -f k8s/prisma-service.yaml
### Check prisma service
kubectl get services -n prisma

## Configure Prisma CLI
kubectl port-forward svc/prisma 5000:4466 -n prisma
