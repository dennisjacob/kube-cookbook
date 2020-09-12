## COOKBOOK 20

## Environment Varaibles


#### Create a secret as an environment variables, typically like passwords loaded on env variable ( remove the dry-run for actual run).

```bash
root@k8s-ub-master:~# kubectl create secret generic sec1 --from-literal=DB_PASSWORD=testPassword --from-literal=DB_USER=user1 --dry-run=client -oyaml
apiVersion: v1
data:
  DB_PASSWORD: dGVzdFBhc3N3b3Jk
  DB_USER: dXNlcjE=
kind: Secret
metadata:
  name: sec1

# Testing to check the encoded string
root@k8s-ub-master:~# echo -n dGVzdFBhc3N3b3Jk | base64 -d
testPassword

root@k8s-ub-master:~# kubectl get secrets
NAME                  TYPE                                  DATA   AGE
sec1                  Opaque                                2      13s

# Deploy a test application

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        envFrom:
        - secretRef:
            name: sec1


# Check  the environment variables in the pod
root@k8s-ub-master:~# kubectl exec -it po/app1-54b55966c6-qhw9p -- env | grep DB
DB_PASSWORD=testPassword
DB_USER=user1


```

#### Create a configmap as an environment variables, typically like passwords loaded on env variable ( remove the dry-run for actual run).

```bash

kubectl create cm cm1 --from-literal=ENV=PROD --from-literal=CAT=ALPHA001 --dry-run=client -oyaml

# Deploy an application with env varaibles from configMap

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        envFrom:
        - configMapRef:
            name: cm1


root@k8s-ub-master:~# kubectl exec -it po/app1-7476d8cc65-rh4mz -- env | grep -E 'CAT|ENV'
CAT=ALPHA001
ENV=PROD

```


#### Create a secret as an environment variables, typically like passwords loaded on env variable , and use with env key valueFrom ( remove the dry-run for actual run).

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: sec1
              key: DB_USER


#  Check the value
root@k8s-ub-master:~# kubectl exec -it po/app1-69cb446b64-qwcjj -- env | grep DB
DB_USER=user1

```

#### Create a configmap as an environment variables, typically like passwords loaded on env variable , and use with env key valueFrom ( remove the dry-run for actual run).

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        env:
        - name: CAT
          valueFrom:
            configMapKeyRef:
              name: cm1
              key: CAT

#  Check the value
root@k8s-ub-master:~# kubectl exec -it po/app1-58b779d6d9-t9tbv -- env | grep CAT
CAT=ALPHA001


```

#### Create the secret as environment variables from a env variable file instead of literal option

```bash
root@k8s-ub-master:~# kubectl create secret generic sec2 --from-env-file=env_file --dry-run=client -oyaml
apiVersion: v1
data:
  DB_PASSWORD: cGFzc3dvcmQ=
  DB_USER: dXNlcjE=
kind: Secret
metadata:
  creationTimestamp: null
  name: sec2

```

#### Create the configmap as environment variables from a env variable file instead of literal option

```bash

kubectl create cm  cm3 --from-env-file=start.sh

# create a deployment with config map

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: app1
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: rhel7-custom
        volumeMounts:
        - name: my-share
          mountPath: /var/mware
      volumes:
      - name: my-share
        configMap:
          name: cm3


#Validate the change in pod

root@k8s-ub-master:~# kubectl exec -it po/app1-67678d4947-hc9wq -- ls -l /var/mware
total 0
lrwxrwxrwx 1 root root 15 Jun 13 09:00 start.sh -> ..data/start.sh


```