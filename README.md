# Kubernetes Demo Project

This project contains Kubernetes Demo Project, commands and notes

## Deployment hierarchy
Deployment manages a replicates
Replicaset manages a Pod
Pod manages a Container

Deployment -> ReplicaSet -> Pod -> Container
Everything below Deployment is handles by Kubernetes


## Installation on Mac

```sh
brew update
brew install hyperkit
brew install minikube
```

## Minikube Commands
```sh
minikube start —vm-driver=hyperkit
minikube stop
minikube delete
```

## Kubectl Commands
```sh
kubectl get nodes
kubectl get services
kubectl get all
```

### CRUD Operations (Create, Read, Update, Delete)
#### Create
```sh
kubectl create deployment nginx-depl --image=nginx
kubectl apply -f <config file>.yaml
```

#### Read
```sh
kubectl get deployment
kubectl get replicaset
kubectl get pod
kubectl get nodes
kubectl get services
```

#### Update
```sh
kubectl edit deployment nginx-depl
kubectl apply -f <config file>.yaml
```

#### Delete
```sh
kubectl delete deployment nginx-depl
kubectl delete -f <config file>.yam
```

### Debugging
```sh
kubectl logs <pod name>
kubectl describe pod <pod name>
kubectl exec -it <pod name> -- bin/bash
```

### Other Commands
* **Get IP** - kubectl get pods <pod name> -o wide
* **Get YAML or Latest status**: kubectl get deployment <deployment name> -o yaml
* **Convert string to Base64** - echo -n '<string to encode>' | base64

### Access service from Minikube
```
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                ClusterIP      10.96.0.1       <none>        443/TCP          335d
mongodb-express-service   LoadBalancer   10.101.217.13   <pending>     8081:30000/TCP   5s
```

Minikube does not assign Public IP. This is why EXTERNAL-IP is pending. Use the following command to browse the application:
```sh
mininkube service <service name>
```

## Kubernetes Namespaces:
* **Default** - By default kubernetes objects goes here
* **kube-node-lease** - heartbeat of the nodes; determines the availability of the nodes
* **kube-system** - System processes; master and kubectl processes; DO NOT create or modify in this namespace
* **kube-public** - Publicly accessible data; ConfigMap that contains cluster info

Command to create a new namespace:
```sh
kubectl create namespace <namespace name>
```

* ConfigMap in one Namespace cannot be referenced in another namespace
* Service can be referenced in another namespace by appending namespace at the end e.g mongodb-service.development. Here 'development' is the name of a namespace

Global objects that cannot be put in a namespace - volumes, nodes. Command to check the Global objects:
```sh
kubectl api-resources --namespaced=false
```
Command to check the objects that can be put in a namespace:
```sh
kubectl api-resources --namespaced=true
```

Command to create an object in a namespace
```sh
kubectl apply -f <filename>.yaml -n <namespace>
```
or
mention the namespace in the metadata section (Best approach)
```sh
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: mongoteam
```

Command to get resources in a different namespace
```sh
kubectl get configmap -n <namespace>
```

### Change active namespace

Kubectl does not come with the solution to change active namespace. We have to install `kubens`

#### Installation on Mac
```sh
brew install kubectx
```
This will install kubens as well

#### kubens commands
Check the current namespace:
```sh
kubens
```

Change it to another namespace
```sh
kubens <namespace>
```

## Install Ingress Controller in Minikube

The following command automatically starts the K8s Nginx implementation of Ingress Controller
```sh
minikube addons enable ingress
```

Verify the ingress controller is installed
```sh
kubectl get pod -n kube-system
```

Minikube comes with out-of-box Kubernetes dashboard. To get information of the dashboard, use the following command
```sh
kubectl get all -n kubernetes-dashboard
```

Let's create an Ingress to access the dashboard

Error on MacBook Pro Intel Chip - Intel UHD Graphics 630 1536 MB
Exiting due to MK_ADDON_ENABLE: run callbacks: running callbacks: [waiting for app.kubernetes.io/name=ingress-nginx pods: timed out waiting for the condition]

```sh
minikube delete
```

Create the new cluster with docker driver but make sure docker is running:
```sh
minikube start --driver=docker --bootstrapper=kubeadm --kubernetes-version=latest --force
```

Create ingress for Kubernetes Dashboard
```sh
kubectl apply -f dashboard-ingress.yaml
```

Verify the ingress:
```sh
kubectl get ingress -n kubernetes-dashboard
```

Add the IP obtained from above command to /etc/hosts file to access the dashboard

### Another way to access minikube dashboard
```sh
minikube dashboard
```

Default Backend in Ingress
It is a default service when a path is not found - usually used for custom error messages