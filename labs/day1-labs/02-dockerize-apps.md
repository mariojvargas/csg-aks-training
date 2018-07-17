# Dockerizing Applications

## Build Container Images

For the first container, we will be creating a Dockerfile from scratch. For the other containers, the Dockerfiles are provided.

### Web Container

1. Create a Dockerfile

    * In the `~/repos/cbus-aks-training/app/web` directory, add a file called `Dockerfile`
        * If you are in an SSH session, use Vim as the editor
        * In RDP or in a *nix desktop environment, you can use Visual Studio Code

    * Add the following lines and save:

        ```Docker
        FROM node:9.4.0-alpine

        ARG VCS_REF
        ARG BUILD_DATE
        ARG IMAGE_TAG_REF

        ENV GIT_SHA $VCS_REF
        ENV IMAGE_BUILD_DATE $BUILD_DATE
        ENV IMAGE_TAG $IMAGE_TAG_REF

        WORKDIR /usr/src/app
        COPY package*.json ./
        RUN npm install

        COPY . .
        RUN apk --no-cache add curl
        EXPOSE 8080

        CMD [ "npm", "run", "container" ]
        ```

2. Create a container image for the Node.js Web app. From the terminal session, type the following commands. Please remember to supply the "`.`" at the end of the `docker build` command, as this indicates the current working directory: 

    ```bash
    $ cd ~/repos/cbus-aks-training/app/web
    
    $ docker build \
        --build-arg BUILD_DATE=`date -u +"%Y-%m-%dT%H:%M:%SZ"` \
        --build-arg VCS_REF=`git rev-parse --short HEAD` \
        --build-arg IMAGE_TAG_REF=v1 \
        -t rating-web .
    ```

3. Validate the image was created with `docker images`

### API Container

In this step, the Dockerfile has been created for you. Feel free to inspect it.

1. Create a container image for the Node.js API app

    ```bash
    $ cd ~/repos/cbus-aks-training/app/api

    $ cat Dockerfile

    $ docker build -t rating-api .
    ```

2. Validate image was created with `docker images`

### MongoDB Container

1. Create a MongoDB image with data files

    ```bash
    $ cd ~/repos/cbus-aks-training/app/db

    $ docker build -t rating-db .
    ```

2. Validate image was created with `docker images`


## Run Containers

### Setup Docker Network

Create a docker bridge network to allow the containers to communicate internally. 

```bash
$ docker network create --subnet=172.18.0.0/16 my-network
```

### MongoDB Container

1. Run mongo container

    ```bash
    $ docker run -d --name db --net my-network \
        --ip 172.18.0.10 -p 27017:27017 rating-db
    ```

2. Validate by running `docker ps -a`

3. Import data into database

    ```bash
    $ docker exec -it db bash
    ```

    You will have a prompt inside the MongoDB container. From that prompt, run the import script (`./import.sh`). Below is an example output after running the import script.

    ```
    root@61f9894538d0:/# ./import.sh
    2018-01-10T19:26:07.746+0000	connected to: localhost
    2018-01-10T19:26:07.761+0000	imported 4 documents
    2018-01-10T19:26:07.776+0000	connected to: localhost
    2018-01-10T19:26:07.787+0000	imported 72 documents
    2018-01-10T19:26:07.746+0000	connected to: localhost
    2018-01-10T19:26:07.761+0000	imported 2 documents
    ```

4. Type `exit` to exit out of container

### API Container

1. Run api app container

    ```bash
    $ docker run -d --name api \
        -e "MONGODB_URI=mongodb://172.18.0.10:27017/webratings" \ 
        --net my-network --ip 172.18.0.11 -p 3000:3000 rating-api
    ```

    > Note that environment variables are used here to direct the api app to mongo.

2. Validate by running `docker ps -a`

3. Test api app with curl
    ```bash
    $ curl http://localhost:3000/api/heroes
    ```

### Web Container

1. Run web app container

    ```bash
    $ docker run -d --name web -e "API=http://172.18.0.11:3000/" \
        --net my-network --ip 172.18.0.12 -p 8080:8080 rating-web
    ```

2. Validate by running `docker ps -a`

