# Kubernetes Handbook for NodeJS

This is a k8s "handbook" for local development with kind clusters based on a  Node app by [Learnk8s](https://learnk8s.io/developing-and-packaging-nodejs-docker).

> Table of Contents
- [Runbooks for containerization and orchestration](#Runbooks)
- [Kubernetes Resources](#Kubernetes-Resources)
- [Controllers](#Controllers)
- [Operators](#Operators)
- [Custom Resource Definitions](#Custom-Resource-Definitions)
- [Draining Nodes](#Draining-Nodes)
- [Logs](#Logs)
- [fluentd](#fluentd)
- [Kubernetes CLI](#Kubernetes-CLI)
## Dependencies (brew)
- Kubernetes
- NodeJS
- Mongo (note to install via `brew tap mongodb/brew`, then `brew install mongodb-community`, then `brew services start mongodb-community`).
- Docker Desktop for Mac 
- [kind](https://kind.sigs.k8s.io/) to run local Kubernetes cluster

> Nice to have dependencies (brew)
- kubectx
- kubens

## Runbooks

### Dockerization and manually running containers

> A little "Runbook" for local development using the `kind` tooling for k8s

> Build to containerize, tag and push image to Docker hub container registry
-  `docker build -t knote .`
- `docker tag knote <username>/knote.js:1.0.0`
- `docker push <username>/knote.js:1.0.0`
- Create Docker Network to connect Node app and MongoDB
- `docker network create knote-mongodb`

> Pull and run Docker's default Mongo image on your local Docker network
- `docker run --name=mongo --rm --network=knote-mongodb mongo`
- MongoDB is now running.

> Pull and run your Docker image for NodeJS app to run on your Docker network
- `docker pull <username>/knote.js:0.0.1`

> Start NodeJS container. Use port forwarding to map ports in container to local
- `docker run --name=knote --rm --network=knote-mongodb -p 3000:3000 -e MONGO_URL=mongodb://mongo:27017/dev <username>/knote.js:0.0.1`

> Make sure you shutdown your containers with `docker stop` to cleanup
- To differentiate between using Docker versus using Kubernetes for orchestration,
consider shutting down these containers you've been running.

### Container orchestration with Kubernetes

> A little "Runbook" for local development using the `kind` tooling for k8s
- The Docker tutorial above uses Docker Networks to run and "orchestrate" containers.
- Shut those containers down and let's instead use Kubernetes to declaratively
run, stop and start those containers.

> Create Kind cluster and namespace
- `kind create cluster --name awesomeness` // You just created a kind cluster `awesomeness` w `kind` prefix
- To view cluster `kubectl cluster-info`
- use `kubectx` to view cluster(s) and `kubens` for namespaces

> Define NodeJS Kubernetes Deployments and Services Resources
- Define [Services and Deployments for Node app](./kube/knote.yaml)
- View `specs` with `kubectl explain deployment`, compare `specs` with state of resources (e.g. `get`)
- Use `labels` to define which Services relate to which Deployments
- Use a ClusterIP (default) Service `type`, as local kind clusters do not support `LoadBalancer` like cloud providers do

> Define PersistentVolumeClaim, Service and Deployment for MongoDB Resources
- Define [Volume Resources for MongoDB](./kube/mongo.yaml)
- Mount Volume at `/data/db`, where Mongo default should be
- This is a persistent storage volume with a different lifecycle than Mongo container
- _hostname_ is `mongo` because the _name of the service_ is `mongo`.

> Service Discovery 
- labels
- internal DNS of each cluster / pod

> Deploying the containers to the cluster
- `kubectl apply -f ./kube` will _declaratively apply_ the resources to the cluster selected in the current context
- `kubectl create` imperatively creates each resource and complains if the resources already exist
- Then watch: `kubectl get pods --watch`

> Port forward NodeJS Kubernetes pod port 3000 to port 3000 of your local machine
- `kubectl port-forward <PODNAME> 3000:3000`

> cURL or view in browser at port 3000 of local machine 


## Kubernetes Resources

[k8s resources API](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/)

### Resource Categories

- **Workload Resources:** run and manage cluster
- **Storage and Config Resources:** Inject initial data into apps. PERSIST external data (outside of container).
- **Discovery and Load Balancing Resources:** Expose workloads into external services
- **Cluster Resources:** Define cluster config ... typically used by cluster operators!
- **Metadata Resources:** configure other resources


### Kind: A kind of REST resource.

### Workload Resources

> Containers: Created by Controllers, through Pods.

> Pods: Run containers. Smallest unit of deployment.

> Controllers: Deployment, Job, StatefulSet, more ...
  - `Deployment`: Stateless apps w persistence (ex: HTTP server).
  - `Job`: Run-to-completion applications (batch jobs)
  - `StatefulSet`: Stateful, persistent apps (ex: databases).

> Defining a deployment
  - A Deployment defines how to run an app in the cluster, but it doesn't make it available to other apps.

> Labels: match pods to deployments
  - `labels` to define pods that wrap containers, `matchLabels` to match _those_ labels to a deployment.

> Defining a Service
  - To expose your app, you need a Service. A Service will expose a Pod by forwarding requests to that pod.
  - A Service also guarantees availability because it only routes traffic to pods with containers that are ready! It also reassigns the IP when new pod comes up!

> Contextual Fields (e.g. `selector`)
  - When in a `Deployment definition`, `selector` selects what Service(s) to deploy.
  - When in a `Service definition`, `selector` selects what pods to expose.

## Controllers

## Operators

## Custom Resource Definitions

## Draining Nodes

## Logs

## fluentd

## Kubernetes CLI

