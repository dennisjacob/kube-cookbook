## COOKBOOK 02
## Creating pod with different options


#### create a pod with very basic attributes

```bash

root@k8s-ub-master:~# kubectl run test-pod --image=jacdenn/rhel7-httpd
pod/test-pod created


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: test-pod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod


```


#### Create pod with label

```bash
kubectl run test-pod --image=jacdenn/rhel7-httpd -l ENV=Prod  

apiVersion: v1
kind: Pod
metadata:
  labels:
    ENV: Prod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod

```

#### Create pod with label and port

```bash
kubectl run test-pod --image=jacdenn/rhel7-httpd -l ENV=Prod  --port=80

apiVersion: v1
kind: Pod
metadata:
  labels:
    ENV: Prod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod
    ports:
    - containerPort: 80

```

#### Create pod with label, port and expose it with default ClusterIP service ( remove the dry-run for actual run)

```bash
root@k8s-ub-master:~# kubectl run test-pod --image=jacdenn/rhel7-httpd -l ENV=Prod  --port=80 --expose --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    ENV: Prod
status:
  loadBalancer: {}
---
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

#### Create pod with label, port, and with resource limits and expose it with default ClusterIP service ( remove the dry-run for actual run)

```bash
root@k8s-ub-master:~# kubectl run test-pod --image=jacdenn/rhel7-httpd -l ENV=Prod --limits="cpu=2,memory=512Mi" --requests="cpu=100m,memory=64Mi" --port=80 --expose --dry-run=client -oyaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    ENV: Prod
status:
  loadBalancer: {}
---
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  containers:
  - image: jacdenn/rhel7-httpd
    name: test-pod
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: "2"
        memory: 512Mi
      requests:
        cpu: 100m
        memory: 64Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

#### Create pod with label, port, a command to run, and with resource limits and expose it with default ClusterIP service ( remove the dry-run for actual run)

```bash
root@k8s-ub-master:~# kubectl run test-pod --image=jacdenn/rhel7-httpd -l ENV=Prod --limits="cpu=2,memory=512Mi" --requests="cpu=100m,memory=64Mi" --port=80 --expose --dry-run=client -oyaml --command -- sleep infinity
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    ENV: Prod
status:
  loadBalancer: {}
---
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    ENV: Prod
  name: test-pod
spec:
  containers:
  - command:
    - sleep
    - infinity
    image: jacdenn/rhel7-httpd
    name: test-pod
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: "2"
        memory: 512Mi
      requests:
        cpu: 100m
        memory: 64Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

<<<<<<< HEAD
```
=======
```
>>>>>>> d9500542fc32cc03053793bf42cf507f4fb93ee7
