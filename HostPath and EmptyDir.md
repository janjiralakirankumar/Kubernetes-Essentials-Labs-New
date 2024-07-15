## Task 1: Creating Deployment with hostPath
Create a file named mydep-hp.yaml using the content given below
```
vi mydep-hp.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydep-hp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mydep
  template:
    metadata:
      labels:
        app: mydep
    spec:
      containers:
      - image: nginx:latest
        name: con1
        ports:
        - containerPort: 80
        volumeMounts:
        - name: myvol
          mountPath: /data
      volumes:
      - name: myvol
        hostPath:
          path: /data
  ```
Save and exit the editor and Apply the Deployment yaml created in the previous step
```
kubectl apply -f mydep-hp.yaml
```
View the objects created by Kubernetes Deployment and verify hostPath mounting in POD
```
kubectl get deployments
```
```
kubectl get pods -o wide
```
From the above take a note on which pods are running on which particular node, as this would be required in the below steps.
```
kubectl describe pod <pod-name>
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -- /bin/bash
```
```
cd /data
```
```
echo 'Welcome to DevOps Training' > index.html
```
```
exit
```
Now delete your deployment
```
kubectl delete -f mydep-hp.yaml 
```
Now ssh into the Node on which the particular pod( in which you went inside and created the index.html file) was running.
```
ssh ubuntu@<External-IP>
```
Navigate to the Source location. In this case /data
```
cd /data
```
```
ls -la
```
You would see the index.html file still existing
```
cat index.html
```
```
exit
```


## Task 2 : Creating Deployment with emptyDir
Create a file named mydep-empty.yaml using content given below
```
vi mydep-empty.yaml
```
Paste the following content into the file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydep-empty
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mydep
  template:
    metadata:
      labels:
        app: mydep
    spec:
      containers:
      - image: nginx:latest
        name: con1
        ports:
        - containerPort: 80
        volumeMounts:
        - name: empty-vol
          mountPath: /data
      - image: tomcat:latest
        name: con2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: empty-vol
          mountPath: /opt/data
      volumes:
      - name: empty-vol
        emptyDir: {}
  ```
Save and exit the editor
Apply the Deployment yaml created in the previous step
```
kubectl apply -f mydep-empty.yaml
```
View the objects created by Kubernetes Deployment and verify hostPath mounting in POD
```
kubectl get deployments
```
```
kubectl get pods
```
```
kubectl describe pod <pod-name>
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
## Task 3: Cleanup the resources using the below command
Delete the resources created during the lab:
```
kubectl delete -f mydep-empty.yaml
```
```
kubectl delete -f mydep-hp.yaml
```
