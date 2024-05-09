# batch-az-cli-dock-cmds

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

Just create a new ACI through the Azure portal (```portal.azure.com```), select the ACR, app name, etc. **DON"T** forget to add a DNS label name if you want to connect to a (sub)domain eventually. EZ & straightforward.
