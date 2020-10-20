# aks-terraform

### Prerequisites Steps
* configured Azure CLI
`az login`
* Create an Active Directory service principal account
`az ad sp create-for-rbac --skip-assignment`
Output should be something like the following:
```json
{
  "appId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "displayName": "azure-cli-2019-04-11-00-46-05",
  "name": "http://azure-cli-2019-04-11-00-46-05",
  "password": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "tenant": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
}
```
* Update `terraform.tfvars` file with `appId` and `password`

### Running Terraform to provision the AKS cluster
```sh
terraform init
terraform apply
```

### Post provision steps
##### Setting up Azure Container Registry:
`az acr create --resource-group homework-rg --name homeworkContainerRegistry101 --sku Basic`
> Using Github secret for CD pipelines:
> REGISTRY_LOGIN_SERVER = homeworkcontainerregistry101.azurecr.io
##### Getting push permissions to ACR:
`groupId=$(az group show --name homework-rg --query id --output tsv)`
`az ad sp create-for-rbac --scope $groupId --role Contributor --sdk-auth`
This generates a service principal and the output of the above command will be in the following format:

```json
{
  "clientId": "<client id>",
  "clientSecret": "<client secret>",
  "subscriptionId": "<subscription id>",
  "tenantId": "<tenant id>",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```
> Using Github secrets for CD pipelines:
> AZURE_CREDENTIALS = entire JSON
> REGISTRY_USERNAME = clientId
> REGISTRY_PASSWORD = clientSecret

##### Setup HTTP Add-ons - Ingress and external DNS
`az aks enable-addons --resource-group homework-rg --name homework-aks --addons http_application_routing`
> Getting the DNS Zone from the HTTP Addon from the JSON output:
> "HTTPApplicationRoutingZoneName": "AAAAAAAAAAAAAA.westus2.aksapp.io"

> Update both services Helm charts values.yml file with proper ingress.hosts record

##### Get Kubernetes Config File
`az aks get-credentials --resource-group homework-rg --name homework-aks`
> Using Github secrets for CD pipelines:
> AKS_KUBECONFIG = kubeconfig file content

##### Create service-b namespace
`kubectl create namespace service-b`