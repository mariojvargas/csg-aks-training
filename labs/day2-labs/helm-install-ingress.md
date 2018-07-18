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
    $ kubectl get pods -n kube-system --selector "app=nginx-ingress"

    NAME                                                     READY     STATUS    RESTARTS   AGE
    ingress-nginx-ingress-controller-768fcd5669-w8lbn        1/1       Running   1          15h
    ingress-nginx-ingress-default-backend-5884c46f87-grgkw   1/1       Running   1          15h
    ```

    The nginx-ingress helm chart deploys a Nginx ingress controller and also a backend for the ingress controller. The backend is used when a route is not found and will display a 404 error. You can browse to the public IP to preview this.

    ``` bash
    $ kubectl get svc -n kube-system -o wide --selector "app=nginx-ingress"

    NAME                                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE       SELECTOR
    ingress-nginx-ingress-controller        LoadBalancer   10.0.77.71     52.179.211.89   80:32319/TCP,443:30487/TCP   20d       app=nginx-ingress,component=controller,release=ingress
    ingress-nginx-ingress-default-backend   ClusterIP      10.0.232.223   <none>          80/TCP                       20d       app=nginx-ingress,component=default-backend,release=ingress
    ```

    The Nginx controller will use a LoadBalancer type service where the backend is of type ClusterIP.

If you arrived here from the *Ingress Controllers*, [return to that lab now](ingress-controller.md) to continue.

## In Case You Cannot Install Nginx Ingress

If after executing the `helm install` command shown above, you receive an error such as the one below, RBAC is probably not enabled in your environment. If that is the case, follow the steps shown below to recover and reinstall Nginx Ingress.

### Error Message
```text
Error: release ingress failed: clusterroles.rbac.authorization.k8s.io "ingress-nginx-ingress" is forbidden: attempt to grant extra privileges: [PolicyRule{Resources:["configmaps"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["configmaps"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["endpoints"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["endpoints"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["nodes"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["nodes"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["pods"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["pods"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["secrets"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["secrets"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["nodes"], APIGroups:[""], Verbs:["get"]} PolicyRule{Resources:["services"], APIGroups:[""], Verbs:["get"]} PolicyRule{Resources:["services"], APIGroups:[""], Verbs:["list"]} PolicyRule{Resources:["services"], APIGroups:[""], Verbs:["update"]} PolicyRule{Resources:["services"], APIGroups:[""], Verbs:["watch"]} PolicyRule{Resources:["ingresses"], APIGroups:["extensions"], Verbs:["get"]} PolicyRule{Resources:["ingresses"], APIGroups:["extensions"], Verbs:["list"]} PolicyRule{Resources:["ingresses"], APIGroups:["extensions"], Verbs:["watch"]} PolicyRule{Resources:["events"], APIGroups:[""], Verbs:["create"]} PolicyRule{Resources:["events"], APIGroups:[""], Verbs:["patch"]} PolicyRule{Resources:["ingresses/status"], APIGroups:["extensions"], Verbs:["update"]}] user=&{system:serviceaccount:kube-system:default 83206e96-893d-11e8-a230-5af9fec2a633 [system:serviceaccounts system:serviceaccounts:kube-system system:authenticated] map[]} ownerrules=[] ruleResolutionErrors=[]
```

### Recovery Steps

```bash
$ helm delete ingress --purge
release "ingress" deleted

$ helm install --name ingress stable/nginx-ingress \
   --namespace kube-system --set rbac.create=false
```

You should see the following output once Nginx Ingress is installed in the Kubernetes cluster.

```text
NAME:   ingress
LAST DEPLOYED: Wed Jul 18 11:34:51 2018
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/PodDisruptionBudget
NAME                                   MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
ingress-nginx-ingress-controller       1              N/A              0                    2s
ingress-nginx-ingress-default-backend  1              N/A              0                    2s

==> v1/Pod(related)
NAME                                                    READY  STATUS             RESTARTS  AGE
ingress-nginx-ingress-controller-3612824525-srgr8       0/1    ContainerCreating  0         2s
ingress-nginx-ingress-default-backend-3666368176-0ctx2  0/1    ContainerCreating  0         2s

==> v1/ConfigMap
NAME                              DATA  AGE
ingress-nginx-ingress-controller  1     2s

==> v1/ServiceAccount
NAME                   SECRETS  AGE
ingress-nginx-ingress  1        2s

==> v1/Service
NAME                                   TYPE          CLUSTER-IP   EXTERNAL-IP  PORT(S)                     AGE
ingress-nginx-ingress-controller       LoadBalancer  10.0.160.54  <pending>    80:31214/TCP,443:31352/TCP  2s
ingress-nginx-ingress-default-backend  ClusterIP     10.0.149.76  <none>       80/TCP                      2s

==> v1beta1/Deployment
NAME                                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
ingress-nginx-ingress-controller       1        1        1           0          2s
ingress-nginx-ingress-default-backend  1        1        1           0          2s


NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace kube-system get services -o wide -w ingress-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

The `--set` switch interactively configures the values of a Helm Chart. For more information on possible options for the Nginx Ingress Chart, [see their GitHub repository](https://github.com/kubernetes/charts/tree/master/stable/nginx-ingress).
