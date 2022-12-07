# GitOps  - ArgoCD Demo


## Lets create the GKE Cluster using the notepad-infra folfer.
```
export student_name=<STUDENT_NAME>
export ORG=<ORG>
export PRODUCT=<PRODUCT>
export ENV=<ENV>
export PROJECT_ID=<project-id>
gcloud config set project $PROJECT_ID

cd notepad-infrastructure
terraform init
terraform plan -var-file terraform.tfvars
terraform apply -var-file terraform.tfvars
```
Connect to your cluster
```
gcloud container clusters get-credentials gke-$ORG-$PRODUCT-$ENV --region us-central1
```
### Create ArgoCD Namespace
This will create a new namespace, argocd, where Argo CD services and application resources will live.

```
kubectl create namespace argocd
```
### Install ArgoCD
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### Install ArgoCD CLI
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```
### Access The Argo CD API Server
By default, the Argo CD API server is not exposed with an external IP. To access the API server, choose one of the following techniques to expose the Argo CD API server:

Change the argocd-server service type to LoadBalancer:
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
Get the External IP running:
```
kubectl get svc -n argocd
```
http://externalIP

### Get the initial password. 
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
Default user is: admin

###Creating Apps Via UI

After logging in, click the + New App button

Give your app the name simple-app, use the project default, and leave the sync policy as Manual
```
Repository URL, use: https://github.com/jtapia05/cloud-journey
Path: simple-app
```
For Destination, set cluster URL to https://kubernetes.default.svc (or in-cluster for cluster name) and namespace to dev. So ArgoCD use a different namespace than default. 

Let the rest as default and in the top, click on create. 

### Once the guestbook application is created, you can now view its status using CLI
```
argocd app get simple-app
```
### To sync the app with the repo
```
argocd app sync simple-app
```
Alternatively, to sync from the ArgoCD UI, go to simple-app and click on SYNC.


### In order to create another app but using the CLI. Use the following code:
```
argocd app create bookinfo\
--project default \
--repo https://github.com/jtapia05/cloud-journey \
--path "./bookinfo" \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```
### To sync the app with the repo
```
argocd app sync bookinfo
```

### Scale up the application and pay attention to the GUI
As the replica is set to one in git, ArgoCD won't accept a manual change directly to the cluster, so it will revert the cluster to the original state.

```
kubectl scale deployment/bookinfo --replicas=5 -n default
```

### Fork my repo and feel free to continue playing with ArgoCD.
Resources:
https://learning.codefresh.io/
https://argo-cd.readthedocs.io/en/stable/getting_started/


### Cleanup
```
cd notepad-infrastructure
terraform destroy -var-file terraform.tfvars
```

