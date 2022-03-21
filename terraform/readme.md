# Demo of AKS using Terraform

Tool Chain:

1. VS CODE
2. Terraform
3. Azure Subscription
4. Azure CLI
5. GIT

## Azure CLI

``` Text

Download the Azure CLI and Install
You can get the Azure CLI on [docs.microsoft.com](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli) <br/>
We'll need the Azure CLI to gather information so we can build our Terraform file.

```

## Login to Azure and follow prompts

``` TEXT
az login
TENTANT_ID=**"your-tenant-id"**
``` TEXT

## _view and select your subscription account_

``` TEXT

az account list -o table SUBSCRIPTION=**"ID"**
az account set --subscription $SUBSCRIPTION

```

## _Create Service Principal_

``` TEXT
Kubernetes needs a service account to manage our Kubernetes cluster </br>
Lets create one! </br>

$SERVICE_PRINCIPAL_JSON=$(az ad sp create-for-rbac --skip-assignment --name aks-getting-started-sp -o json)

```

**_Keep the `appId` and `password` for later use_**

``` TEXT

$SERVICE_PRINCIPAL=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.appId')
$SERVICE_PRINCIPAL_SECRET=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.password')

# note: reset the credential if you have any sinlge or double quote on password

az ad sp credential reset --name "aks-getting-started-sp"

```

## Grant contributor role over the subscription to our service principal

``` TEXT

az role assignment create --assignee $SERVICE_PRINCIPAL \
--scope "/subscriptions/$SUBSCRIPTION" \
--role Contributor

For extra reference you can also take a look at the Microsoft Docs: [here](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/aks/kubernetes-service-principal.md) </br>
```

## Terraform CLI

``` TEXT

# Get Terraform and Install or Extract
``` TEXT

Download terraForm click [here]<https://releases.hashicorp.com/terraform/1.0.11/terraform_1.0.11_linux_amd64.zip> and unzip on file system.

open commnad prompt or powershell and navigate to folder where terrafomr has been extracted

```

## Generate SSH key

``` TEXT

ssh-keygen -t rsa -b 4096 -N "VeryStrongSecret123!" -C "your_email@example.com" -q -f  C:\Users\sp_admin\.ssh\id_rsa
SSH_KEY=$(cat ~/.ssh/id_rsa.pub)

```

## Terraform Azure Kubernetes Provider

Documentation on all the Kubernetes fields for terraform [here](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

```TEXT

terraform init

terraform plan -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENTANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"

terraform apply -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENTANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"

```

## Lets see what we deployed

```TEXT

# grab our AKS config

az aks get-credentials -n aks-getting-started -g aks-getting-started

# Get kubectl

Download and install kubectl

kubectl get svc

```

## Any issues in Pods

```TEXT

kubectl get all
kubectl get pods
kubectl  describe pod <podname>

```

## Clean up

```TEXT

terraform destroy -var serviceprinciple_id=$SERVICE_PRINCIPAL \
    -var serviceprinciple_key="$SERVICE_PRINCIPAL_SECRET" \
    -var tenant_id=$TENTANT_ID \
    -var subscription_id=$SUBSCRIPTION \
    -var ssh_key="$SSH_KEY"
```

## Building a Container

There is no Dockerfile in this project. You can build a container image (if you have a docker daemon) using the Spring Boot build plugin:

``` TEXT

docker-compose -f "docker-compose.yml" up -d --build
docker push <myregistry>

After building the conatiner we will push the image to docker hub.

```

## Update Code

``` TEXT

kubectl apply -f '<Deployment Path of YAML File >'

e.g.

kubectl create -f deployment.yaml

```

## More resources

```TEXT

Terraform provider for Azure [here](https://github.com/terraform-providers/terraform-provider-azurerm)

```
