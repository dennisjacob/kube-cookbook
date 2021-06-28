## COOKBOOK 09

## Use affinity ( nodeAffinity) for deployment


#### Set a label to the node

```bash

root@k8s-ub-master:~# kubectl label no/k8s-ub-node2 priority=high
node/k8s-ub-node2 labeled

# check the labels
root@k8s-ub-master:~# kubectl get no -L priority
NAME            STATUS   ROLES    AGE   VERSION   PRIORITY
k8s-ub-master   Ready    master   40d   v1.18.2
k8s-ub-node1    Ready    <none>   40d   v1.18.2
k8s-ub-node2    Ready    <none>   40d   v1.18.2   high

```

#### Create a deployment with affinity

```bash
# Define a deployment with nodeAffinity

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: websrvr
  name: websrvr
spec:
  replicas: 3
  selector:
    matchLabels:
      app: websrvr
  template:
    metadata:
      labels:
        app: websrvr
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: priority
                operator: In
                values:
                - high


## Check the pod deployed to see if its all deployed on k8s-ub-node2 
root@k8s-ub-master:~# kubectl get po -owide
NAME                       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
websrvr-55778d54c9-mcb9z   1/1     Running   0          41s   10.244.106.85   k8s-ub-node2   <none>           <none>
websrvr-55778d54c9-nw7r7   1/1     Running   0          41s   10.244.106.83   k8s-ub-node2   <none>           <none>
websrvr-55778d54c9-q8pzg   1/1     Running   0          41s   10.244.106.84   k8s-ub-node2   <none>           <none>

```
