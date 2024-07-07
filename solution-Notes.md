# Solution Notes

## Step 1 - Create Dockerfile for Web Server

This command will create an image for Docker Hub.

```bash
docker build -t betuldurankaplan/webserver .
```

## Step 2 - Create Dockerfile for Result Server (Same as Step 1)

This command lists all Docker images, confirming that the images are ready.

```bash
docker build -t betuldurankaplan/resultserver .
docker image ls
```

## Step 3 - Docker Login and Push

If the Docker Hub password is changed, delete the .docker directory in the home folder and login again with the new credentials.

```bash

docker login
docker push betuldurankaplan/webserver
docker push betuldurankaplan/resultserver
```

## Step 4 - Persistent Volume and Persistent Volume Claim

# 4a - Create mysql-pv.yaml

Retain means the volume won't be deleted automatically; available means it's ready for use.

```bash
kubectl apply -f mysql-pv.yaml
k get pv
```

# 4b - Create mysql-pv-claim.yaml

Ensure the following three items are consistent between PV and PVC: storageClassName, storage, accessModes.
The PVC capacity should match the PV capacity.

```bash

kubectl apply -f mysql-pv-claim.yaml
k get pv,pvc
```

## Step 5 - Create mysql-deployment.yaml

You can create this file in three ways: using an extension, referencing the Kubernetes documentation, or using the dry-run command.
Modify the generated file as needed, adding volumes and environment variables from the MySQL documentation. Use secrets for passwords and config maps for non-sensitive information.

```bash
kubectl create deploy mysql --image=mysql:5.7 --port=3306 --dry-run=client -o yaml > mysql-deploy.yaml
```

## Step 6 - Apply MySQL Deployment

This command creates the deployment, with the PV at /mnt/data on the worker node.

```bash 
kubectl apply -f mysql-deploy.yaml
```

## Step 7 - Create and Apply Service for MySQL

Since this is a database service, a NodePort is not necessary; ClusterIP is sufficient.

```bash 
kubectl expose deploy/mysql --port=3306 --dry-run=client -o yaml > mysql-svc.yaml
kubectl apply -f mysql-svc.yaml
k get svc
```

## Step 8 - Create Deployment for Web Server

```bash 
kubectl create deploy webserver-deploy --image=betuldurankaplan/webserver --port=80 --dry-run=client -o yaml > webserver-deploy.yaml
kubectl apply -f webserver-deploy.yaml
k get po
```

## Step 9 - Create Service for Web Server

Ensure the selector sections of the pod and service match for proper routing.

```bash
kubectl expose deploy webserver-deploy --port=80 --dry-run=client -o yaml > webserver-service.yaml
kubectl apply -f webserver-service.yaml
k get svc
```

## Step 10 - Create Deployment for Result Server

Copy the web server YAML files and replace references to webserver with resultserver.

```bash
kubectl apply -f resultserver-deploy.yaml
k get pod
k get po,deploy,svc
```

## Step 11 - Create Service for Result Server

Copy the web server service YAML file, changing the name and port.

```bash
kubectl apply -f resultserver-service.yaml
k get po,deploy,svc
```

## Step 12 - Access the Application

Access the application at ip:30002

## Step 13 - Enhance YAML Files with Secrets and ConfigMaps

Create mysql-secret.yaml and server-configmap.yaml files. Use base64 encoding for passwords.

```bash
echo -n "Pl123456" | base64
```

Apply the secrets and config maps, then reference them in the deployment YAML files.

```bash

kubectl apply -f .
kubectl get secret,cm
kubectl get po,deploy,svc
```

## Step 14 - Update Environment Variables in Deployment YAML Files

Ensure environment variables are sourced from secrets and config maps. Apply the changes.

```bash

k delete -f .
k apply -f .
k get secret,cm
k get po,deploy,svc
```

## Step 15 - Test the Application

Test the application in the browser.

## Step 16 - Ingress Configuration

Ingress requires a Cloud Controller Manager for load balancing and routing.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/deploy.yaml
k apply -f ingress.yaml
k get ing
```

## Step 17 - Helm Installation

Install Helm from the official documentation.
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

## Step 18 - Create Helm Chart

Create a Helm chart and move YAML files to the templates folder.

```bash
helm create phonebook-chart
rm -r phonebook-chart/templates/*

Copy YAML files to the templates directory and update values.yaml.

```

## Step 19 - Update Deployment YAML Files

Reference images from values.yaml

```bash
image: {{ .Values.webserver_image }}
```

## Step 20 - Create NOTES.txt

```markdown

Thank you for installing {{ .Chart.Name }}.
Your release is named {{ .Release.Name }}.

To learn more about the release, try:
  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}

To use own images execute as below:

helm upgrade --install phonebook-app phonebook-repo/phonebook-chart --set webserver_image=<image-name> --set resultserver_image=<image-name> 
```

## Step 21 - Create a public repo in github for using as a Helm Repo and clone it.

use token when your Repo cloning.
It's name can be "phonebook-helm-repo "
use token when your Repo cloning.

## Step 22 - Package and Push Helm Chart

```bash
cd phonebook-helm-repo 
helm package ../phonebook-chart/
helm repo index .
git add .
git commit -m "phonebook" # your github credential could be necessery
git push
```

## Step 23 - Add and Install Helm Repo

Add the Helm repo and install the chart.

```bash
helm repo add app-repo https://raw.githubusercontent.com/betul-kaplan/phonebook-helm-repo/main #must be raw
helm install apprelease app-repo/phonebook-chart
```

the chart is released and can be installed using Helm.

## Step 24 -Delete Helm Repo

```bash
helm delete apprelese
k get po # check if the release delete .