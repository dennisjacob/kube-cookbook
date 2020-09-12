## COOKBOOK 14

## Check the various options of rollout


#### Check the rollout history for the past rollouts

```bash

# Create a deployment, and with a change-cause annotation CHG101

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: newapp
  name: newapp
  annotations:
    kubernetes.io/change-cause: CHG101
spec:
  replicas: 3
  selector:
    matchLabels:
      app: newapp
  template:
    metadata:
      labels:
        app: newapp
        tier: main-CHG103
    spec:
      containers:
      - image: jacdenn/rhel7-custom
        name: container1


root@k8s-ub-master:~# kubectl apply -f newapp.yaml
deployment.apps/newapp created

root@k8s-ub-master:~# kubectl rollout status deployment.apps/newapp
deployment "newapp" successfully rolled out

root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
1         CHG101

```

#### Make another few changes and check the revisions

```bash

root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
1         CHG101
2         CHG102
3         CHG103

```

#### Undo and rollback to old revision

```bash
root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
1         CHG101
2         CHG102
3         CHG103

# Rollback the last deployment 

root@k8s-ub-master:~# kubectl rollout undo deploy/newapp
deployment.apps/newapp rolled back

root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
1         CHG101
3         CHG103
4         CHG102


# Rollback to a particular revision CHG101
root@k8s-ub-master:~# kubectl rollout undo deploy/newapp --to-revision=1
deployment.apps/newapp rolled back

root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
3         CHG103
4         CHG102
5         CHG101


```

#### Restart a deployment. This will create a new revision with same change cause.

```bash
root@k8s-ub-master:~# kubectl rollout restart deploy/newapp
deployment.apps/newapp restarted

root@k8s-ub-master:~# kubectl rollout history deploy/newapp
deployment.apps/newapp
REVISION  CHANGE-CAUSE
3         CHG103
4         CHG102
5         CHG101
6         CHG101

```
