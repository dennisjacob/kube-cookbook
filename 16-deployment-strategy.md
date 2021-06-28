## COOKBOOK 16

## Deployment strategy can be set as RollingUpdate or Recreate


#### Create the deployment with Rolling Update, and with max new pods that can be created (maxSurge) is 3, and maximum pods that can be unavailable (maxUnavailable) as 2. Both can also be represented in percentage.

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: newapp
  name: newapp
spec:
  replicas: 8
  selector:
    matchLabels:
      app: newapp
  strategy:
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: newapp
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: webc


```

#### If the deployment is preferred in a BigBang, Recreate Option can be used. Explained with annotations for change-cause and rollout status

```bash
# Create a deployment definition with change-cause annotations to "change1". Strategy is set to Recreate.

root@k8s-ub-master:~# cat newapp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: newapp
  name: newapp
  annotations:
    kubernetes.io/change-cause: change1
spec:
  replicas: 8
  selector:
    matchLabels:
      app: newapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: newapp
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: webc


# Deploy the change
root@k8s-ub-master:~# kubectl apply -f newapp.yaml
deployment.apps/newapp configured


# Check the revision history
root@k8s-ub-master:~# kubectl rollout history deployment.apps/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
5         <none>
6         change1


#View the rollout status
root@k8s-ub-master:~# kubectl rollout status  deployment.apps/newapp
Waiting for deployment "newapp" rollout to finish: 0 out of 8 new replicas have been updated...
Waiting for deployment "newapp" rollout to finish: 0 out of 8 new replicas have been updated...
Waiting for deployment "newapp" rollout to finish: 0 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 1 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 2 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 3 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 4 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 5 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 6 of 8 updated replicas are available...
Waiting for deployment "newapp" rollout to finish: 7 of 8 updated replicas are available...
deployment "newapp" successfully rolled out
```

