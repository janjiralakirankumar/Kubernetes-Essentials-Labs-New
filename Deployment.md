## Deployment

### Task 1: Write a Deployment yaml and Apply it
Create a dep-nginx.yaml using content given below
```
vi dep-nginx.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80
```
Apply the Deployment yaml created in the previous step
```
kubectl apply -f dep-nginx.yaml
```
View the objects created by Kubernetes, Deployment and Replica Set 
```
kubectl get deployments
```
```
kubectl get rs
```
Access one of the Pods and view nginx version
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```

### Task 2: Update the Deployment with a Newer Image
Update the nginx image in Pod using below
```
kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11
```
Describe the deployment and see that the old pods are replaced with newer ones
```
kubectl describe deployments
```
Access one of the Pods and view nginx version
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```

### Task 3: Rollback of Deployment 
View the history of Deployments
```
kubectl rollout history deployment/nginx-dep
```
Rollback the Deployment done in the previous task
```
kubectl rollout undo deployment/nginx-dep --to-revision=1
```
```
kubectl get rs
```
Access one of the Pods and view nginx version
```
kubectl get pods
```
```
kubectl exec -it <pod_name> -- /bin/bash
```
```
nginx -v
```
```
exit
```

### Task 4: Scaling of Deployments
View the number of Pod replicas created by the Deployment
```
kubectl get deployments
```
```
kubectl get pods
```
Scale up the deployment to have 8 Pod replicas
```
kubectl scale deployment nginx-dep --replicas=8
```
Check the Pods and deployment to and verify that the number of Pod replicas are 8
```
kubectl get deployments
```
```
kubectl get pods
```
Scale down the deployments to 2 Pod replicas
```
kubectl scale deployment nginx-dep --replicas=2
```
Check the Pods and deployment to and verify that the number of Pod replicas are down to 2
```
kubectl get deployments
```
```
kubectl get pods
```

### Task 5: Deployment Auto Scaling HPA – Horizontal Pod 
Autoscaling
```
kubectl autoscale deployment nginx-dep --min=3 --max=8 --cpu-percent=70
```
```
kubectl get hpa
```

### Task 6: Cleanup the resources using below command
```
kubectl delete -f dep-nginx.yaml
```
