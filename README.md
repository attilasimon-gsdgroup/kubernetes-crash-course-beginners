# Kubernetes for Absolute Beginners

> https://www.youtube.com/watch?v=s_o8dwzRlu4

## Kubernetes:  
→ Open source container **orchestration tool**  
→ Originally developed by Google  
→ Helps manage containerized apps in different deployment environments

Need for container orchestration tool:  
→ Trend from Monolith to Microservices  
→ Increased usage of containers  

Features:
- High Availability (no downtime)
- Scalability (high performance)
- Disaster Recovery (backup and restore)

## Architecture
- Cluster
  - Control-plane node (at least one)
    - Worker nodes (where the actual work is running)  
      Each has:   
      → Kubelet - primary "node agent"  
      → Multiple containers deployed

Virtual Network: is used to connect and isolate the control plane and worker nodes, plus any external clients and services, creating one unit that connects everything together

Control-plane node (legacy: Master node):  
Contains several Kubernetes processes that are absolutely necessary to run a Cluster
  - **API Server**
    - is the entrypoint to the cluster
    - UI, API and CLI clients will all talk to this API server
  - **Controller Manager**
    - keeps track of what's happening in the cluster
    - wheather something needs to be prepared
    - if a container has to be restarted
  - **Scheduler**
    - deciding where to put the pods: schedulig pods on different worker nodes, based on workload and available resources
    - the decision is based on the pod requirements, which includes information about containers and their resource needs
  - **etcd**
    - key-value storage
    - holds the current state of the k8s cluster
    - backup and restore can be done using the etcd snapshots

Worker nodes → higher workload (running containers) → more resouces 

Control-plane nodes → a handful of processes →  less resources → BUT more important than a worker nodes, because it runs the whole cluser → have at least 2x nodes and backups

## Main Kubernetes Components

```
[Pod]       [ConfigMap]
                          [StatefulSet]
[Service]   [Secret]
                          [DaemonSet]
[Ingress]   [Deployment]
```


```
          ┌────────────────────────┐[Ingress]      ┌──────────────────────┐
          │ ┌─────────┐┌──────┐    │    │          │                      │
          │ │ConfigMap││Secret│    │    │          │                      │
          │ ├─────────┴┴──────┴──┐ │    │          │ ┌──────────────────┐ │
          │ │  my-app            │─┼[Service]┬─────┼─┤  my-app          │ │
          │ │                pod │ │    │ [load b] │ │    pod (replica) │ │
          │ └────────────────────┘ │    │          │ └──────────────────┘ │
          │ ┌────────────────────┐ │    │          │ ┌──────────────────┐ │
          │ │  DB                │ │    │          │ │  DB              │ │
          │ │                pod │─┼[Service]┬─────┼─┤    pod (replica) │ │
          │ ├──────┬─────────────┘ │      [load b] │ └──────────────────┘ │
  [Cloud]─┼─┤Volume├─[drive]       │               │                      │
          │ └──────┘               │               │                      │
          │                 Node   │               │       Node           │
          └────────────────────────┘               └──────────────────────┘
```

### Node and Pod

**Node** (worker) = virtual of physical machine  

**Pod**:
  - Smallest unit in Kubernetes
  - Abstraction over a container
  - Usually: 1 app / Pod
  - Each Pod gets its own IP address
  - ! Pods are ephemeral (can die easily)

When a Pod dies, a new one is created in its place with a NEW IP address. Because of this IP address change the Service component is used.

### Service and Ingress

**Service**:
- Permanent IP address
- Lifecycle of Pod and Service are not connected
  - if the Pod dies, the Service and its IP address stays
- acts like a load balacer between replicas of Pods

**Service type**:  
 - External
   - ex: browser access
   - not very practical: http://124.89.101.2:8080
   - good only for testing
   - for HTTPS protocol and domains **Ingress** is used
 - Internal, ex: database

**Ingress**:
- request --> Ingress --fw--> Service

### ConfigMap & Secret

**ConfigMap**:
- External configuration of the application
- Ex: DB_URL = mongo-db-service
- connected to the Pod
- can be easily changed without affecting the application
- not for confidential data, use **Secret**

**Secret**:
- Used to store secret data (credentials, env vars, properties)
- base64 encoded format
- 3rd party encryption tools have to be used to encrypt it
- connected to Pod so that the pod can read data from it

### Volume

When a database pod dies and is restarted, the data is gone. To resolve this problem, Kubernetes Volumes are used

Volume:
- a way for containers in a pod to access and share data via the filesystem, allowing for data persistence beyond the lifecycle of individual containers
- attaches a physical storage to the Pod
  - local, on the same machine with the Pod
  - or remote, outside of cluster, like a cloud
- this way the DB pod can be restarted without data loss
- ! Kubernetes doesn't manage data persistance

### Deployment & StatefulSet

