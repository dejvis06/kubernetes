# Introduction
Repository starts with kubernetes objects and it also contains a lightweight microservices architecture (Voting App).

## Contents
- [Lens IDE for Kubernetes](#lens-ide-for-kubernetes)
- [Minikube](#minikube)
- [Kubernetes Objects Template Structure](#kubernetes-objects-template-structure)
- [Pods](#pods)
- [ReplicaSet](#replicaset)
- [Deployments](#deployments)
- [Auto Scaling in Kubernetes](#auto-scaling-in-kubernetes)
- [RollingUpdate](#rollingupdate)
- [Services](#services)
  - [NodePort](#nodeport)
  - [ClusterIP](#clusterip)
  - [LoadBalancer](#loadbalancer)
- [Kubernetes Pod Scheduling: Anti-Affinity](#kubernetes-pod-scheduling-anti-affinity)
- [Voting App](#voting-app)
  - [Architecture](#architecture)
  - [Run](#run)
  - [Delete](#delete)


## Lens IDE for Kubernetes
[Download and install Kubernetes Lens](https://k8slens.dev/)

## Minikube
- minikube start
- minikube status
- minikube stop

### Kubernetes Objects Template Structure

**Install VS Code plugin: _YAML (YAML Language Support by Red Hat, with built-in Kubernetes syntax support
)_**

Check the server version of kubernetes running: **_kubectl version_** 

The server version must correspond to the _apiVersion_ in the yaml file:

```yaml
apiVersion:
kind: 
metadata:
spec:
    containers:
  - name: 
    image:
```
_apiVersion_ can either be _v1_ or _apps/v1_ depending on the kubernetes object type.

## Pod's

Can have 1 or more containers, configuration example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx:latest
```

<span style="font-size: larger;">Running pod: <em>kubectl apply -f pod-example.yml</em></span>


## ReplicaSet

Can be used for replicating pods, for cases such as  multiple consumers for parallel processing in a kafka scenario.


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-pod
  template:
    metadata:
      labels:
        app: example-pod
    spec:
      containers:
      - name: example-container
        image: nginx:latest
```
### Explanation of the top-level `spec` in ReplicaSet

1. **replicas**: Specifies the desired number of pod instances that should be running at all times

2. **selector**: Used to identify which pods fall under the management of this ReplicaSet. The ReplicaSet manages all pods that match the labels specified in the selector.

   - **matchLabels**: This map of `{key: value}` pairs identifies pods. A pod must have all the specified labels (with matching values) to be managed by this ReplicaSet. In the example, the label is `app: example-pod`.

3. **template**: Defines the template for creating new pods under this ReplicaSet. It includes:
   - **metadata**: Contains metadata for the pods to be created, mainly labels which should match the selector in the ReplicaSet spec.
   - **spec**: Outlines the specifications of the pods, including containers, volumes, and other settings.

<span style="font-size: larger;">Create and run: <code>kubectl apply -f replicaset-example.yml</code></span>

## Deployment's
Same structure as ReplicaSet but with a different kind:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-pod
  template:
    metadata:
      labels:
        app: example-pod
    spec:
      containers:
      - name: example-container
        image: nginx:latest
```
<span style="font-size: larger;">Create and run: <code>kubectl create -f deployment-example.yml</code></span>

This deployment comes with a static replicaset configuration. 

### Auto Scaling in Kubernetes:

Under spec/container configure the following properties:

```yaml
resources:
  requests:
    cpu: 100m  # 100 millicores (mCPU) request
  limits:
    cpu: 200m  # 200 millicores (mCPU) limit
```

Create a HorizontalPodAutoscaler:
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50  # target CPU utilization percentage
```

Explanation:

The HorizontalPodAutoscaler (HPA) targets a CPU usage -> 50% of 100m (the value in the deployment under resources.requests.cpu)
- If the actual CPU usage per Pod exceeds 50 millicores, the HPA interprets this as the Pod being under high load and scales up.  
- If the CPU usage is below 50 millicores, the HPA might scale down (reduce the number of Pods).

### RollingUpdate

Used for handling updates or newer versions of containers:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

1. **Start Update**: When an update to the Deployment is initiated (for example, changing the Docker image version), Kubernetes starts by creating an additional new Pod (because of `maxSurge: 1`) if there is sufficient capacity.
2. **Remove Old Pod**: Once the new Pod is up and running (ready to serve traffic), Kubernetes then terminates one of the old Pods (maintaining the limit set by `maxUnavailable`).
3. **Progressively Update**: This process repeats for each Pod in the Deployment until all Pods are updated to the new version.

## Service

### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-nodeport-service
spec:
  type: NodePort
  selector:
    app: example-pod
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```
- **port**: The port on which the service is exposed internally within the cluster. 
- **targetPort**: The port on the pod to which the service routes the traffic. This is where the application inside the pod is listening.
- **nodePort**: The port on each node's external IP at which the service can be accessed from outside the cluster. Traffic sent to this port is forwarded to the port of the service, which then routes to targetPort on the pods.

<code>kubectl create -f service-nodeport.yml</code></span>

### ClusterIP

A **ClusterIP** service is used for internal cluster communication. More suitable for management (for example load balancing, directing the request to another pod in case of failure) in dynamic environments (pod replication, failure or rollbacks), serving as a single point of entry with pre configured dns name.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: example-pod
```
- **port**: 
  - This is the port on which the service is exposed within the cluster.

- **targetPort**: 
  -  This is where the application inside the pod is actually listening.

These two properties ensure that traffic arriving at the service's `port` is correctly directed to the `targetPort` on the pods that match the service's selector. 

### LoadBalancer

Acts like a front service between the client and the cluster instances for balancing the load:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-loadbalancer
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      targetPort: 8080
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 8443
      nodePort: 30443
    - name: metrics
      port: 9100
      targetPort: 9100
      nodePort: 30100
  selector:
    app: myapp
```

## Kubernetes Pod Scheduling: Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3  # Ensure you have at least as many nodes as replicas
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - my-app
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: my-container
        image: nginx
```

The `affinity` field in the PodSpec allows you to set rules that affect how pods are scheduled in relation to other pods. These rules can either attract pods to each other (affinity) or repel them (anti-affinity).

### `podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution.labelSelector.matchExpressions`

This specifies rules that tell the Kubernetes scheduler to avoid placing the pod on a node where certain conditions are true regarding other pods on that node. It aims to place the pod such that it is not co-located with certain other pods, based on their labels.

- **Key**: For example, `"app"` refers to the label key based on which pods will be identified.
- **Operator**: In this example, `In`, which means the rule applies if the label's value for the key matches any in the specified list.
- **Values**: Here, it matches pods that have a label `app=my-app`.

### `topologyKey`

The scheduler ensures that pods with the same identifying labels (based on your anti-affinity rules) are not placed on nodes (kubernetes.io/hostname) that have the same hostname values.

### Voting App

#### Architecture
![Architecture Diagram](https://raw.githubusercontent.com/dockersamples/example-voting-app/main/architecture.excalidraw.png "Architecture Diagram of Voting App")

- A front-end web app in Python which lets you vote between two options
- A Redis which collects new votes
- A .NET worker which consumes votes and stores them in a Postgres database backed by a Docker volume
- A Postgres database backed by a Docker volume
- A Node.js web app which shows the results of the voting in real time


#### Run
```bash
kubectl create -f voting-app/
```

#### Delete
```bash
kubectl delete -f voting-app/
```
