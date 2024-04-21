# kubernetes-learning
Repository used for learning about Kubernetes

## Lens IDE for Kubernetes
[Download and install Kubernetes Lens](https://k8slens.dev/)

## Minikube
- minikube start
- minikube status
- minikube stop

### Kubernetes Objects Template Structure

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



