# CSG Columbus AKS Traning

## Overview

This workshop will guide you through migrating an application from "on-premise" to containers running in Azure Kubernetes Service.

The labs are based upon a node.js application that allows for voting on the Justice League Superheroes (with more options coming soon). Data is stored in MongoDB.

The labs assume you have a basic understanding of *nix environments, such as Linux and MacOS, as the labs utilize heavy use of the command line (terminal).

Certain labs, such as launching the Kubernetes UI or running Docker containers locally, require use of a Desktop environment. If you have a Ubuntu VM running in Azure, you can [configure Remote Desktop for it and install a desktop environment](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/use-remote-desktop).

> Note: These labs are designed to run on a Linux Ubuntu VM running in Azure or locally. They can potentially be run locally on a Mac without any issues or on Windows with some caveats. In fact, the labs were tested and run using a MacOS X system. If you intend to follow these instructions on a Windows environment, although we strongly discourage you from doing so, "you're on your own."

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
