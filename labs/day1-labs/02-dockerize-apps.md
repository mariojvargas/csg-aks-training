# Dockerizing Applications

## Build Container Images

For the first container, we will be creating a Dockerfile from scratch. For the other containers, the Dockerfiles are provided.

### Web Container

In the `~/repos/csg-aks-training/app/web` directory, there is already a Dockerfile created for you. Examine the Dockerfile by opening it or displaying its contents using `cat Dockerfile`. Its contents are shown below for your convenience. Notice the use of both build arguments (`ARG`) and ENVironment variables.

```Dockerfile
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

After examining the contents of the Node.js Web app's Dockerfile, let us now build the container image.

1. From the terminal session, open the `~/repos/csg-aks-training/app/web` directory. Then type in the below `docker build` command. Please remember to supply the "`.`" at the end of the `docker build` command, as this indicates to Docker it should build out of the current working directory:

    ```bash
    $ cd ~/repos/csg-aks-training/app/web

    $ docker build \
        --build-arg BUILD_DATE=`date -u +"%Y-%m-%dT%H:%M:%SZ"` \
        --build-arg VCS_REF=`git rev-parse --short HEAD` \
        --build-arg IMAGE_TAG_REF=v1 \
        -t rating-web .
    ```

2. Validate the image was created with `docker images`

### API Container

In this step, the Dockerfile has been created for you. Feel free to inspect it.

1. Create a container image for the Node.js API app

    ```bash
    $ cd ~/repos/csg-aks-training/app/api

    $ cat Dockerfile

    $ docker build -t rating-api .
    ```

2. Validate image was created with `docker images`

### MongoDB Container

1. Create a MongoDB image with data files

    ```bash
    $ cd ~/repos/csg-aks-training/app/db

    $ docker build -t rating-db .
    ```

2. Validate image was created with `docker images`

## Run Containers

### Setup Docker Network

Create a docker bridge network to allow the containers to communicate internally.

```bash
$ docker network create --subnet=172.18.0.0/16 heroes-league
```

### MongoDB Container

1. Run mongo container

    ```bash
    $ docker run -d --name db --net heroes-league \
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
        --net heroes-league --ip 172.18.0.11 -p 3000:3000 rating-api
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
        --net heroes-league --ip 172.18.0.12 -p 8080:8080 rating-web
    ```

2. Validate by running `docker ps -a`

3. Test web app by navigating to [http://localhost:8080/](http://localhost:8080/) in your Web browser (assuming your *nix machine has a desktop environment). Otherwise, you can also test via curl:

    ```bash
    $ curl http://localhost:8080/

    <html>
    ...
    </html>
    ```

Now that the images have been built locally, let us now deploy them to an external registry, such as us Azure Container Registry.

[Continue to Next Section (Azure ACR)](02.01-deploy-docker-img-acr.md)
