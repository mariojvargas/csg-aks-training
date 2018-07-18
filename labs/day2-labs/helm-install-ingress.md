# Installing Nginx Ingress Using Helm

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

If you arrived here from the *Ingress Controllers*, [return to that lab now](ingress-controller.md) to continue.