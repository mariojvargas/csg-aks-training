# Deploy the Superhero Ratings App to AKS

## Review/Edit the YAML Config Files

1. In your favorite editor open the file `~/repos/cbus-aks-training/labs/helper-files/heroes-db.yaml`.

    * Review the yaml file and learn about some of the settings
    * Update the yaml file for the proper container image name
    * Update the yaml file with a `namespace` of your choice. For example, you will find the placeholder `<namespace-name>`. Replace it with your first initial and your lastname (no spaces; use dashes). For example: `namespace: mvargas`. This placeholder is located in two places.
    * You will need to replace the `<login server>` placeholder with the ACR login server created in lab 2
    * Example:

        ```yaml
        spec:
        containers:
        - image: mycontainerregistry.azurecr.io/azureworkshop/rating-db:v1
            name:  heroes-db-cntnr
        ```

2. Now edit the file `~/repos/cbus-aks-training/labs/helper-files/heroes-web-api.yaml`

    * Review the yaml file and learn about some of the settings. Note the environment variables that allow the services to connect
    * Again, update the yaml file with a `namespace` of your choice. For example, you will find the placeholder `<namespace-name>`. Replace it with your first initial and your lastname (no spaces; use dashes). For example: `namespace: mvargas`. This placeholder is located in four places.
    * Update the yaml file for the proper container image names.
    * You will need to replace the `<login server>` with the ACR login server created in lab 2
        > Note: You will update the image name ***twice*** updating the web and api container images.

    * Example:

        ```yaml
        spec:
        containers:
        - image: mycontainerregistry.azurecr.io/azureworkshop/rating-web:v1
            name:  heroes-web-cntnr
        ```

## Create a Kubernetes Secret with Access to Azure Container Registry

There are a few ways that AKS clusters can access your private Azure Container Registry. Generally the service account that Kubernetes utilizes will have rights based on its Azure credentials. In our lab config, we must create a secret to allow this access.

We're making use of namespaces in our Kubernetes manifests. A namespace is a construct for grouping together related services and deployments. In the YAML files above, there is a section called `imagePullSecrets`, containing the name of the Kubernetes secret `acr-secret`, which will be created below. It is important for this secret to be created in the same namespace as the manifests above.

We assume the secret and all the deployments from here on out will be located in a namespace following the format `first-initial-plus-lastname`, such as `mvargas`

```bash
# set these values to yours
ACR_SERVER=youracrserver.azurecr.io
ACR_USER=youracrusername
ACR_PASSWORD=ASuperSecretBase64String

$ kubectl create secret docker-registry acr-secret \
    --namespace mvargas \
    --docker-server=$ACR_SERVER --docker-username=$ACR_USER \
    --docker-password=$ACR_PASSWORD --docker-email=superman@heroes.com
```

> Note: You can review the `heroes-db.yaml` and `heroes-web-api.yaml` to see where the `imagePullSecrets` are configured.

## Deploy database container to AKS

* Use the kubectl CLI to deploy each app
    ```bash
    $ cd ~/repos/cbus-aks-training/labs/helper-files

    $ kubectl apply -f heroes-db.yaml
    ```

* Get MongoDB pod name and store it in a variable

    ```bash
    $ kubectl get pods --namespace mvargas

    NAME                                 READY     STATUS    RESTARTS   AGE
    heroes-db-deploy-2357291595-k7wjk    1/1       Running   0          3m

    $ MONGO_POD=heroes-db-deploy-2357291595-k7wjk
    ```

* Import data into MongoDB using `./import.sh` script

    ```bash
    # ensure the pod name variable is set to your pod name
    # once you exec into pod, run the `import.sh` script

    $ kubectl exec -it --namespace mvargas $MONGO_POD bash
    ```

    Now in the pod, run the `./import.sh` script. Once the script finishes executing, exit the pod by entering `exit`.

    ```text
    root@heroes-db-deploy-2357291595-xb4xm:/# ./import.sh

    2018-01-16T21:38:44.819+0000	connected to: localhost
    2018-01-16T21:38:44.918+0000	imported 4 documents
    2018-01-16T21:38:44.927+0000	connected to: localhost
    2018-01-16T21:38:45.031+0000	imported 72 documents
    2018-01-16T21:38:45.040+0000	connected to: localhost
    2018-01-16T21:38:45.152+0000	imported 2 documents

    root@heroes-db-deploy-2357291595-xb4xm:/# exit

    # be sure to exit pod as shown above
    ```

## Deploy the web and api containers to AKS

Use the `kubectl` CLI to deploy both the web and API apps (combined into one YAML file).

```bash
$ cd ~/repos/cbus-aks-training/labs/helper-files

$ kubectl apply -f heroes-web-api.yaml
```

## Validate

* Check to see if pods are running in your cluster
    ```bash
    $ kubectl get pods --namespace mvargas

    NAME                                 READY     STATUS    RESTARTS   AGE
    heroes-api-deploy-1140957751-2z16s   1/1       Running   0          2m
    heroes-db-deploy-2357291595-k7wjk    1/1       Running   0          3m
    heroes-web-1645635641-pfzf9          1/1       Running   0          2m
    ```

* Check to see if services are deployed.
    ```bash
    $ kubectl get service --namespace mvargas

    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    api          LoadBalancer   10.0.20.156   52.176.104.50    3000:31416/TCP   5m
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          12m
    mongodb      ClusterIP      10.0.5.133    <none>           27017/TCP        5m
    web          LoadBalancer   10.0.54.206   52.165.235.114   8080:32404/TCP   5m
    ```

* Browse to the External IP for your web application (on port 8080) and try the app
    > The public IP can take several minutes to create with a new cluster.
