## COOKBOOK 04
## Expose a deployment or replicaset for a service


#### Current state of deployment

```bash
root@k8s-ub-master:~# kubectl get deploy/webapp -owide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                SELECTOR
webapp   0/3     3            0           30s   rhel7-httpd   jacdenn/rhel7-httpd   app=webapp
```

#### Exposing a deployment with a port ( remove the dry-run for actual run)

```bash
root@k8s-ub-master:~# kubectl expose deploy/webapp --port=80 --session-affinity=ClientIP --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webapp
  sessionAffinity: ClientIP
status:
  loadBalancer: {}


# Check the service
# Ways to access  is using cluster_ip:80  or endpoint:80
root@k8s-ub-master:~# kubectl get service/webapp -owide
NAME     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
webapp   ClusterIP   10.96.54.67   <none>        80/TCP    3s    app=webapp

```

#### Exposing a deployment with NodePort ( remove the dry-run for actual run) 

```bash

root@k8s-ub-master:~# kubectl expose deploy/webapp --port=80 --type=NodePort --session-affinity=ClientIP --dry-run=client -oyaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webapp
  sessionAffinity: ClientIP
  type: NodePort
status:
  loadBalancer: {}

# Check the service
# Ways to access  is using cluster_ip:80  or endpoint:80 or node:31368

root@k8s-ub-master:~# kubectl get service/webapp -owide
NAME     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   SELECTOR
webapp   NodePort   10.99.49.224   <none>        80:31368/TCP   9s    app=webapp

```


#### Exposing a deployment with NodePort ( remove the dry-run for actual run), and uses a defined clusterIP port

```bash

root@k8s-ub-master:~# kubectl expose deploy/webapp --port=8080 --target-port=80 --type=NodePort --session-affinity=ClientIP --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: webapp
  sessionAffinity: ClientIP
  type: NodePort
status:
  loadBalancer: {}

# Check the service
# Ways to access  is using cluster_ip:8080  or endpoint:80 or node:30599
root@k8s-ub-master:~# kubectl get service/webapp -owide
NAME     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE   SELECTOR
webapp   NodePort   10.99.89.14   <none>        8080:30599/TCP   3s    app=webapp
root@k8s-ub-master:~#

```


#### Exposing a deployment with Loadbalancer ( remove the dry-run for actual run), and uses a defined clusterIP port. This requires a load balancer IPs  available to use.
#### Alternatively, you may use metalLB



```bash
root@k8s-ub-master:~# kubectl expose deploy/webapp --port=8080 --target-port=80 --type=LoadBalancer --session-affinity=ClientIP --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webapp
  name: webapp
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: webapp
  sessionAffinity: ClientIP
  type: LoadBalancer
status:
  loadBalancer: {}


# Check the service
# Ways to access  is using cluster_ip:8080  or loadbalancer:8080 or endpoint:80 or node:32520

root@k8s-ub-master:~# kubectl get service/webapp -owide
NAME     TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE   SELECTOR
webapp   LoadBalancer   10.97.183.234   192.168.52.16   8080:32520/TCP   7s    app=webapp

```


#### Expose a replicaset

```bash
# Created a replicaset
root@k8s-ub-master:~# kubectl create deployment web --image=nginx --dry-run=client -oyaml | sed -e 's/Deployment/ReplicaSet/' -e '/strategy\|status/d'  | kubectl apply -f -
replicaset.apps/web created

root@k8s-ub-master:~# kubectl get rs/web -owide
NAME   DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES   SELECTOR
web    1         1         1       2m13s   nginx        nginx    app=web

#expose the replicaset
root@k8s-ub-master:~# kubectl expose rs/web --port=80 --target-port=80 --type=LoadBalancer --session-affinity=ClientIP --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
  sessionAffinity: ClientIP
  type: LoadBalancer
status:
  loadBalancer: {}

```



