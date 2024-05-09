# python-installation

## Quick handy az-cli & docker commands to deploy docker containers to Azure.


### P1: Create container image
Create a Dockerfile in root folder, add the content (change accordingly).<br />
```Dockerfile
  FROM node:x.x.x-*******
  RUN mkdir -p /usr/src/app
  COPY ./app/* /usr/src/app/
  WORKDIR /usr/src/app
  RUN npm install
  CMD node /usr/src/app/index.js
```
<br />

Build image using ```docker build```<br />
  ```
    docker build ./project-name -t project-app-name
  ```
 <br />

Check container image ``` docker images ```<br /><br />


Running container locally <br />

```sh
  docker run -d -p 8080:<PORT> project-app-name
```

### P2: Pushing image to Azure Container Registry

Create a resource group with the ```az group create``` command<br />
```
  az group create --name myResourceGroup --location eastus
```
<br />

Create an Azure **container registry** with the ```az acr create``` command<br />
```
  az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```
<br />

Login to **container registry**<br />
```
  az acr login --name <acrName>
```
<br />

Get the full login server name for the Azure container registry<br />
```
  az acr show --name <acrName> --query loginServer --output table
  az acr show --name notchacr --query loginServer --output table
```
<br />

Now, display the list of local images with ```docker images``` command<br /><br />

Tag the  image with the login server of your container registry.<br />
```
  docker tag project-app-name <acrLoginServer>/aci-tutorial-app:v1
```
<br />

Confirm with ```docker images```<br /><br />

Finally, **push** image to *Azure Container Registry*<br />
```
  docker push <acrLoginServer>/project-app-name:v1
  docker push notchacr.azurecr.io/project-app-name:v1
```
<br />

Verify pushed images with<br />
```
  az acr repository list --name <acrName> --output table
  az acr repository list --name notchacr --output table
```
<br />


### P3: Deploy container application to *Azure Container Instances*

Get Registry Credentials / Run this script to create a service principal. *Take note of the service principal id and password*<br />
```sh
  #!/bin/bash
  # This script requires Azure CLI version 2.25.0 or later. Check version with `az --version`.
  
  # Modify for your environment.
  # ACR_NAME: The name of your Azure Container Registry
  # SERVICE_PRINCIPAL_NAME: Must be unique within your AD tenant
  notchacr=$containerRegistry
  faxmishok=$servicePrincipal
  
  # Obtain the full registry ID
  ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query "id" --output tsv)
  # echo $registryId
  
  # Create the service principal with rights scoped to the registry.
  # Default permissions are for docker pull access. Modify the '--role'
  # argument value as desired:
  # acrpull:     pull only
  # acrpush:     push and pull
  # owner:       push, pull, and assign roles
  PASSWORD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query "password" --output tsv)
  USER_NAME=$(az ad sp list --display-name $SERVICE_PRINCIPAL_NAME --query "[].appId" --output tsv)
  
  # Output the service principal's credentials; use these in your services and
  # applications to authenticate to the container registry.
  echo "Service principal ID: $USER_NAME"
  echo "Service principal password: $PASSWORD"
```
<br />

To launch a container in Azure Container Instances using a service principal, specify its ID for ```--registry-username```, and its password for ```--registry-password```<br />
```
  az container create --resource-group myResourceGroup --name mycontainer --image mycontainerregistry.azurecr.io/myimage:v1 --registry-login-server mycontainerregistry.azurecr.io --registry-username <service-      principal-ID> --registry-password <service-principal-password>
```
