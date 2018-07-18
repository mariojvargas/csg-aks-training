# Ingress Controllers

This lab will show how to use Kubernetes ingress controllers to route traffic to our Heroes web application.

## Add Ingress Controller to an Azure Kubernetes Service Cluster

There are a number of ingress controllers available today. Here is a quick, but not exhaustive list for reference purposes:

* [Nginx](https://github.com/kubernetes/ingress-nginx)
* [Traefik](https://traefik.io/)
* [Linkerd](https://linkerd.io/)
* HAProxy
* Custom Ingress Controllers

For the purposes of this lab we will be using Nginx as our ingress controller.

## Configure Helm

We will use Helm to install Nginx. We had configured Helm in prior labs.

1. Validate your Helm install by running the below commands.

    ``` bash
    $ helm version

    Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
    Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
    ```

    > Note: If helm was not configured, you must run `helm init`

## Install Nginx using Helm

The [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/) (or visit [Helm Chart](https://github.com/kubernetes/charts/tree/master/stable/nginx-ingress)) is an Ingress controller that uses a ConfigMap to store the Nginx configuration and provides [layer 7 capabilities](https://en.wikipedia.org/wiki/OSI_model) for applications deployed on Kubernetes.

1. Install Nginx using Helm CLI. This will install the Nginx Ingress Controller into the K8s cluster under the `kube-system` namespace, which is created and reserved for internal Kubernetes services, such as `kube-dns`, `kube-proxy`, etc.  

    ``` bash
    $ helm install --name ingress stable/nginx-ingress --namespace kube-system
    ```

2. Validate that Nginx was installed

    ``` bash
    $ kubectl get pods -n kube-system | grep nginx

    ingress-nginx-ingress-controller-86bf69bcfc-jqvsg        1/1       Running   0          1d
    ingress-nginx-ingress-default-backend-86d6db4c47-td2k8   1/1       Running   0          1d
    ```

    The nginx-ingress helm chart deploys a Nginx ingress controller and also a backend for the ingress controller. The backend is used when a route is not found and will display a 404 error. You can browse to the public IP to preview this.

    ``` bash
    $ kubectl get svc -n kube-system | grep nginx

    ingress-nginx-ingress-controller       LoadBalancer  10.0.231.143  52.173.190.190  80:30910/TCP,443:30480/TCP  1d
    ingress-nginx-ingress-default-backend  ClusterIP     10.0.175.123  <none>          80/TCP                      1d
    ```

    The Nginx controller will use a LoadBalancer type service where the backend is of type ClusterIP.

## Deploy the heroes Web and API apps with Ingress

We will now deploy the application with a configured Ingress resource.

1. Clear anything out of your cluster by deleting your deployments

    ```bash
    $ kubectl delete -f heroes-db.yaml

    $ kubectl delete -f heroes-web-api.yaml
    ```

2. Switch to the `helper-files` directory and view the
   `heroes-web-api-ingress.yaml` file.

    ``` bash
    $ cd ~/repos/csg-aks-training/labs/helper-files

    $ cat heroes-web-api-ingress.yaml
    ```

3. Change all image field in the YAML files to match your docker registry url.

    * Update the yaml files for the proper container image names.
    * Update all instances of `namespace: <namespace-name>` to match your first initial and last name. For example: `namespace: mvargas`.
    * You will need to replace the `<login server>` with the ACR login server created in earlier labs.
        > Note: You will update the image name ***twice*** in the Web and API container images and ***once*** in the database container image.

        **Example:**

        ```yaml
        spec:
        containers:
        - image: mycontainerregistry.azurecr.io/azureworkshop/rating-web:v1
            name:  heroes-web-cntnr
        ```

4. Deploy the heroes-db.yaml manifest and import the data

    ``` bash
    $ kubectl apply -f heroes-db.yaml
    ```

5. Connect into the pod and import the hero data

    ```bash
    $ kubectl get pods --namespace mvargas

    NAME                                 READY     STATUS    RESTARTS   AGE
    heroes-db-deploy-2357291595-k7wjk    1/1       Running   0          3m

    $ MONGO_POD=heroes-db-deploy-2357291595-k7wjk

    $ kubectl exec -it --namespace mvargas $MONGO_POD bash

    # Execute the ./import.sh file
    root@heroes-db-deploy-2357291595-xb4xm:/# ./import.sh

    2018-01-16T21:38:44.819+0000	connected to: localhost
    2018-01-16T21:38:44.918+0000	imported 4 documents
    2018-01-16T21:38:44.927+0000	connected to: localhost
    2018-01-16T21:38:45.031+0000	imported 72 documents
    2018-01-16T21:38:45.040+0000	connected to: localhost
    2018-01-16T21:38:45.152+0000	imported 2 documents
    root@heroes-db-deploy-2357291595-xb4xm:/# exit
    ```

    Be sure to exit pod as shown in the last line above

6. Deploy heroes-web-api-ingress.yaml

    ``` bash
    $ kubectl apply -f heroes-web-api-ingress.yaml
    ```

    > Note: Below is an example of a Ingress configuration. Place special attention to the `rules:` section and how elements are indented, as it is one of the trickiest parts about configuring an Ingress. You can list out multiple paths, each pointing to a different `backend` if necessary. The `host:` can be set to an empty host or any valid domain name, such as `host: heroes.csgtraining.cardinalsolutions.com`.

    ```yaml
    ---
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: heroes-web-ingress
      namespace: mvargas
      annotations:
        kubernetes.io/ingress.class: nginx
        # Add to generate certificates for this ingress
        kubernetes.io/tls-acme: 'false'
    spec:
      rules:
        - host:
          http:
            paths:
              - path: /
                backend:
                  serviceName: web
                  servicePort: 8080
    ```

7. Browse to the web app via the Ingress controller by obtaining its public (external) IP address.

    ```bash
    $ kubectl get svc -n kube-system | grep ingress

    ingress-nginx-ingress-controller        LoadBalancer   10.0.155.205   52.186.29.245   80:32045/TCP,443:31794/TCP   2h
    ingress-nginx-ingress-default-backend   ClusterIP      10.0.171.59    <none>          80/TCP                       2h
    ```

8. Navigate to the external IP of the controller. In the output above, the public IP address is `52.186.29.245`, so we use that as our HTTP URL using port 80 (default): http://52.186.29.245/

    > Note: you will likely see a privacy SSL warning.

## A Quick Word on Ingress Controllers

Ingress controllers are typically accessible at ports 80 and 443 (SSL/TLS). Don't make the mistake of trying to change these, as you will encounter issues. In the output above, notice that the Ingress controller is exposed on ports 80 and 443, each mapping internally to ports 32045 and 31794, respectively. Those ports are internal to the cluster.

The Ingress configuration (YAML) depends highly on the `host:`, `path:`, and service names/ports you expose in your deployment's ingress configuration. This means that one single host can expose multiple public-facing API endpoints, where only their root paths are different.

For additional information on Kubernetes Ingress Controllers and on how they work, visit:

* [Kubenetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Kubernetes NodePort vs LoadBalancer vs Ingress? When should I use what?](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)

 Ingress controllers are an important concept to understand and you will certainly encounter them in many of your Kubernetes implementations.
