# Tekton - OpenShift Pipelines

Tekton is a powerful and flexible open-source framework for creating CI/CD systems, allowing developers to build, test, and deploy across cloud providers and on-premise systems (Kubernetes, serverless, VMs, etc) by abstracting away the underlying details.

OpenShift Pipelines is a cloud-native, continuous integration and delivery (CI/CD) solution for building pipelines using Tekton. The following list includes some features which are key in this new CI/CD pipelines generation:

-   Standard CI/CD pipeline definition based on Tekton
-   Build images with Kubernetes tools such as S2I, Buildah, Buildpacks, Kaniko, etc
-   Deploy applications to multiple platforms such as Kubernetes, serverless and VMs
-   Easy to extend and integrate with existing tools
-   Scale pipelines on-demand
-   Portable across any Kubernetes platform
-   Designed for microservices and decentralized teams
-   Integrated with the OpenShift Developer Console

This repository tries to give a brief summary of the most basic and important concepts in Tekton. Some examples of the most important objects have been included in order to be able to test and start working with this solution.

On the other hand, this repository implements a simple example about how to create and run a NodeJS pipeline for managing the continuous deployment of this containerized app on OpenShift handling an emulated GitHub webhook request to kickoff the process automatically.

## First Steps

Firstly, a set of elements will be required in order to be able to implement the example or create any of the example objects included in this repository. The following list gives a short resume about these elements:

-   An OpenShift 4 Cluster installed is required. Please, visit [Try Openshift](https://www.openshift.com/try) for more information about set up an OCP cluster. On the other hand, keep in mind that it is possible to use CRC (_Red Hat CodeReady Containers_).
-   OpenShift Pipelines (Tekton) installed by [Openshift Pipelines Operator](https://www.openshift.com/learn/topics/pipelines)
-   `oc` and `tkn` clients installed
-   A NodeJS App GitHub repository (In this example, it will be used [NodeJS App Example Repository](https://github.com/acidonper/nodejs-app-example))

## Tekton Objects

Tekton defines a number of Kubernetes custom resources as building blocks in order to standardize pipeline concepts and provide a terminology that is consistent across CI/CD solutions. The custom resources needed to define a complete workflow are listed below:

-   **PipelineResource** -> inputs (e.g. git repository) and outputs (e.g. image registry) which should be used during a Pipeline execution
-   **Task** -> a reusable, loosely coupled number of steps that perform a specific task (e.g. building a container image, perform a code test, etc)
-   **Pipeline** -> a set of Tasks which should be performed using some PipelineSources
-   **PipelineRun** -> a pipeline execution definition which is triggered manually or by an automated process. Basically, it is responsible for linking resources to pipelines.
-   **TriggerTemplate** -> a set of PipelineResources, created dynamically based on an event, and PipelineRuns. Typically, an event is a webhook event (commit, pull_request, etc).
-   **TriggerBinding** -> a map which enables users to capture fields from an event and store them as parameters (e.g. Commit ID in a webhook event when a repository commit/push event is performed)
-   **EventListener** -> a HTTP service which listens for events

The following graph describes how these Tekton objects interact with each other:

![OpenShift_Tekton_Graph](./docs/openshift_tekton_diagram.png)

### ServiceAccounts

A part of main Tekton objects, a group of service accounts are required in order to perform several actions in Kubernetes (Openshift). The following list include some roles or service accounts which will be integrated or used during pipelines or workflows execution and have to be into account:

-   Tekton Objects Manager -> This service account creates all tekton related objects in Openshift (A service account named _pipeline_ is created by Openshift Pipelines Operator automatically)
-   Git Repository Manager -> This service account is required to interact with a Git Repository (Not required in public repositories)
-   Docker Repository Manager -> This service account is required to interact with a Container Images Repository wherever necessary
-   App Objects Manager -> This service account manages Apps objects in Openshift and it will be required as a part of the application deployment

Please, visit [Tekton Authentication Objects Management](https://github.com/tektoncd/pipeline/blob/master/docs/auth.md) for more information about external components authentication in Tekton.

## NodeJs application CD example

This section tries to give an example about how to implement continuous deployment in a microservice based application in Openshift. This application is based on Javascript language and it is deployed using NodeJS Red Hat official container image in order to build de application.

Please, visit [link](./nodejs-cd-example/README.md) in order to get the instructions of the example implementation.

## Other Examples

Finally, it is possible to find some Tekton Objects definitions in [example folder](./examples).

## License

BSD

## Author Information

Asier Cidon
