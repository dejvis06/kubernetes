# kubernetes-learning
Repository used for learning about Kubernetes

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

## POD's

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
### Explanation of the Top-Level `spec` in ReplicaSet

1. **replicas**: Specifies the desired number of pod instances that should be running at all times

2. **selector**: Used to identify which pods fall under the management of this ReplicaSet. The ReplicaSet manages all pods that match the labels specified in the selector.

   - **matchLabels**: This map of `{key: value}` pairs identifies pods. A pod must have all the specified labels (with matching values) to be managed by this ReplicaSet. In the example, the label is `app: example-pod`.

3. **template**: Defines the template for creating new pods under this ReplicaSet. It includes:
   - **metadata**: Contains metadata for the pods to be created, mainly labels which should match the selector in the ReplicaSet spec.
   - **spec**: Outlines the specifications of the pods, including containers, volumes, and other settings.

<span style="font-size: larger;">Create and run: <code>kubectl apply -f replicaset-example.yml</code></span>
