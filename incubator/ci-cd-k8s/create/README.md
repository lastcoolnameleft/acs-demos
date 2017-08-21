# Create an Azure Container Service, Install Jenkins and a CI/CD Pipeline for a container

Here, we will perform the following:
* Create an instance of the Azure Container Service
* Deploy Helm
* Install Jenkins via Helm
* Deploy a CI/CD Pipeline for Jenkins

## Prerequisites

We need to have installed the [Azure CLI](../../azure_compute/cli/install/).
We need to have created an [ACS (Kubernetes) cluster](https://raw.githubusercontent.com/Azure/acs-demos/master/kubernetes/create_cluster/script.md).
We need to have created an [ACR Instance](../../azure_compute/acr/install/).
We need to have installed [Helm](../../azure_compute/helm/install/).

## Environment Setup

The current environment setup is:

```
env | grep SIMDEM_.*
```

## Init Helm

```
helm init
helm repo update
```

# Install Kube-Lego Chart

```
helm install stable/kube-lego --set config.LEGO_EMAIL=$KUBE_LEGO_EMAIL,config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory
```

# Install Nginx Ingress Chart

```
helm --namespace jenkins --name jenkins -f ./jenkins-values.yaml install stable/jenkins
sleep 30

unset SERVICE_NAME
while [[ $SERVICE_NAME == "" ]]; do SERVICE_NAME=$(helm list | grep nginx-ingress | tail -1 | cut -f 1 | xargs); done;

echo $SERVICE_NAME

unset SERVICE_IP
while [[ $SERVICE_IP == "" ]]; do SERVICE_IP=$(kubectl get svc/$SERVICE_NAME-nginx-ingress-controller --output json  | jq --raw-output ".status.loadBalancer.ingress[0].ip" | xargs); done;

echo $SERVICE_IP
```

# Add DNS Entry

At this point, break out and add the following to /etc/hosts:
echo $SERVICE_IP $DEMO_SUB_DOMAIN.$DEMO_BASE_DOMAIN
sudo echo $SERVICE_IP $DEMO_SUB_DOMAIN.$DEMO_BASE_DOMAIN | sudo tee -a /etc/hosts

```
curl -I $DEMO_SUB_DOMAIN.$DEMO_BASE_DOMAIN
```

# Setup Jenkins

First, get the Jenkins setup password:

```
kubectl get secret --namespace jenkins jenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode
```

# Create Pipeline from Repo

Follow Steps
* Open Browser to http://ci-cd.azure-demo.com
  * User: Admin
  * Password: Jenkins password from above

# Enter ACR credentials for Container Publish

Get the credentials from ACR

```
ACR_PASSWORD=$(az acr credential show -n $ACR_NAME| jq --raw-output ".passwords[0].value")
echo $ACR_PASSWORD

kubectl delete secret myregistrykey

kubectl create secret docker-registry myregistrykey --namespace croc-hunter --docker-server=$ACR_NAME.azurecr.io --docker-username=$ACR_NAME --docker-password=$ACR_PASSWORD --docker-email=$EMAIL

kubectl get secret myregistrykey -o yaml
```

* Click "Credentials"
* Click "System"
* Click "Global credentials (unrestricted)"
* Click "Add Credentials"
  * Username: sandboxacr
  * Password: Output from "az acr credential show"
  * ID: registry_creds
  * Click "Save"

# Create Build Pipeline

* Click "Open Blue Ocean"
  * "Where do you store your code?"
	* Select Github
  * "Which Organization does the repository belong to?"
	* Select your personal
  * "Create a single Pipeline or discover all Pipelines?"
    * Select "New Pipeline"
  * "Choose a Repository"
	* Select "croc-hunter"