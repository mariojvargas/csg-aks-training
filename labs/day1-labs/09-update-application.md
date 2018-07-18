# Update and Deploy New Version of Application

In this lab, we will make a change to the web application and then re-deploy the new container image into AKS. 

## Update web application code

1. Use the editor of your choice and browse to `~/cbus-aks-training/app/web/src/components/`
2. Edit code for the `Footer.vue`
3. Find the snippet below *(line 13)* and change the text _"Azure Global Blackbelt Team"_ to your name or whatever you would like to display.

    ```html
    <div class="row at-row flex-center flex-middle">
      <div class="col-lg-6">
      </div>
      <div class="col-lg-12 credits">
        Cardinal Solutions Blackbelt Team 2018-07-18
      </div>
      <div class="col-lg-6">
      </div>
    </div>
    ```

4. Save your edits and close the file

## Create new container image and push to ACR

1. Browse to `~/cbus-aks-training/app/web`
2. You should still have a Dockerfile created in an earlier lab
3. Create a new image with an updated image tag

    ```bash
    $ docker build -t rating-web:v1.0.1 .
    ```

4. Tag the new image and push to your Azure Container Registry

    ```bash
    # you may need to login again to your ACR. set these values to yours
    $ ACR_SERVER=mvargasacr.azurecr.io
    $ ACR_USER=mvargasacr
    $ ACR_PASSWORD=Supersecret+base64String

    $ docker login --username $ACR_USER --password $ACR_PASSWORD $ACR_SERVER

    $ docker tag rating-web:v1.0.1 \
        $ACR_SERVER/azureworkshop/rating-web:v1.0.1

    $ docker push $ACR_SERVER/azureworkshop/rating-web:v1.0.1
    ```

5. Verify image tagged `v1.0.1` was pushed to ACR by checking your registry using either the Azure portal or the AZ CLI. AZ CLI command shown below for ACR named `mvargasacr`. Replace this name with your ACR instance name.

    ```bash
    $ az acr repository show-tags -g cbus-aks-training \
        -n mvargasacr --repository azureworkshop/rating-web -o table

    Result
    --------------
    v1
    v1.0.1
    ```

## Update kubernetes deployment

There are two ways to update the application with the new version. Both are described below. Choose one to proceed.
1. Edit the YAML file and re-apply
2. Update the deployment and re-set the image tag

### Edit YAML and apply

1. As we did in a prior lab, open the  `helper-files` directory and review the file `heroes-web-api.yaml`
2. Update the yaml file and replace the tag from `v1` to `v1.0.1`. Be sure to use the correct name for the ACR server. The name `mvargasacr.azurecr.io` is being used in the snippet below.

    ```yaml
    spec:
    containers:
    - image: mvargasacr.azurecr.io/azureworkshop/rating-web:v1.0.1
        name:  heroes-web-cntnr
    ```

3. Apply the new yaml file

    ```bash
    $ cd ~/cbus-aks-training/labs/helper-files

    $ kubectl apply -f heroes-web-api.yaml
    ```

### Update the deployment

1. Set the new image tag on the deployment object
    ```bash
    $ kubectl get deployments --namespace mvargas

    NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    heroes-api-deploy   1         1         1            1           29m
    heroes-db-deploy    1         1         1            1           32m
    heroes-web-depoy    1         1         1            1           29m

    $ kubectl set image deployment/heroes-web-deploy \
        heroes-web-cntnr=$ACR_SERVER/azureworkshop/rating-web:v1.0.1 \
        --namespace mvargas

    # Output from kubectl set image
    deployment.apps "heroes-web-deploy" image updated
    ```

## Check status

1. You can see the updates and history for the changes from above

    ```bash
    # this command will verify that latest deployment was successful
    $ kubectl rollout status deployment/heroes-web-deploy --namespace mvargas

    # each deployment creates a new replicaset
    $ kubectl get replicaset --namespace mvargas

    heroes-web-556f6f976c          0         0         0         34m
    heroes-web-64f4795689          0         0         0         8m
    heroes-web-67b4b7b887          1         1         1         1m

    $ kubectl rollout history deployment/heroes-web-deploy --namespace mvargas
    ```

## Browse to your newly deployed web application

Now navigate to the external IP address of the web application and verify that your changes are in place.
