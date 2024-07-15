## Namespace in Kubernetes

### Task 1: Understanding Namespace commands

To create a namespace
```
kubectl create ns test-ns
```
To list all the available namespaces
```
kubectl get ns
```
To list all  the objects in a particular name-space
```
kubectl -n kube-system get all
```
To describe a namespace
```
kubectl describe ns kube-system
```


## Pod in Kubernetes

### Task 2: Create a pod using below Commands
```
kubectl run pod-1 --image nginx --port 80 
```
Check the newly created Pod
```
kubectl get pod
```
```
kubectl get pod -o wide
```
Describe Pod using below command
``` 
kubectl describe pod pod-1
```
Generate spec for running pod nginx and write it into a file called pod.yaml 
```
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
``` 

### Task 3: Create a pod using below yaml
To check the version of the Object you are creating
```
kubectl api-resources
```
Let's create the YAML file
```
vi httpd-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    env: prod 
    type: front-end
    app: httpd-ws
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
       - containerPort: 80
``` 
Apply the pod definition yaml
```
kubectl apply -f httpd-pod.yaml
```
Check the newly created Pod
```
kubectl get pods
```
```
kubectl get pods -o wide
```
Describe Pod using below command
```
kubectl describe pod httpd-pod
```
Check the labels of the Pod.
```
kubectl get pod --show-labels
```
To add label to a running pod
```
kubectl label pod httpd-pod cloudthat=k8s
```
Check the labels again
```
kubectl get pod httpd-pod --show-labels
```
To add a label when creating a prod
```
kubectl run test-pod --image nginx --labels task=3
```
Enter inside the container of the pod
```
kubectl exec -it test-pod -- /bin/bash
```
```
exit
```