3. Test web app by navigating to [http://localhost:8080/](http://localhost:8080/) in your Web browser (assuming your *nix has a desktop environment). Otherwise, you can also test via curl:

    ```bash
    $ curl http://localhost:8080
    ```

## Azure Container Registry (ACR)

Now that we have container images for our application components, we need to store them in a secure, central location. In this lab we will use [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) for this.

### Create Azure Container Registry instance

Creating an ACR instance is very simple. You can accomplish this via the Azure Portal or from the command line. We'll be using the AZ CLI.

1. Login to Azure if you aren't already logged in:

    ```bash
    $ az login
    ```

2. Once logged in to Azure with the AZ CLI, you're now ready to create an ACR instance. For the ACR instance name, please use your name in the form "first initial last name" followed by the "acr" suffix. For example, `mvargasacr`. This ACR instance will be using the Basic tier and be located in East US. Note we're creating it in our resource group named `cbus-aks-training`.

    ```bash
    $ az acr create --name <your-acr-name> \
        --resource-group cbus-aks-training \
        --location eastus --sku Basic
    ```

    Once the ACR instance is created, information about the ACR will be returned in JSON format. This contains valuable details, such as the login server name. Below is an example output:

    ```json
    {
        "adminUserEnabled": false,
        "creationDate": "2018-07-17T13:29:27.296322+00:00",
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cbus-aks-training/providers/Microsoft.ContainerRegistry/registries/mvargasacr",
        "location": "eastus",
        "loginServer": "mvargasacr.azurecr.io",
        "name": "mvargasacr",
        "provisioningState": "Succeeded",
        "resourceGroup": "cbus-aks-training",
        "sku": {
            "name": "Basic",
            "tier": "Basic"
        },
        "status": null,
        "storageAccount": null,
        "tags": {},
        "type": "Microsoft.ContainerRegistry/registries"
    }    
    ```

3. Next, we need to enable Admin access to our ACR instance. This will allow us to retrieve the ACR credentials via the command line. In the output JSON, verify that the `adminUserEnabled` property is set to `true`. You should reset this value to `false` later.

    ```bash
    $ az acr update -n <your-acr-name> --admin-enabled true
    ```

### Login to your ACR with Docker

Now, in order for us to push our Docker images to ACR, we need to be able to login to ACR with Docker. You can achieve this by going to the Azure Container Registry in the Azure Portal, but we'll opt to accomplish this with the AZ CLI.

1. Show the ACR credentials. The AZ CLI will return them in JSON format. Note that the username is the same as your ACR name. The AZ ACR command will display two passwords. You can use either one.

    ```bash
    $ az acr credential show --name <your-acr-name> --resource-group cbus-aks-training

    {
    "passwords": [
        {
        "name": "password",
        "value": "Supersecret+base64String"
        },
        {
        "name": "password2",
        "value": "Supersecret=base64String"
        }
    ],
    "username": "mvargasacr"
    }

    ```
4. In the terminal session, set each value to a variable as shown below. Your ACR name will probably be different, depending on which name you used above to create it. We'll be using `mvargasacr` in this example:

    ```bash
    $ ACR_SERVER=mvargasacr.azurecr.io
    $ ACR_USER=mvargasacr
    $ ACR_PASSWORD=Supersecret+base64String

    $ docker login --username $ACR_USER --password $ACR_PASSWORD $ACR_SERVER

    # Output from the above command. Ignore the warning shown
    WARNING! Using --password via the CLI is insecure. Use --password-stdin.
    Login Succeeded
    ```

### Tag images with ACR server and repository 

In the same terminal session, since we stored the ACR login server in the environment variable `ACR_SERVER`, enter the following. The name `azureworkshop` is arbitrary.

```bash
$ docker tag rating-db $ACR_SERVER/azureworkshop/rating-db:v1

$ docker tag rating-api $ACR_SERVER/azureworkshop/rating-api:v1

$ docker tag rating-web $ACR_SERVER/azureworkshop/rating-web:v1
```

### Push images to registry

In the same terminal sessions, let's push our Docker images to ACR.

```bash
$ docker push $ACR_SERVER/azureworkshop/rating-db:v1

$ docker push $ACR_SERVER/azureworkshop/rating-api:v1

$ docker push $ACR_SERVER/azureworkshop/rating-web:v1
```

Output from a successful `docker push` command is similar to:

```
The push refers to a repository [mvargasacr.azurecr.io/azureworkshop/rating-db]
035c23fa7393: Pushed
9c2d2977a0f4: Pushed
d7b18f71e002: Pushed
ec41608c0258: Pushed
ea882d709aca: Pushed
74bae5e77d80: Pushed
9cc75948c0bd: Pushed
fc8be3acfc2d: Pushed
f2749fe0b821: Pushed
ddad740d98a1: Pushed
eb228bcf2537: Pushed
dbc5f877c367: Pushed
cfce7a8ae632: Pushed
v1: digest: sha256:f84eba148dfe244f8f8ad0d4ea57ebf82b6ff41f27a903cbb7e3fbe377bb290a size: 3028
```

### Validate images in Azure

There are two ways to validate that the images are indeed in Azure. You can verify their creation from within the Azure Portal (Resource Groups > cbus-aks-training > your-acr-name > repositories, and then click on each repository). An example screen shot is shown below. However, the cool kids prefer the AZ CLI. Let's do that.

1. List out repositories in ACR:

    ```bash
    $ az acr repository list --resource-group cbus-aks-training \
        --name <your-acr-name> --output table

    Result
    ------------------------
    azureworkshop/rating-api
    azureworkshop/rating-db
    azureworkshop/rating-web
    ```

2. Pick one of the above repositories. Let's list out the tags it has. For example, we're going to list the tags associated with the `azureworkshop/rating-api` repository:

    ```bash
    $ az acr repository show-tags -g cbus-aks-training \
        -n <your-acr-name> --repository azureworkshop/rating-api \
        -o table

    Result
    --------
    v1
    ```

#### Viewing Docker repositories in Azure Container Registry

![alt text](img/acr-repository-images.png "Example Repository Images in ACR")
