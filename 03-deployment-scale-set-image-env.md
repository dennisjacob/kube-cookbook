## COOKBOOK 03
## Creating deployment and scale it, change image

#### Creating deployment ( remove the dry-run for actual run)
```bash
root@k8s-ub-master:~# kubectl create deployment test-deploy --image=jacdenn/rhel7-httpd --dry-run=client -oyaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test-deploy
  name: test-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test-deploy
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: rhel7-httpd
        resources: {}
status: {}
```


#### Scale up the replicas to 3
```
kubectl scale deploy/test-deploy --replicas=3

root@k8s-ub-master:~# kubectl get deploy/test-deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
test-deploy   3/3     3            3           24m

```

#### Change the image of the container
```bash
# Change th image of the container with name rhel7-httpd
root@k8s-ub-master:~# kubectl set image deploy/test-deploy rhel7-httpd=jacdenn/rhel7-custom
deployment.apps/test-deploy image updated

root@k8s-ub-master:~# kubectl get deploy/test-deploy -o=jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
jacdenn/rhel7-custom

```

#### Set a environment variable in the container of the deployment
```bash
root@k8s-ub-master:~# kubectl set env deploy/test-deploy ENV=PROD
deployment.apps/test-deploy env updated

root@k8s-ub-master:~# kubectl get deploy/test-deploy -o=jsonpath='{.spec.template.spec.containers[0].env[*].name}={.spec.template.spec.containers[0].env[*].value}{"\n"}'
ENV=PROD
```

#### Quick way to create a replicaset

```bash
root@k8s-ub-master:~# kubectl create deployment web --image=nginx --dry-run=client -oyaml | sed -e 's/Deployment/ReplicaSet/' -e '/strategy\|status/d'  | kubectl apply -f -
replicaset.apps/web created

root@k8s-ub-master:~# kubectl get rs/web
NAME   DESIRED   CURRENT   READY   AGE
web    1         1         1       20s


```