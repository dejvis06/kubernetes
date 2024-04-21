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

### POD's

Configuration example:

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

Running pod: _kubectl apply -f pod-example.yml_



