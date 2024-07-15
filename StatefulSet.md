## StatefulSet

### Task 1: Create Stateful Set
Create the yaml definition for an nginx Stateful Set 
```
vi nginx-sts.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx-svc
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-sts
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sts
spec:
  serviceName: nginx-svc
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Create a Stateful Set and  headless service by applying the yaml
```
kubectl apply -f nginx-sts.yaml
```
Validate the headless service creation
```
kubectl get service nginx-svc
```
Validate the stateful set creation
```
kubectl get statefulset nginx-sts
```
Watch the pods getting created in an ordinal index fashion
```
kubectl get pods -w -l app=nginx-sts
```
Go to each of the pods to see the hostname, as a DNS entry is created for each Pod, the hostname inside the Pod should match the pod-name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```
Create a busybox pod to test the ping to one of the Ngninx pods by using DNS name of the pod
```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
```
nslookup nginx-sts-0.nginx-svc
```
```
nslookup nginx-sts-1.nginx-svc
```
```
exit
```
Delete the pods for a stateful set
```
kubectl delete pod -l app=nginx-sts
```
In another window notice the new pods getting created in a proper order
```
kubectl get pod -w -l app=nginx-sts
```
Verify the hostname in each of the newly created pods, it will match the pod name
```
for i in 0 1; do kubectl exec "nginx-sts-$i" -- sh -c 'hostname'; done
```

### Task 2: Scaling a Stateful Set
Scale the Stateful Set to 5 replicas using below.
```
kubectl scale sts nginx-sts --replicas=5
```
Verify the pods getting created in ordinal way
```
kubectl get pods -w -l app=nginx-sts
```
Verify the PV Claim getting created in ordinal fashion
```
kubectl get pvc -l app=nginx-sts
```
Edit the stateful Set yam and reduce replicas to 3 
```
kubectl edit sts nginx-sts
```
Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next
```
kubectl get pods -w -l app=nginx-sts
```
Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 
```
kubectl get pvc -l app=nginx-sts
```

### Task 3: Cleanup the resources using below command 
Delete a Stateful Set
```
kubectl delete -f nginx-sts.yaml
```
List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.
```
kubectl get pvc
```
```
kubectl delete pvc --all
```
