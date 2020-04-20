# NodeJS Example App Continuous Deployment

This repository has been created with the mission of giving a general Tekton objects and pipelines implementation overview. It is important to bear in mind that an objective of introduction in Tekton components is pursued. Please, review [Tekton basics](../README.md) if you have not took at look at it so thus far.

As an example, the following laboratory implements a continuous deployment workflow in an Openshift Application based on Tekton technology.

## NodeJS App Example Deployment

Firstly, an application running is required in order implement its continuous deployment. A part of the application, it will be required some OpenShift resources in order to integrate all Tekton Objects.

The following list includes a set of steps to set up a sample NodeJS application. Please, follow this procedure for taking the first steps :

-   Create Openshift projects

```
$ oc login -u user -p password https://console-openshift-console.apps.example.com
$ oc new-project nodejs-app-example
$ oc new-project nodejs-app-example-cicd
```

-   Download NodeJS App Example Repository

```
$ git clone https://github.com/acidonper/nodejs-app-example.git
```

-   Deploy _NodeJS App Example_ in Openshift (Please, visit [Deploy NodeJS App Example in Openshift ](https://github.com/acidonper/nodejs-app-example/tree/master/.openshift) for more information about this App deployment in Openshift)

```
$ cd nodejs-app-example/.openshift
$ sh openshift-nodejs-app-example.sh nodejs-app-example
```

### Test Application

Once the procedure has been executed and _NodeJS App Example_ objects have been created in Openshift, it would be a good idea to perform the following steps in order to test the new deployed App's status:

-   Check the App's Pod is running

```
$ oc get pods -n nodejs-app-example
NAME                          READY   STATUS      RESTARTS   AGE
nodejs-app-example-1-92jnr    1/1     Running     0          28m
...
```

-   Check the App's ImageStream has been built

```
$ oc get is -n nodejs-app-example
NAME                 IMAGE REPOSITORY                                                                                TAGS     UPDATED
nodejs-app-example   default-route-openshift-image-registry.apps-crc.testing/nodejs-app-example/nodejs-app-example   latest   26 minutes ago
```

-   Check the App's route

```
$ oc get route -n nodejs-app-example
$ curl <route>
```

## Continuous Deployment procedure implementation (Tekton)

From a general point of view, in a complete CD Tekton automated procedure will be necessary to have involved the following objects:

-   A service account to generate Tekton objects automatically (\*Created by default -> "pipeline")
-   A service account responsible for start new app builds and deployments
-   A set of task to implement application test and perform app builds/deployements
-   A pipeline to group and organize tasks
-   A Trigger Template which links pipelines to the dynamic resources (git repository based on a specific commit/push)
-   A Trigger Binding which captures events parameters (git commit ID)
-   An Event listener to capture commits/push events from the repository and trigger Tekton automatized procedure

In this laboratory, an Openshift template has been designed in order to create the previous described objects in an easy way automatically. In any case, it is a good idea to review this template carefully to identify these objects, test API versions, discover relations between them, etc.

Once the Openshift template has been reviewed, it is required to perform the following procedure:

```
$ cd nodejs-cd-example
$ oc process -f nodejs-app-tekton-cicd.yaml -p APP_NAME=nodejs-app-example APP_NAME_BC=nodejs-app-example APP_NAME_DC=nodejs-app-example APP_NAMESPACE=nodejs-app-example| oc apply -f -
```

### Check Tekton Objects

Template application would have added some new tekton objects in `<app>-cicd` project. In order to be able to check these new objects, it is required to follow the procedure included in this section:

-   Check new Tekton objects

```
$ oc project nodejs-app-example-cicd

$ tkn tasks ls
NAME                                             AGE
task-nodejs-app-example-new-commit-deploy-app    12 minutes ago
task-nodejs-app-example-new-commit-start-build   12 minutes ago
task-nodejs-app-example-new-commit-test          12 minutes ago

$ tkn pipeline ls
NAME                                                     AGE              LAST RUN   STARTED   DURATION   STATUS
pipeline-nodejs-app-example-new-commit-test-and-deploy   13 minutes ago   ---        ---       ---        ---

$ tkn triggertemplate ls
NAME                                                            AGE
triggertemplate-nodejs-app-example-new-commit-test-and-deploy   13 minutes ago

$ tkn triggerbinding ls
NAME                                                           AGE
triggerbinding-nodejs-app-example-new-commit-test-and-deploy   13 minutes ago

$ tkn eventlistener ls
NAME                                                          AGE
eventlistener-nodejs-app-example-new-commit-test-and-deploy   7 minutes ago

```

-   Check EventListener's Pod is running and get its route

```
$ oc get pods -n nodejs-app-example-cicd
NAME                                                              READY   STATUS    RESTARTS   AGE
el-eventlistener-nodejs-app-example-new-commit-test-and-de55zr7   1/1     Running   0          4m22s

$ oc get route -n nodejs-app-example-cicd
NAME                               HOST/PORT                                                                   PATH   SERVICES                                                         PORT            TERMINATION     WILDCARD
el-nodejs-app-example-nctad        el-nodejs-app-example-nctad-nodejs-app-example-cicd.apps-crc.testing               el-eventlistener-nodejs-app-example-new-commit-test-and-deploy   http-listener   edge/Redirect   None

```

## Push a new commit

Once all above steps have been performed, it is time to trigger a new push event in order to test the EvenListener operation. It is important to remember that EventListener triggers a new pipeline execution in reaction to events.

If OpenShift cluster is accessible from Internet, it is possible to generate a push/commit events through git command and test the results. In order to add a new repository webhook, please visit the following link [Creating Webhooks](https://developer.github.com/webhooks/creating/).

### Emulate a new commit

In this case, a new push/commit event is emulated using a curl tool in order to avoid Internet access to our cluster. Please, replace required fields and execute the following command:

```
$ curl -X POST \
-H "Content-Type: application/json" \
-d '{"head_commit":{"id":"<COMMIT_ID>"},"repository":{"name":"<GITHUB_REPO_NAME>","url":"<GITHUB_REPO_URL>"}}' https://<EL_ROUTE>
```

Specific example:

```
$ curl -X POST -k \
-H "Content-Type: application/json" \
-d '{"head_commit":{"id":"edba9c2d6ee51350d0b51c950d12282eecfcdf4c"},"repository":{"name":"nodejs-app-example","url":"https://github.com/acidonper/nodejs-app-example.git"}}' https://el-nodejs-app-example-nctad-nodejs-app-example-cicd.apps-crc.testing

{"eventListener":"eventlistener-nodejs-app-example-new-commit-test-and-deploy","namespace":"nodejs-app-example-cicd","eventID":"tz5bl"}
```

### Result

As a result, a new build and deployment instances have been triggered by Tekton. In order to review these events state, it is required to follow next steps:

-   Check EventListener's Pod logs

```
$ oc get pods -n nodejs-app-example-cicd
NAME                                                              READY   STATUS    RESTARTS   AGE
el-eventlistener-nodejs-app-example-new-commit-test-and-dekcrxs   1/1     Running   0          14m

$ oc logs el-eventlistener-nodejs-app-example-new-commit-test-and-dekcrxs
...
{"level":"info","logger":"eventlistener","caller":"sink/sink.go:147","msg":"params: %+v[{git-repo-url {string https://github.com/acidonper/nodejs-app-example.git []}} {git-repo-name {string nodejs-app-example []}} {git-revision {string edba9c2d6ee51350d0b51c950d12282eecfcdf4c []}}]","knative.dev/controller":"eventlistener","/triggers-eventid":"rtxpv","/trigger":"nodejs-github-trigger"}
{"level":"info","logger":"eventlistener","caller":"resources/create.go:91","msg":"Generating resource: kind: &APIResource{Name:pipelineresources,Namespaced:true,Kind:PipelineResource,Verbs:[delete deletecollection get list patch create update watch],ShortNames:[],SingularName:pipelineresource,Categories:[tekton tekton-pipelines],Group:tekton.dev,Version:v1alpha1,}, name: nodejs-app-example-git-repo-hwqlb","knative.dev/controller":"eventlistener"}
{"level":"info","logger":"eventlistener","caller":"resources/create.go:99","msg":"For event ID \"rtxpv\" creating resource tekton.dev/v1alpha1, Resource=pipelineresources","knative.dev/controller":"eventlistener"}
{"level":"info","logger":"eventlistener","caller":"resources/create.go:91","msg":"Generating resource: kind: &APIResource{Name:pipelineruns,Namespaced:true,Kind:PipelineRun,Verbs:[delete deletecollection get list patch create update watch],ShortNames:[pr prs],SingularName:pipelinerun,Categories:[tekton tekton-pipelines],Group:tekton.dev,Version:v1alpha1,}, name: build-deploy-nodejs-app-example-hwqlb","knative.dev/controller":"eventlistener"}
{"level":"info","logger":"eventlistener","caller":"resources/create.go:99","msg":"For event ID \"rtxpv\" creating resource tekton.dev/v1alpha1, Resource=pipelineruns","knative.dev/controller":"eventlistener"}

```

-   Check PipelineRun status

```
$ tkn pipelinerun ls
NAME                                    STARTED          DURATION   STATUS
build-deploy-nodejs-app-example-hwqlb   14 minutes ago   1 minute   Succeeded
```

-   Review nodejs-app-example application

```
$ oc get pods -n nodejs-app-example
NAME                          READY   STATUS      RESTARTS   AGE
nodejs-app-example-1-build    0/1     Completed   0          89m
nodejs-app-example-1-deploy   0/1     Completed   0          91m
nodejs-app-example-2-7dlm9    1/1     Running     0          12m
nodejs-app-example-2-build    0/1     Completed   0          13m
nodejs-app-example-2-deploy   0/1     Completed   0          12m

$ oc get is -n nodejs-app-example
NAME                 IMAGE REPOSITORY                                                                                TAGS     UPDATED
nodejs-app-example   default-route-openshift-image-registry.apps-crc.testing/nodejs-app-example/nodejs-app-example   latest   13 minutes ago

```

## Nice to have

This repository implements a basic Continuous Deployment workflow in Tekton. The following list talks about features which will be nice to have in production workflows:

-   EventListener security through secret and interceptors (Please visit [interceptor](https://github.com/tektoncd/triggers/blob/master/docs/eventlisteners.md#interceptors) for more information)
-   Multi-environment implementation (Dev, Pre, Pro...)
-   Continuous Integration workflows
-   Initial Deployments workflows
-   Pull request approval workflows
-   Code quality metrics

## License

BSD

## Author Information

Asier Cidon