If the application Pod dies, there will be some downtime. Distributed systems resolve this problem. We replicate the Pod on another node. The replica will be connected to the same service, which will act as a load balancer. We don't create a second Pod, but instead we define a **blueprint** for Pods and specify how many replicas to run. This blueprint is called a "Deployment" in Kubernetes. 

**Deployment**:
- is an Abstraction of Pods
- this is created instead of creating Pods
- a blueprint for pods, where we:
  - set how many pod replicas to create
- used for stateless apps

That means, in practice we mostly work with **Deployments** and not with Pods. When one pod dies, the Service redirects the request to another replica.

We cannot replicate database pods using a Deployment, because database has state. All the clones of DB pods will need to access the same shared data storage, where we need a mechanism to manage pods reading and wrinting from and to that storage, to avoid data incosistencies. That mechanism is called "StatefulSet".

**StatefulSet (STS)**:
- used for stateful apps or databases
- takes care of replicating the pods scaling them up or down
- synchronizes the pods to avoid data incosistencies
- deploying StatefulSet is more difficult than Deployments
  - because of this, DB are often hosted outside of Kubernetes cluster 

## Kubernetes Configuration

All the configuration goes through a Control-plane node, with a process called "API Server". Kubernetes clients, UI, API, CLI, they all send their configuration requests to the API Server, the only entrypoint to the cluster. Configuration request have to be either in YAML or JSON format.

Each configuration file has 3 parts:

0. apiVersion and kind
1. Metadata
2. Specification
   - spedific to the kind of the configuration
3. Status
   - automatically generated and added by Kubernetes
   - Kubernetes always compares the "desired" state with the "actual" state
   - if "desired" != "actual", K8S will compare the "status" with the "specification and try to resolve the current differences (ex. number of replicas)
   - "etcd" holds the current status of any K8S component 

## minikube

