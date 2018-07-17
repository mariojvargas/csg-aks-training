# Kubernetes Dashboard

The Kubernetes dashboard is a Web utility that lets you view, monitor, and troubleshoot Kubernetes resources. 

> Note: The Kubernetes dashboard is a secured endpoint and can only be accessed using the SSH keys for the cluster.

### Accessing The Dashboard UI

There are multiple ways of accessing the Kubernetes dashboard. You can access it through the `kubectl` command-line interface using the `kubectl proxy` command, assuming the cluster is running on the local host, or through the master server API. We'll be using the AZ CLI, as it provides a secure connection that doesn't expose the UI to the Internet.

1. If you're not already logged in, log in to Azure using the AZ CLI: 

    ```bash
    $ az login
    ```

2. Once you're logged in to Azure, verify the resource group `cbus-aks-training` is listed. The funky query syntax you're seeing below is called [JMESPath](http://jmespath.org/)--*Believe me... It took me a while to figure out the proper query without recourse to `grep`. :-)*

    ```bash
    $ az group list --query "[?name=='cbus-aks-training']" --output table

    Name               Location
    -----------------  ----------
    cbus-aks-training  eastus
    ```

3. Prepare to be mesmerized. Now use the [AZ AKS Browse](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-browse) command to magically open the web browser on the local machine, amazingly opening the Kuberbetes Dashboard in your favorite Web browser. This will work only if your current Linux VM is running a desktop interface. The AKS Browse command uses a lot of black magic behind, called port forwarding, behind the scenes, exposing the Kubernetes Dashboard using a secure connection via the URL [http://127.0.0.1:8001/](http://127.0.0.1:8001/). Slow Clap.

    ```bash
    $ az aks browse --resource-group cbus-aks-training --name cbus-aks-training

    Merged "cbus-aks-training" as current context in /var/folders/jn/7s1wmzf54876vnbxcz3b1rpc6mz47k/T/tmpu24bzf2_
    Proxy running on http://127.0.0.1:8001/
    Press CTRL+C to close the tunnel...
    Forwarding from 127.0.0.1:8001 -> 9090
    Forwarding from [::1]:8001 -> 9090
    Handling connection for 8001
    ```

### Explore Kubernetes Dashboard

Please note that the screen shots below may differ from what you see in your cluster.

1. In the Kubernetes Dashboard select **Nodes** to view
![](img/ui_nodes.png)
2. Explore the different node properties available through the dashboard
3. Explore the different pod properties available through the dashboard ![](img/ui_pods.png)
4. In this lab feel free to take a look around other at  other resources Kubernetes provides through the dashboard

> To learn more about Kubernetes objects and resources, browse the documentation: <https://kubernetes.io/docs/user-journeys/users/application-developer/foundational/#section-3>
