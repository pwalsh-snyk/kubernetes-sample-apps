# Overview

Sample javascript application implementing the classic [2048 game](https://en.wikipedia.org/wiki/2048_(video_game)). Main project is based on the [game-2048 library](https://www.npmjs.com/package/game-2048) and [Webpack](https://webpack.js.org).

Main purpose is to better understand containerization and practice deploying resources to a kubernetes cluster.

## Requirements

To complete all steps and deploy the `2048-game` sample application, you will need:

1. A Kubernetes cluster configured and running.
2. Latest [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) version for Kubernetes interaction.
3. [Git](https://git-scm.com/downloads) client for interacting with the [kubernetes-sample-apps](https://github.com/digitalocean/kubernetes-sample-apps) repository.
4. [NodeJS](https://nodejs.org) and `npm` to build and test the 2048-game application code.
5. [Docker Desktop](https://www.docker.com/products/docker-desktop) to build and test the 2048-game application docker image locally.

## Building the 2048 Game Application

Main project is `javascript` based, hence you can build the application via `npm`:

```shell
npm install --include=dev

npm run build
```

You can test the application locally, by running below command:

```shell
npm start
```

Above command will start a web server in development mode, which you can access at [localhost:8080](http://localhost:8080). Please visit the main library [configuration](https://www.npmjs.com/package/game-2048#config) section from the public npm registry to see all available options.

## Building the Docker Image

A sample [Dockerfile](./Dockerfile) is provided in this repository as well, to help you get started with dockerizing the 2048 game app. Depending on the `NODE_ENV` environment variable, you can create and publish a `development` or `production` ready Docker image.

First, you need to clone this repository (if not already):

```shell
git clone https://github.com/pwalsh-snyk/kubernetes-sample-apps.git
```

Then, change directory to your local copy:

```shell
cd kubernetes-sample-apps/game-2048-example
```

Next, issue below command to build the docker image for the 2048 game app, Below examples assume you already have a [Docker Hub Repository](https://www.docker.com/products/docker-hub/) set up (make sure to replace the `<>` placeholders accordingly):

```shell
docker build -t <docker-username>/<docker-repo-name>:game-2048 .
```

**Note:**

The sample [Dockerfile](./Dockerfile) provided in this repository is using the [multistage build](https://docs.docker.com/develop/develop-images/multistage-build) feature. It means, the final image contains only the application assets (build process artifacts are automatically discarded).

Then, you can issue bellow command to launch the `2048-game` container (make sure to replace the `<>` placeholders accordingly):

```shell
docker run --rm -it -p 8080:8080 <docker-username>/<docker-repo-name>:game-2048
```

Now, visit [localhost:8080](http://localhost:8080) to check the 2048 game app in your web browser. Finally, you can push the image to your DigitalOcean docker registry (make sure to replace the `<>` placeholders accordingly):

```shell
docker push <docker-username>/<docker-repo-name>:game-2048
```

## Deploying to Kubernetes

The [kustomization manifest](kustomize/kustomization.yaml) provided in this repository will get you started with deploying the `2048-game` Kubernetes resources.

Next, edit the game-2048 [deployment manifest](kustomize/resources/deployment.yaml) using your favorite text editor (preferably with YAML lint support), and replace the `<>` placeholders with the docker image you just pushed to your Docker Hub repo. For example, you can use [VS Code](https://code.visualstudio.com/):

```shell
vim game-2048-example/kustomize/resources/deployment.yaml
```

Now, create Kubernetes resources using the kubectl kustomize option (`-k` flag):

```shell
kubectl apply -k game-2048-example/kustomize
```

The output looks similar to:

```text
namespace/game-2048 created
service/game-2048 created
deployment.apps/game-2048 created
```

If everything went well, you should have a new Kubernetes namespace created named `game-2048`. Inside the new namespace, you can inspect all resources created by the kustomization manifest from the sample apps repository (all game-2048 application pods should be up and running):

```shell
kubectl get all -n game-2048
```

The output looks similar to:

```text
NAME                            READY   STATUS    RESTARTS   AGE
pod/game-2048-f96755947-dgj7z   1/1     Running   0          5m19s

NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/game-2048   ClusterIP   10.245.120.202   <none>        8080/TCP    5m21s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/game-2048   1/1     1            1           5m22s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/game-2048-f96755947   1         1         1       5m22s
```

Finally, port-forward the `game-2048` service using `kubectl`:

```shell
kubectl port-forward service/game-2048 -n game-2048 8080:8080
```

Open a web browser and point to [localhost:8080](http://localhost:8080/). You should see the `game-2048` welcome page:

![2048 Game Welcome Page](assets/images/game-2048-welcome-page.png)

## Cleaning Up

To clean up all Kubernetes resources created by the 2048 game application, below command must be used:

```shell
kubectl delete ns game-2048
```

**Note:**

Kubectl kustomize subcommand has a delete option that can be used - `kubectl delete -k game-2048-example/kustomize`. But, it won't work well in this case because if the namespace is deleted first then the remaining operations will fail.
