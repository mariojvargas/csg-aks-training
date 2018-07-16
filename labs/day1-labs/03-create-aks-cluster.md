# Azure Kubernetes Service (AKS) Deployment

## Create AKS cluster

This is a very simple step, yet very powerful, as all the intricacies of configuring and installing a Kubernetes Cluster are abstracted out for you. Behind the scenes, Azure uses [ACS Engine](https://github.com/Azure/acs-engine) for creating the cluster. ACS Engine, not to be confused with Azure Container Services (ACS), is an open source utility for generating ARM Templates for Docker-enabled clusters, such as Kubernetes. It's worth noting that AKS *is* the replacement for Azure Container Services. Confusing... We know.

1. On your workstation (Linux VM or equivalent *nix environment), open the terminal.

2. If you haven't logged in to Azure, login now using the AZ CLI:

    ```bash
    $ az login
    ```

3. After logging in to Azure, create a brand-new Resource Group named `cbus-aks-training`. Take note of the resource group name we're using. We'll be reusing this name for our cluster.

    ```bash
    $ az group create --name cbus-aks-training --location eastus
    ```

4. Now we will create the AKS cluster named after its resource group. The cluster can be named anything, but this makes it easy for us to track down where the cluster is located. The backslashes (`\`) can be omitted if you intend to type one continuous line. **Be prepared for a long wait.** Creating an AKS cluster can take several minutes. In most cases, it can take about 30 minutes. You should see a message reading "`- Running ..`" once you enter the below command.

    ```bash
    $ az aks create --name cbus-aks-training \
       --resource-group cbus-aks-training \
       --node-count 2 \
       --kubernetes-version 1.7.7 \
       --generate-ssh-keys \
       ---location eastus
    ```

Once the cluster is created, the AZ CLI should output information about the cluster in JSON format. For automated scenarios, this can be helpful. You can ignore it for now, as it is a sign that the cluster was created successfully. Below is an example:

```json
{
"aadProfile": null,
"addonProfiles": null,
"agentPoolProfiles": [
    {
    "count": 2,
    "dnsPrefix": null,
    "fqdn": null,
    "maxPods": 110,
    "name": "nodepool1",
    "osDiskSizeGb": null,
    "osType": "Linux",
    "ports": null,
    "storageProfile": "ManagedDisks",
    "vmSize": "Standard_DS1_v2",
    "vnetSubnetId": null
    }
],
"dnsPrefix": "cbus-aks-t-cbus-aks-trainin-2ee8d5",
"enableRbac": false,
"fqdn": "cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io",
"id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/cbus-aks-training/providers/Microsoft.ContainerService/managedClusters/cbus-aks-training",
"kubernetesVersion": "1.7.7",
"linuxProfile": {
    "adminUsername": "azureuser",
    "ssh": {
    "publicKeys": [
        {
        "keyData": "ssh-rsa aBase64String== example@example.com\n"
        }
    ]
    }
},
"location": "eastus",
"name": "cbus-aks-training",
"networkProfile": {
    "dnsServiceIp": "10.0.0.10",
    "dockerBridgeCidr": "172.17.0.1/16",
    "networkPlugin": "kubenet",
    "networkPolicy": null,
    "podCidr": "10.244.0.0/16",
    "serviceCidr": "10.0.0.0/16"
},
"provisioningState": "Succeeded",
"resourceGroup": "cbus-aks-training",
"servicePrincipalProfile": {
    "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "keyVaultSecretRef": null,
    "secret": null
},
"tags": null,
"type": "Microsoft.ContainerService/ManagedClusters"
}
```

5. Verify your cluster status. The `ProvisioningState` should be `Succeeded`

    ```
    Name               Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
    -----------------  ----------  -----------------  -------------------  -------------------  -----------------------------------------------------------------
    cbus-aks-training  eastus      cbus-aks-training  1.7.7                Succeeded            cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io
    mario-aks-sandbox  eastus2     mario-aks-sandbox  1.8.11               Succeeded            mario-aks--mario-aks-sandbo-2ee8d5-1f2ba3ec.hcp.eastus2.azmk8s.io
    ```

6. Get the Kubernetes config files for your new AKS cluster. It should output a message saying that it merged the `cbus-aks-training` cluster as current cluster.

    ```bash
    $ az aks get-credentials -n cbus-aks-training -g cbus-aks-training

    Merged "cbus-aks-training" as current context in /Users/mvargas/.kube/config
    ```

7. Verify you have API access to your new AKS cluster

    > Note: It can take 5 minutes or even longer for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status. 
    
    ```bash
    $ kubectl get nodes

    NAME                       STATUS    ROLES     AGE       VERSION
    aks-nodepool1-57028559-0   Ready     agent     15m       v1.7.7
    aks-nodepool1-57028559-1   Ready     agent     14m       v1.7.7
    ```
    
    To see more details about your cluster: 
    
    ```bash
    $ kubectl cluster-info
    
    Kubernetes master is running at https://cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io:443
    Heapster is running at https://cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/heapster/proxy
    KubeDNS is running at https://cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    kubernetes-dashboard is running at https://cbus-aks-t-cbus-aks-trainin-2ee8d5-f1886e11.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
    ```

You should now have a Kubernetes cluster running with 2 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes. 
