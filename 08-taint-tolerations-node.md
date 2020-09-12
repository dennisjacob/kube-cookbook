## COOKBOOK 08
## Create the taints on nodes, and deploy pods with tolerations

#### Check if there is any tolerations on nodes

```bash

root@k8s-ub-master:~# for each in $(kubectl get no -o=custom-columns=NAME:.metadata.name | awk 'NR>1' | xargs); do echo "$each $(kubectl describe no/$each | grep Taint)"; done
k8s-ub-master Taints:             node-role.kubernetes.io/master:NoSchedule
k8s-ub-node1 Taints:             <none>
k8s-ub-node2 Taints:             <none>

```

#### Set a taint on k8s-ub-node1 with a NoExecute, which evicts the pods on the node

```bash
# Pod deployment distribution

root@k8s-ub-master:~# kubectl get po -owide
NAME                      READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
newapp-6d988c8f46-lphcq   1/1     Running   1          34d   10.244.106.70   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-mcxqm   1/1     Running   1          34d   10.244.106.71   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-nt4nv   1/1     Running   1          34d   10.244.63.198   k8s-ub-node1   <none>           <none>
newapp-6d988c8f46-zwf9n   1/1     Running   1          34d   10.244.63.199   k8s-ub-node1   <none>           <none>
webapp-59d9889648-4gbx2   1/1     Running   0          23s   10.244.63.200   k8s-ub-node1   <none>           <none>
webapp-59d9889648-8hp45   1/1     Running   0          13s   10.244.106.74   k8s-ub-node2   <none>           <none>
webapp-59d9889648-m7b8w   1/1     Running   0          13s   10.244.63.201   k8s-ub-node1   <none>           <none>

# Putting a taint on k8s-ub-node1 with NoExecute effect, which will evict the pods on that node
root@k8s-ub-master:~# kubectl taint nodes k8s-ub-node1 priority=medium:NoExecute
node/k8s-ub-node1 tainted

# Pod deployments after the taint applied

root@k8s-ub-master:~# kubectl get po -owide
NAME                      READY   STATUS    RESTARTS   AGE     IP              NODE           NOMINATED NODE   READINESS GATES
newapp-6d988c8f46-2cdb5   1/1     Running   0          47s     10.244.106.78   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-lphcq   1/1     Running   1          34d     10.244.106.70   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-mcxqm   1/1     Running   1          34d     10.244.106.71   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-vxmwv   1/1     Running   0          47s     10.244.106.77   k8s-ub-node2   <none>           <none>
webapp-59d9889648-8hp45   1/1     Running   0          2m11s   10.244.106.74   k8s-ub-node2   <none>           <none>
webapp-59d9889648-kr48f   1/1     Running   0          46s     10.244.106.76   k8s-ub-node2   <none>           <none>
webapp-59d9889648-tg2hh   1/1     Running   0          47s     10.244.106.79   k8s-ub-node2   <none>           <none>

```

#### Create a deployment with toleration on the taint, so that pods will get deployed eventhough the taint applied on that node. Tolerations do not guarentee any affinity, and you need to use nodeAffinity for that.

```bash
# Deployment definition with tolerations

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
      tolerations:
      - key: priority
        operator: Equal
        value: medium
        effect: NoExecute


# Pod deployment distribution. websrvr deployed on the tainted k8s-ub-node1. It also can deploy on other nodes depends how scheduler decide the pod distribution.

root@k8s-ub-master:~# kubectl get po -owide
NAME                      READY   STATUS    RESTARTS   AGE     IP              NODE           NOMINATED NODE   READINESS GATES
newapp-6d988c8f46-2cdb5   1/1     Running   0          7m34s   10.244.106.78   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-lphcq   1/1     Running   1          34d     10.244.106.70   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-mcxqm   1/1     Running   1          34d     10.244.106.71   k8s-ub-node2   <none>           <none>
newapp-6d988c8f46-vxmwv   1/1     Running   0          7m34s   10.244.106.77   k8s-ub-node2   <none>           <none>
webapp-59d9889648-8hp45   1/1     Running   0          8m58s   10.244.106.74   k8s-ub-node2   <none>           <none>
webapp-59d9889648-kr48f   1/1     Running   0          7m33s   10.244.106.76   k8s-ub-node2   <none>           <none>
webapp-59d9889648-tg2hh   1/1     Running   0          7m34s   10.244.106.79   k8s-ub-node2   <none>           <none>
websrvr-98989f884-495rg   1/1     Running   0          14s     10.244.63.204   k8s-ub-node1   <none>           <none>
websrvr-98989f884-bh4xb   1/1     Running   0          14s     10.244.63.203   k8s-ub-node1   <none>           <none>
websrvr-98989f884-c4gzm   1/1     Running   0          14s     10.244.63.202   k8s-ub-node1   <none>           <none>


```

#### Remove the taint 

```bash

# Remove the taint
root@k8s-ub-master:~# kubectl taint nodes k8s-ub-node1 priority-
node/k8s-ub-node1 untainted

# Redeploying the deployments to see if the pods are now distributed on nodes including k8s-ub-node1
root@k8s-ub-master:~# kubectl rollout restart deploy/webapp
deployment.apps/webapp restarted
root@k8s-ub-master:~# kubectl rollout restart deploy/newapp
deployment.apps/newapp restarted
root@k8s-ub-master:~# kubectl rollout restart deploy/websrvr
deployment.apps/websrvr restarted

# Looks fine now
root@k8s-ub-master:~# kubectl get po -owide
NAME                       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
newapp-6dcd6dffd-47mx4     1/1     Running   0          81s   10.244.63.206   k8s-ub-node1   <none>           <none>
newapp-6dcd6dffd-d87wr     1/1     Running   0          76s   10.244.63.209   k8s-ub-node1   <none>           <none>
newapp-6dcd6dffd-rqj5d     1/1     Running   0          81s   10.244.63.207   k8s-ub-node1   <none>           <none>
newapp-6dcd6dffd-zjp4g     1/1     Running   0          73s   10.244.63.211   k8s-ub-node1   <none>           <none>
webapp-648b5744ff-l85xd    1/1     Running   0          69s   10.244.106.80   k8s-ub-node2   <none>           <none>
webapp-648b5744ff-n5v5v    1/1     Running   0          80s   10.244.63.208   k8s-ub-node1   <none>           <none>
webapp-648b5744ff-q2vqk    1/1     Running   0          86s   10.244.63.205   k8s-ub-node1   <none>           <none>
websrvr-5fc6998fc9-pl47d   1/1     Running   0          63s   10.244.106.81   k8s-ub-node2   <none>           <none>
websrvr-5fc6998fc9-q7bbq   1/1     Running   0          73s   10.244.63.210   k8s-ub-node1   <none>           <none>
websrvr-5fc6998fc9-zvfhk   1/1     Running   0          58s   10.244.106.82   k8s-ub-node2   <none>           <none>
root@k8s-ub-master:~#


```
