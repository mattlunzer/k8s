## Deploy a k8s service type LoadBalancer with a static IP sourced from an Azure Public IP Prefix

### Create Resource Group
az group create --name rg-myResourceGroup --location eastus

### Create AKS Cluster
```
az aks create -g rg-myResourceGroup -n myAKSCluster --enable-managed-identity --node-count 1 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
```

### Create Public IP Prefix (note --length, up to /28 in size)
```
az network public-ip prefix create \
    --length 30 \
    --name prefix-for-myAKSCluster \
    --resource-group rg-myResourceGroup \
    --location eastus \
    --version IPv4
```

### Create Public IP Prefix
```
az network public-ip create \
    --name pip-for-myAKSCluster \
    --resource-group rg-myResourceGroup \
    --allocation-method Static \
    --public-ip-prefix prefix-for-myAKSCluster \
    --sku Standard \
    --version IPv4
```

### Assign MI permissions to RG containing prefix
```
CLIENT_ID=$(az aks show --name myAKSCluster --resource-group rg-myResourceGroup --query identity.principalId -o tsv)
RG_SCOPE=$(az group show --name rg-myResourceGroup --query id -o tsv)
az role assignment create \
    --assignee ${CLIENT_ID} \
    --role "Network Contributor" \
    --scope ${RG_SCOPE}
```
