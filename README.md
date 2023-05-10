## Deploy a k8s service type LoadBalancer with a static IP sourced from an Azure Public IP Prefix

### Set variables
```
rg=myAKSResourceGroup
location=eastus
aksName=myAKSCluster
prefixLength=30
```
### Create Resource Group
```
az group create --name $rg --location $location
```

### Create AKS Cluster
```
az aks create -g $rg -n $aksName --enable-managed-identity --node-count 1 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```

### Create Public IP Prefix (note --length, up to /28 in size)
```
az network public-ip prefix create \
    --length $prefixLength \
    --name prefix-for-$aksName \
    --resource-group $rg \
    --location $location \
    --version IPv4
```

### Create Public IP Prefix
```
az network public-ip create \
    --name pip-for-$aksName \
    --resource-group $rg \
    --allocation-method Static \
    --public-ip-prefix prefix-for-$aksName \
    --sku Standard \
    --version IPv4
```

### Assign MI permissions to RG containing prefix
```
CLIENT_ID=$(az aks show --name $aksName --resource-group $rg --query identity.principalId -o tsv)
RG_SCOPE=$(az group show --name $rg --query id -o tsv)
az role assignment create \
    --assignee ${CLIENT_ID} \
    --role "Network Contributor" \
    --scope ${RG_SCOPE}
```
