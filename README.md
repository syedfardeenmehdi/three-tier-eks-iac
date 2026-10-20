# three-tier-eks-iac

# Prerequisite 

**Install Kubectl**
https://kubernetes.io/docs/tasks/tools/


**Install Helm**
https://helm.sh/docs/intro/install/

```
helm repo update
```

**Install/update latest AWS CLI:** (make sure install v2 only)
https://aws.amazon.com/cli/

#update the Kubernetes context
aws eks update-kubeconfig --name my-eks-cluster --region us-west-2

# verify access:
```
kubectl auth can-i "*" "*"
kubectl get nodes
```

# Verify autoscaler running:
```
kubectl get pods -n kube-system
```

# Check Autoscaler logs
```
kubectl logs -f \
  -n kube-system \
  -l app=cluster-autoscaler
```

# Check load balancer logs
```
kubectl logs -f -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

<!-- aws eks update-kubeconfig \
  --name my-eks \
  --region us-west-2 \
  --profile eks-admin -->


# Buid Docker image :
**For Mac:**

```
export DOCKER_CLI_EXPERIMENTAL=enabled
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/w8u5e4v2
```

Buid Front End :

```
docker buildx build --platform linux/amd64 -t workshop-frontend:v1 . 
docker tag workshop-frontend:v1 public.ecr.aws/w8u5e4v2/workshop-frontend:v1
docker push public.ecr.aws/w8u5e4v2/workshop-frontend:v1
```


Buid Back End :

```
docker buildx build --platform linux/amd64 -t workshop-backend:v1 . 
docker tag workshop-backend:v1 public.ecr.aws/w8u5e4v2/workshop-backend:v1
docker push public.ecr.aws/w8u5e4v2/workshop-backend:v1
```

**For Linux/Windows:**

Buid Front End :

```
docker build -t workshop-frontend:v1 . 
docker tag workshop-frontend:v1 public.ecr.aws/w8u5e4v2/workshop-frontend:v1
docker push public.ecr.aws/w8u5e4v2/workshop-frontend:v1
```


Buid Back End :

```
docker build -t workshop-backend:v1 . 
docker tag workshop-backend:v1 public.ecr.aws/w8u5e4v2/workshop-backend:v1
docker push public.ecr.aws/w8u5e4v2/workshop-backend:v1
```



**Create Namespace**
```
kubectl create ns workshop

kubectl config set-context --current --namespace workshop
```

# MongoDB Database Setup

**To create MongoDB Resources**
```
cd k8s_manifests/mongo_v1
kubectl apply -f secrets.yaml
kubectl apply -f deploy.yaml
kubectl apply -f service.yaml
```

# Backend API Setup

Create NodeJs API deployment by running the following command:
```
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
``


**Frontend setup**

Create the Frontend  resource. In the terminal run the following command:
```
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

Finally create the final load balancer to allow internet traffic:
```
kubectl apply -f full_stack_lb.yaml
```


# Any issue with the pods ? check logs:
kubectl logs -f POD_ID -f


# Grafana setup 
Username: admin
Password: prom-operator

Import Dashboard ID: 1860

Exlore more at: https://grafana.com/grafana/dashboards/


---

# Local Minikube Deployment (No AWS Required)

This section explains how to deploy and run the three-tier application locally using Docker Desktop and Minikube, without creating AWS resources.

## Prerequisites

- Docker Desktop installed and running
- Minikube installed
- Project application images built and loaded into Minikube

## 1. Start Minikube

Open PowerShell in the project directory:

```powershell
cd D:\Projects\three-tier-eks-iac
minikube start --driver=docker --cpus=2 --memory=3072
minikube status
```

## 2. Load the application images

Run these commands if the images are not already loaded into Minikube:

```powershell
minikube image load three-tier-eks-iac-backend:latest
minikube image load three-tier-eks-iac-frontend:latest
minikube image load mongo:4.4.6
```

## 3. Deploy the Kubernetes resources

The local configuration is stored in `local-minikube.yaml`. It is separate from the original AWS/EKS manifests.

```powershell
minikube kubectl -- apply -f .\local-minikube.yaml
```

## 4. Verify the deployment

```powershell
minikube kubectl -- -n local-demo get pods
minikube kubectl -- -n local-demo get services
minikube kubectl -- -n local-demo get pvc
minikube kubectl -- -n local-demo get deployments
```

Wait until all three application pods are ready and the MongoDB persistent volume claim shows `Bound`.

## 5. Access the application

Open two separate PowerShell windows in the project directory. Keep both windows open while using the application.

**Terminal 1 — Backend port forwarding**

```powershell
minikube kubectl -- -n local-demo port-forward service/backend 8082:8080
```

**Terminal 2 — Frontend port forwarding**

```powershell
minikube kubectl -- -n local-demo port-forward service/frontend 3000:3000
```

Open the application in your browser:

http://localhost:3000

The frontend communicates with the backend through:

http://localhost:8082/api/tasks

## 6. View application logs

```powershell
minikube kubectl -- -n local-demo logs deployment/backend --tail=30
minikube kubectl -- -n local-demo logs deployment/frontend --tail=30
minikube kubectl -- -n local-demo logs deployment/mongodb --tail=30
```

## 7. Verify MongoDB persistence

Create a task in the application and refresh the page to confirm it remains available.

To test persistence after a MongoDB pod restart:

```powershell
minikube kubectl -- -n local-demo rollout restart deployment/mongodb
minikube kubectl -- -n local-demo rollout status deployment/mongodb
```

After the restart completes, refresh the application and confirm that the task is still available.

## Notes

- Kubernetes resources for this deployment use the `local-demo` namespace.
- MongoDB stores its data using a PersistentVolumeClaim (PVC).
- Both port-forwarding terminals must remain open while accessing the application.
- This local deployment uses Minikube and does not require AWS resources.
- The original AWS/EKS instructions and manifests are retained separately.
