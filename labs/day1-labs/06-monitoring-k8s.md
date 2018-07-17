# Add Monitoring to an Azure Kubernetes Service Cluster

There are a number of monitoring solutions available today. Here is a quick, but not exhaustive list for reference purposes:
* Datadog
* Sysdig
* Elastic Stack
* Splunk
* Operations Management Suite
* Prometheus

For the purposes of this lab we will be focusing in on [Prometheus](https://prometheus.io/) and using [Grafana](https://grafana.com/) to provide a visual Dashboard of our Azure Kubernetes Service Cluster.

## Install Helm

We are going to be installing Prometheus and Grafana into our K8s cluster using Helm and Tiller. You can think of Helm as a package manager for Kubernetes with Tiller being the server-side component.

1. Initialize Helm by running `helm init`

    ```bash
    $ helm init

    $HELM_HOME has been configured at /Users/mvargas/.helm.

    Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

    Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
    For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
    Happy Helming!
    ```

2. Validate Helm and Tiller were installed successfully

    ```bash
    $ helm version

    Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
    Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
    ```

## Install Prometheus using Helm

Prometheus is a Cloud Native Computing Foundation (CNCF) project used to collect and process metrics. It collects metrics from configured targets, in our case it is a Kubernetes Cluster.

1. Install Prometheus using Helm CLI

    Switch to the `helper-files` directory and view the `prometheus-configforhelm.yaml` file. This configures Helm to install Prometheus with our desired settings.

    ```bash
    $ cd ~/blackbelt-aks-hackfest/labs/helper-files
    # The following command will install Prometheus into the K8s cluster using custom settings

    $ helm install --name gbbhackprometheus stable/prometheus \
        --version 4.6.13 -f prometheus-configforhelm.yaml
    ```

2. Validate that Prometheus was Installed. Sample output shown.
    ``` bash
    $ kubectl get pods | grep prometheus

    gbbhackprometheus-prometheus-kube-state-metrics-5b9f4d9d9-vctrx   1/1       Running   0          3m
    gbbhackprometheus-prometheus-node-exporter-v6frn                  1/1       Running   0          3m
    gbbhackprometheus-prometheus-server-54f5bcb797-sbzsp              2/2       Running   0          3m
    ```

    List out services associated with Prometheus. This is a cheap way to accomplish this using grep. Sample shown.

    ```bash
    $ kubectl get service | grep prometheus

    gbbhackprometheus-prometheus-kube-state-metrics   ClusterIP      None           <none>          80/TCP           3m
    gbbhackprometheus-prometheus-node-exporter        ClusterIP      None           <none>          9100/TCP         3m
    gbbhackprometheus-prometheus-server               LoadBalancer   10.0.212.145   52.168.100.25   9090:32340/TCP   3m
    ```

## Install Grafana
Grafana is a dashboard visualization tool that can use all kinds of data sources. In our case, Prometheus will be used as the data source.

1. Install Grafana using Helm CLI
    The following command will install Grafana into the K8s cluster with a few custom settings to make it easier to access.

    * We are setting the default username and password to **admin** to make it easier to remember
    * We are also setting the service type to **LoadBalancer** to expose the service outside of the cluster and make it accessible via the Internet. Note that the username and password are being set via the command line.

    ```bash
    $ helm install --name gbbhackgrafana stable/grafana --version 0.5.1 \
        --set server.service.type=LoadBalancer,server.adminUser=admin,server.adminPassword=admin,server.image=grafana/grafana:latest,server.persistentVolume.enabled=false
    ```

2. Validate that Grafana was Installed.
    ```bash
    $ kubectl get pods | grep grafana

    hgrafana-grafana-855db78dc4-pnzth                           1/1       Running   0          2h
    ```

    List out services containing the word "grafana". Please note that the external IP address may take several minutes to create.

    ```bash
    $ kubectl get svc | grep grafana

    khgrafana-grafana                            LoadBalancer   10.0.163.226   "52.226.75.38"     80:31476/TCP   2h
    ```

3. Test that the Grafana UI comes up. Use the EXTERNAL-IP value from the previous step and put that into your browser:
    * eg. http://52.226.75.38/, EXTERNAL-IP column from above. You should see something like the following come up, be patient it will take a moment or two:

    ![alt text](img/8-grafana_default.png "Grafana Login Page")

## Setting up Grafana

1. Log into Grafana Dashboard using **admin** for the username and password
    * You should see something like the following:

    ![alt text](img/8-grafana_loggedin.png "Grafana landing page")

2. Add Prometheus as a Data Source
    * If you recall from above, we exposed a number of K8s services, one of those services was the Prometheus Server. We are going to use that Service endpoint in our Data Service configuration. The Add Data Source screen should look something like the below screen shot.

    > Use `http://gbbhackprometheus-prometheus-server:9090` for the URL in the HTTP settings. Note that `gbbhackprometheus-prometheus-server` is the name of the Prometheus service within Kubernetes. Kubernetes uses the KubeDNS service to resolve service names to their respective internal cluster IP addresses.

    ![alt text](img/8-grafana_datasource.png "Setting Prometheus as a Data Source in Grafana")

3. Validate Prometheus Data Source
    * Once you have filled in the values similar to the screenshot above, click the **Add** button and ensure no errors come back.

4. Add K8s Monitoring Dashboard to Grafana
    * After the datasource has been added, it is now time to add a dashboard. Grafana dashboards can be shared on Grafana.com. Go to import dashboards via the menu in the top left.

    ![alt text](img/8-grafana_dashboardimport.png "Import Dashboard Menu")

    * Click on the **Upload File** button and browse to the `grafana-dashboard.json` in the `helper-files` directory. You can also paste the contents of the json into the text box.

    ![alt text](img/8-grafana_dashboardid.png "Import Dashboard Modal Window")

    * Set the datasource dropdown to the "AKSPrometheus" that was created in the previous step. 

    ![alt text](img/8-grafana_dashboardsave.png "Import Dashboard Modal Window with Populated Fields")

    * Click the **Import** button.

    ![alt text](img/8-grafana_k8sdashboard.png "Grafana Dashboard")

    You should now have Prometheus and Grafana running in your Azure Kubernetes Service cluster and be able to see the Grafana Dashboard.
