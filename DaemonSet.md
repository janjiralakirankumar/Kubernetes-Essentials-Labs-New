## DaemonSet

Create a DaemonSet using below yaml
```
vi ds-pod.yaml
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-ds
  labels:
    app: fluent-ds
spec:
  selector:
    matchLabels:
      app: fluentd-app
  template:
    metadata:
       labels:
           app: fluentd-app
    spec:
      containers:
      - name: fluentd-ctr
        image: nginx
```
Apply the yaml definition to create a fluent-ds DaemonSet
```
kubectl apply -f ds-pod.yaml
```
Check the available daemonsets in kubernetes cluster
```
kubectl get ds fluent-ds
```
Verify that pods for fluent are created one for each node using DaemonSet
```
kubectl get pods -o wide
```
Increase the worked node count to 3 from the AutoScaling Group on the AWS Management Console and check again the pod count. You would notice a new pod automatically getting deployed to the new node.
```
kubectl get pods -o wide
```
Cleanup the DaemeonSet using below 
```
kubectl delete -f ds-pod.yaml
```
Verify the pods to find (all) the fluentd pods being deleted from each of the nodes
```
kubectl get pods
```
