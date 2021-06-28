## COOKBOOK 06
## View, Create, modify and delete labels.


#### View the labels

```bash
root@k8s-ub-master:~# kubectl get deploy -owide --show-labels
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                SELECTOR     LABELS
webapp   3/3     3            3           33m   rhel7-httpd   jacdenn/rhel7-httpd   app=webapp   app=webapp
```

#### Add a new label

```bash

# Add a label to the metadata.labels in deployment definition
root@k8s-ub-master:~# kubectl label deploy/webapp tier=app
deployment.apps/webapp labeled

# Check labels again
root@k8s-ub-master:~# kubectl get deploy -owide --show-labels
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                SELECTOR     LABELS
webapp   3/3     3            3           33m   rhel7-httpd   jacdenn/rhel7-httpd   app=webapp   app=webapp,tier=app

```

#### Overwrite an existing label

```

root@k8s-ub-master:~# kubectl label deploy/webapp tier=apptier --overwrite
deployment.apps/webapp labeled

# Check labels again
root@k8s-ub-master:~# kubectl get deploy -owide --show-labels
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                SELECTOR     LABELS
webapp   3/3     3            3           49m   rhel7-httpd   jacdenn/rhel7-httpd   app=webapp   app=webapp,tier=apptier

```


#### Create a pod with labels

```bash
root@k8s-ub-master:~# kubectl run test-pod --image=jacdenn/rhel7-custom -l pod-tier=app
pod/test-pod created

root@k8s-ub-master:~# kubectl get po/test-pod --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
test-pod   1/1     Running   0          12s   pod-tier=app

```

#### List pods with a particular label

```bash
root@k8s-ub-master:~# kubectl get po -A -l pod-tier=app -owide --show-labels
NAMESPACE   NAME       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES   LABELS
my-ns       test-pod   1/1     Running   0          98s   10.244.106.76   k8s-ub-node2   <none>           <none>            pod-tier=app

```


#### Remove the labels

```bash

# Check existing labels
root@k8s-ub-master:~# kubectl get po -A -l pod-tier=app -owide --show-labels
NAMESPACE   NAME       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES   LABELS
my-ns       test-pod   1/1     Running   0          82m   10.244.106.76   k8s-ub-node2   <none>           <none>            pod-tier=app,web=nginx

# Remove the labels
root@k8s-ub-master:~# kubectl label po/test-pod web-
pod/test-pod labeled

# Check the labels again
root@k8s-ub-master:~# kubectl get po -A -l pod-tier=app -owide --show-labels
NAMESPACE   NAME       READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES   LABELS
my-ns       test-pod   1/1     Running   0          83m   10.244.106.76   k8s-ub-node2   <none>           <none>            pod-tier=app

```


#### Set a label to a node, and show the label alone

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

```