# CSG Columbus AKS Traning

## Overview

This workshop will guide you through migrating an application from "on-premise" to containers running in Azure Kubernetes Service.

The labs are based upon a node.js application that allows for voting on the Justice League Superheroes (with more options coming soon). Data is stored in MongoDB.

> Note: These labs are designed to run on a Linux CentOS VM running in Azure (jumpbox) along with Azure Cloud Shell. They can potentially be run locally on a Mac or Windows, but the instructions are not written towards that experience. ie - "You're on your own."

> Note: Since we are working on a jumpbox, note that Copy and Paste are a bit different when working in the terminal. You can use Shift+Ctrl+C for Copy and Shift+Ctrl+V for Paste when working in the terminal. Outside of the terminal Copy and Paste behaves as expected using Ctrl+C and Ctrl+V.

## Course Outline

1. [Setup Lab environment](labs/day1-labs/00-lab-environment.md)
2. [Create an Azure Kubernetes Service (AKS) cluster](labs/day1-labs/03-create-aks-cluster.md)
3. [Create Docker images for apps](labs/day1-labs/02-dockerize-apps.md)
4. [Push Docker images to Azure Container Registry](labs/day1-labs/02.01-deploy-docker-img-acr.md)
5. [Deploy application to Azure Kubernetes Service](labs/day1-labs/04-deploy-app-aks.md)
6. [Kubernetes UI Overview](labs/day1-labs/05-kubernetes-ui.md)
7. [Operational Monitoring and Log Management](labs/day1-labs/06-monitoring-k8s.md)
8. [Update and Deploy New Version of Application](labs/day1-labs/09-update-application.md)
9. [Kubernetes Ingress Controllers](labs/day2-labs/ingress-controller.md)

## Demos

These labs, in addition to creating an AKS Cluster, have the potential to take a significant amount of time to complete. Therefore, these are intended to be presented as a demo as is or to be followed outside of a classroom setting.

1. [Application and Infrastructure Scaling](labs/day1-labs/07-cluster-scaling.md)
2. [Upgrade an Azure Kubernetes Service (AKS) cluster](labs/day1-labs/10-cluster-upgrading.md)

## Additional Configuration

* [Installing Nginx Ingress Using Helm](labs/day2-labs/helm-install-ingress.md)

## Credits and Source

This project is based on the [Black Belt AKS Hackfest Labs](https://github.com/Azure/blackbelt-aks-hackfest) and has been adapted for internal training use.

## License

This software is covered under the MIT license. You can read the license [here](LICENSE).

This software contains code from Heroku Buildpacks, which are also covered by the MIT license.

This software contains code from [Helm][], which is covered by the Apache v2.0 license.

You can read third-party software licenses [here][Third-Party Licenses].
