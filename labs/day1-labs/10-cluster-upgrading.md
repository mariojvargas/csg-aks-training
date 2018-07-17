# Upgrade an Azure Kubernetes Service (AKS) cluster

Azure Container Service (AKS) makes it easy to perform common management tasks including upgrading Kubernetes clusters.

## Upgrade an AKS cluster

Before upgrading a cluster, use the `az aks get-upgrades` command to check which Kubernetes releases are available for upgrade.

> **NOTE:**  This operation could take approximately 1 hour!

```bash
$ CLUSTER_NAME=cbus-aks-training
$ RG_NAME=cbus-aks-training

$ az aks get-upgrades --name $CLUSTER_NAME --resource-group $RG_NAME --output table

Name     ResourceGroup      MasterVersion    NodePoolVersion    Upgrades
-------  -----------------  ---------------  -----------------  ---------------------------------------------------------------------------------
default  cbus-aks-training  1.7.7            1.7.7              1.7.9, 1.7.12, 1.7.15, 1.7.16, 1.8.1, 1.8.2, 1.8.6, 1.8.7, 1.8.10, 1.8.11, 1.8.14
```

We have 11 versions available for upgrade: 1.7.9, 1.7.12, 1.7.15, 1.7.16, 1.8.1, 1.8.2, 1.8.6, 1.8.7, 1.8.10, 1.8.11, and 1.8.14. We can use the `az aks upgrade` command to upgrade to the latest available version.  During the upgrade process, nodes are carefully [cordoned and drained](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#maintenance-on-a-node) to minimize disruption to running applications.  Before initiating a cluster upgrade, ensure that you have enough additional compute capacity to handle your workload as cluster nodes are added and removed.

```bash
$ az aks upgrade --name $CLUSTER_NAME \
  --resource-group $RG_NAME --kubernetes-version 1.8.14
```

Output:

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myK8sCluster",
  "location": "eastus",
  "name": "myK8sCluster",
  "properties": {
    "accessProfiles": {
      "clusterAdmin": {
        "kubeConfig": "..."
      },
      "clusterUser": {
        "kubeConfig": "..."
      }
    },
    "agentPoolProfiles": [
      {
        "count": 1,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "myK8sCluster",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D2_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "myK8sClust-myResourceGroup-4f48ee",
    "fqdn": "myk8sclust-myresourcegroup-4f48ee-406cc140.hcp.eastus.azmk8s.io",
    "kubernetesVersion": "1.8.14",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "..."
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "myResourceGroup",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

You can now confirm the upgrade was successful with the `az aks show` command.

```azurecli-interactive
az aks show --name $CLUSTER_NAME --resource-group $RG_NAME --output table
```

Output:

```json
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myK8sCluster  eastus     myResourceGroup  1.8.14                Succeeded            myk8sclust-myresourcegroup-3762d8-2f6ca801.hcp.eastus.azmk8s.io
```

## Attribution:
Content originally created by @gabrtv et al. from [this Azure Doc](https://docs.microsoft.com/en-us/azure/aks/upgrade-cluster)