[minikube](https://minikube.sigs.k8s.io/docs/) quickly sets up a local Kubernetes cluster, where the master processes and worker processes run on one single node.
It is mainly used to test out things locally without needing to have all those resources that a normal K8S setup would need.

There are multiple ways to run minikube, `docker` and `virtual machine` would be the preferred ones.

When running minikube using Docker, we'll have 2 layers of Docker:
- on our system: minikube runs as a Docker container 
- inside minikube: another Docker will run our app containers

[Install](https://minikube.sigs.k8s.io/docs/start/) and start:
```bash
minikube start --driver=docker
```
→ first run will take some time, to get all the images needed and then set up the local cluster

Check status: 
```bash
minikube status
```

## kubectl

Command-line tool for K8S cluster. The most powerful of the 3 clients for interacting with the cluster (UI, API, CLI).

It is part of K8S and can be used with **minikube** too.

After installing and starting minikube, to get a list of nodes to check their status:
```bash
kubectl get node
```
→ for now there will be only one control-plane node (master).

## Demo project

```
 ┌──────────────────────┐         ┌────────────────────────────────────┐
 │ Browser        ○ ○ ○ │         │                                    │
 ├──────────────────────┤         │ ┌──────────┐           ┌─────────┐ │
 │ http://node-ip:30100 │-→[Service]│ WebApp   │-→[Service]│ Mongo   │ │
 ├──────────────────────┤         │ │      pod │           │     pod │ │
 │                      │         │ └───┬────┬─┘           └───┬─────┘ │
 │                      │         │     ↑    ↑___[Secret]______↑       │
 └──────────────────────┘         │     |________[ConfigMap]           │
                                  └────────────────────────────────────┘
```
K8S docs: https://kubernetes.io/docs/home/  
Demo-app: https://hub.docker.com/r/nanajanashia/k8s-demo-app

### Configuration files

Basic configuration for any application

#### ConfigMap

`mongo-config.yaml` ([ConfigMap docs](https://kubernetes.io/docs/concepts/configuration/configmap/))
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongo-url: mongo-service
```

#### Secret

`mongo-secret.yaml` ([Secret docs](https://kubernetes.io/docs/concepts/configuration/secret/))
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-user: bW9uZ291c2Vy
  mongo-password: bW9uZ29wYXNzd29yZA==
```
→ the values are `base64` encoded:
- Linux/Mac Terminal: 
  ```shell
  echo -n mongouser | base64
  ```
- Windows PowerShell: 
  ```ps
  [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("mongopassword"))
  ```

#### Deployment & Service

`mongo.yaml` (docs: [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) & [Service](https://kubernetes.io/docs/concepts/services-networking/service/))

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:                         # "label" optional for Deployments
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:                  # matching all Pods by label
      app: mongo                  # key:value -> app:mongo
  template:                       # template: configuration for Pod
    metadata:                     # has its own "metadata" section
      labels:                     # "label" required for Pods
        app: mongo
    spec:                         # has its own "spec" section
      containers:                 # which image?
      - name: mongodb
        image: mongo:7.0
        ports:                    # which port? 
        - containerPort: 27017
---                               # separates "Deployment" from "Service"
apiVersion: v1
kind: Service
metadata:
  name: mongo-service             # same as defined in the ConfigMap
spec:
  selector:
    app: mongo                    # "label" of Pods belonging to Service 
  ports:
    - protocol: TCP
      port: 27017                 # Service port - different or same as target
      targetPort: 27017           # Pods port
```

→ get image name, tag and port from official docker hub page: [Mongo](https://hub.docker.com/_/mongo)  
→ replicas have a unique `name`, but the given `label` will be the same for all  
→ for databases `Deployment` is not good to make replicas, so set it to `1` for simplicity  
→ `Deployment` and `Service` are separated by `---` in the same file

`webapp.yaml` (data taken from [docker hub page](https://hub.docker.com/r/nanajanashia/k8s-demo-app))

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nanajanashia/k8s-demo-app:v1.0
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

### Pass Secret data to Mongo Deployment

Take variable names from [official docker hub page](https://hub.docker.com/_/mongo) and [define environment variables for container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) in the Deployment config file, using the values from the Secret file:

```yaml
...
    spec:                         
      containers:                 
      - name: mongodb
        image: mongo:7.0
        ports:                    
        - containerPort: 27017
        env:                      # environment variables for container
        - name: MONGO_INITDB_ROOT_USERNAME # name taken from Mongo cont. docs
          valueFrom: 
            secretKeyRef:         # value taken set as a reference to Secret
              name: mongo-secret  # Secret name
              key: mongo-user     # key taken from Secret's data section
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongo-secret
              key: mongo-password
```
### Pass Config Data to WebApp Deployment

The custom webapp container is configured to expect specific environment variable names used to connect to the database, username, password and url.

Add these as environment variables

`webapp.yaml`
```yaml
...
    spec:
      containers:
      - name: webapp
        image: nanajanashia/k8s-demo-app:v1.0
        ports:
        - containerPort: 3000
        env:
        - name: USER_NAME               # defined in the app
          valueFrom:
            secretKeyRef:               # reference from Secret
              name: mongo-secret
              key: mongo-user 
        - name: USER_PWD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password 
        - name: DB_URL
          valueFrom:
            configMapKeyRef:            # reference from ConfigMap
              name: mongo-config
              key: mongo-url
```

### Configure external Service

Making the app accessible from the browser.

`webapp.yaml`
```yaml
...
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort             # external Service type, default: ClusterIP
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30100        # default: 30000-32767
```

### Deploy all resources in Minikube cluster

No components yet:
```shell
kubectl get pod
```
→ "No resources found in default namespace."

We need to create the external configurations first, because mongo and webapp are referencing those configs

Create mongo-config and mongo-secret first:
```shell
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
```

Next, create the database, because the web application depends on it:
```shell
kubectl apply -f mongo.yaml
```

Finally, deploy the web app:
```shell
kubectl apply -f webapp.yaml
```

### Interacting with K8s cluster

Get all components created:
```shell
kubectl get all
```

Get ConfigMap and Secrets:
```shell
kubectl get configmap
kubectl get secret
```

Get list of individual components, like pods:
```shell
kubectl get pod
```

Get `kubectl` help:
```shell
kubectl --help        # general help
kubectl get --help    # help on specific command
```

Get more info on a component, for example a service or a pod:
```shell
kubectl describe service webapp-service
kubectl describe pod webapp-deployment
```

Get logs for a pod:
```shell
kubectl logs webapp-deployment-5766fd95c7-59vzr # use -f at the end to stream logs
```
### Access WebApp in Browser

We need the ip and port of the external service for webapp.  

Get port from the service list:
```shell
kubectl get service
```

NodePort Service is accessible on each Worker Node's IP address

Get IP address from the single node we have (minikube):
```shell
minikube ip
kubectl get node
kubectl get node -o wide
```

Then access the app in the browser with `http://[minikube IP]:[webapp-service port]`, for example:  http://192.168.49.2:30100 

### Timeout issue on Windows

On Windows (Docker Desktop driver), the minikube IP (e.g., 192.168.49.2) is **internal to Docker** and unreachable from host machine/browser.  
Direct access like http://192.168.49.2:30100 times out — normal behavior, not a bug.

**Solutions (pick one):**
1. **Use minikube command** :  
   ```
   minikube service webapp-service
   ```
   - Opens browser to http://127.0.0.1:[random-port] via tunnel
   - check terminal for details
   - more info: [Accessing minikube apps](https://minikube.sigs.k8s.io/docs/handbook/accessing/)

2. **Local forwarding** (using host ports):  
   ```
   kubectl port-forward svc/webapp-service 3000:3000
   ```
   - open http://localhost:3000 in browser
   - port can be any host port, like 8080, or even 30100