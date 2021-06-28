## COOKBOOK 07

## Use nodeName and nodeSelector to deploy a pod or deployment in a particular node
## Using nodeAffinity is another approach


#### Make the pod to deploy on a particular node

```bash

# Use the po.spec.nodeName to set the node where the pod needs to be deployed

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod
    resources: {}
  nodeName: k8s-ub-node2

```

####  Use nodeSelector

```bash

# show the labels with name priority
root@k8s-ub-master:~# kubectl get no -L priority
NAME            STATUS   ROLES    AGE   VERSION   PRIORITY
k8s-ub-master   Ready    master   39d   v1.18.2
k8s-ub-node1    Ready    <none>   39d   v1.18.2
k8s-ub-node2    Ready    <none>   39d   v1.18.2

# Set the label
root@k8s-ub-master:~# kubectl label no/k8s-ub-node1 priority=medium
node/k8s-ub-node1 labeled

# Check the labels agian with name priority
root@k8s-ub-master:~# kubectl get no -L priority
NAME            STATUS   ROLES    AGE   VERSION   PRIORITY
k8s-ub-master   Ready    master   39d   v1.18.2
k8s-ub-node1    Ready    <none>   39d   v1.18.2   medium
k8s-ub-node2    Ready    <none>   39d   v1.18.2


# Create the definition with nodeSelector
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod
    resources: {}
  nodeSelector:
    priority: medium


# Verify that pod deployed on the k8s-ub-node1
root@k8s-ub-master:~# kubectl get po/test-pod -owide
NAME       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
test-pod   1/1     Running   0          43s   10.244.63.203   k8s-ub-node1   <none>           <none>

```