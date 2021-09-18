# Kubernetes  - PODs

## Step-01: PODs Introduction
- What is a POD ?
- What is a Multi-Container POD?

## Step-02: PODs Demo
### Get Worker Nodes Status
- Verify if kubernetes worker nodes are ready. 
```
# Get Worker Node Status
kubectl get nodes

# Get Worker Node Status with wide option
kubectl get nodes -o wide
```

### Create a Pod
- Create a Pod
```
# Template
kubectl run <desired-pod-name> --image <Container-Image>

kubectl run rtls-pod --image stacksimplify/kubenginx:1.0.0
```
### List Pods
- Get the list of pods
```
# List Pods
kubectl get pods

# Alias name for pods is po
kubectl get po
```

### List Pods with wide option
- List pods with wide option which also provide Node information on which Pod is running
```
kubectl get pods -o wide
```

### What happened in the backgroup when above command is run?
  1. Kubernetes created a pod
  2. Pulled the docker image from docker hub
  3. Created the container in the pod
  4. Started the container present in the pod


### Describe Pod
- Describe the POD, primarily required during troubleshooting. 
- Events shown will be of a great help during troubleshooting. 
```
# To get list of pod names
kubectl get pods

# Describe the Pod
kubectl describe pod <Pod-Name>
kubectl describe pod rtls-pod 
```

### Access Application
- Currently we can access this application only inside worker nodes. 
- To access it externally, we need to create a **NodePort Service**. 
- **Services** is one very very important concept in Kubernetes. 


### Delete Pod
```
# To get list of pod names
kubectl get pods

# Delete Pod
kubectl delete pod <Pod-Name>
kubectl delete pod rtls-pod
```

## Step-03: NodePort Service Introduction
- What are Services in k8s?
- What is a NodePort Service?
- How it works?

## Step-04: Demo - Expose Pod with a Service
- Expose pod with a service (NodePort Service) to access the application externally (from internet)
- **Ports**
  - **port:** Port on which node port service listens in Kubernetes cluster internally
  - **targetPort:** We define container port here on which our application is running.
  - **NodePort:** Worker Node port on which we can access our application.
```
# Create  a Pod
kubectl run <desired-pod-name> --image <Container-Image>
kubectl run rtls-pod --image stacksimplify/kubenginx:1.0.0

# Expose Pod as a Service
kubectl expose pod <Pod-Name>  --type=NodePort --port=80 --name=<Service-Name>
kubectl expose pod rtls-pod  --type=NodePort --port=80 --name=rtls-service

# Get Service Info
kubectl get service
kubectl get svc

# Get Public IP of Worker Nodes
kubectl get nodes -o wide
```
- **Access the Application using Public IP**
```
http://<node1-public-ip>:<Node-Port>
```

- **Important Note about: target-port**
  -  If target-port is not defined, by default and for convenience, the **targetPort** is set to the same value as the **port** field.

```
# Below command will fail when accessing the application, as service port (81) and container port (80) are different
kubectl expose pod rtls-pod  --type=NodePort --port=81 --name=rtls-service2     

# Expose Pod as a Service with Container Port (--taret-port)
kubectl expose pod rtls-pod  --type=NodePort --port=81 --target-port=80 --name=rtls-service3

# Get Service Info
kubectl get service
kubectl get svc

# Get Public IP of Worker Nodes
kubectl get nodes -o wide
```
- **Access the Application using Public IP**
```
http://<node1-public-ip>:<Node-Port>
```

## Step-05: Interact with a Pod

### Verify Pod Logs
```
# Get Pod Name
kubectl get po

# Dump Pod logs
kubectl logs <pod-name>
kubectl logs rtls-pod

# Stream pod logs with -f option and access application to see logs
kubectl logs <pod-name>
kubectl logs -f rtls-pod
```
- **Important Notes**
  - Refer below link and search for **Interacting with running Pods** for additional log options
  - Troubleshooting skills are very important. So please go through all logging options available and master them.
  - **Reference:** https://kubernetes.io/docs/reference/kubectl/cheatsheet/

### Connect to Container in a POD
- **Connect to a Container in POD and execute commands**
```
# Connect to Nginx Container in a POD
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it rtls-pod -- /bin/bash

# Execute some commands in Nginx container
ls
cd /usr/share/nginx/html
cat index.html
exit
```

- **Running individual commands in a Container**
```
kubectl exec -it <pod-name> env

# Sample Commands
kubectl exec -it rtls-pod env
kubectl exec -it rtls-pod ls
kubectl exec -it rtls-pod cat /usr/share/nginx/html/index.html
```
## Step-06: Get YAML Output of Pod & Service
### Get YAML Output
```
# Get pod definition YAML output
kubectl get pod rtls-pod -o yaml   

# Get service definition YAML output
kubectl get service rtls-service -o yaml   
```

## Step-07: Clean-Up
```
# Get all Objects in default namespace
kubectl get all

# Delete Services
kubectl delete svc rtls-service
kubectl delete svc rtls-service2
kubectl delete svc rtls-service3

# Delete Pod
kubectl delete pod rtls-pod

# Get all Objects in default namespace
kubectl get all
```