## COOKBOOK 13

### Daemonsets deploy the pod , one per node. Creating a daemon set is easy, by modifying ( Remove replicas and kind)  a minimal deployment spec.

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: webapp
  name: webapp
spec:
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


# DaemonSet deployed per node
root@k8s-ub-master:~# kubectl get po -owide
NAME                         READY   STATUS      RESTARTS   AGE    IP              NODE           NOMINATED NODE   READINESS GATES
my-ds-5pvtm                  1/1     Running     0          69s    10.244.106.78   k8s-ub-node2   <none>           <none>
my-ds-7c6z4                  1/1     Running     0          69s    10.244.63.207   k8s-ub-node1   <none>           <none>

```
