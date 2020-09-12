## COOKBOOK 12

### You can have multiple schedulers configured in a cluster, and can decide which scheduler to do scheduling for the pods of your deployment.



#### Turn off the leader elect of the existing scheduler, and create a new scheduler as a static pod.

```bash

# Modified the existing  scheduler static pod definition in /etc/kubernetes/manifests , to have --leader-elect=false

oot@k8s-ub-master:/etc/kubernetes/manifests# cat kube-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    image: k8s.gcr.io/kube-scheduler:v1.18.2
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}


# Created a new static pod for my custom scheduler "my-scheduler" with --port, --scheduler-name  and --secure-port specified apart from --leader-elect

root@k8s-ub-master:/etc/kubernetes/manifests# cat my-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --port=10281
    - --secure-port=10289
    - --scheduler-name=my-scheduler
    image: k8s.gcr.io/kube-scheduler:v1.18.2
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}

# Check the scheduler pod

root@k8s-ub-master:/etc/kubernetes/manifests# kubectl get po -n kube-system | grep scheduler
kube-scheduler-k8s-ub-master               1/1     Running   0          5m32s
my-scheduler-k8s-ub-master                 1/1     Running   0          5m16s
root@k8s-ub-master:/etc/kubernetes/manifests#


```

#### Create a deployment with pods to use the my-scheduler schedculer

```bash
# Create a deployment with schedulerName set

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webapp
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy: {}
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        command:
        - sh
        - -c
        - >-
          while true; do echo "Hello from $(hostname) at $(date)"; sleep 5; done
      schedulerName: my-scheduler

```