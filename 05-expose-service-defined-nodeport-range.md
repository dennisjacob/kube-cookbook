## COOKBOOK 05
## Exposing a deployment with NodePort,  and uses a defined clusterIP port and a defined NodePort range.


Assumption: The cluster is configured using kubeadm

#### Modify the manifest file on the node where the api-server(s) are running to include service-node-port-range to allow ports from 80-32767.
#### kube-api server will be restarted automatically. Or, forcefully with a kublet restart.

```bash
    - --service-node-port-range=80-32767
```

#### Create the service with NodePort specified in the svc definition yaml

```bash
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
    nodePort: 8443
  selector:
    app: webapp
  sessionAffinity: ClientIP
  type: LoadBalancer
status:
  loadBalancer: {}

```

#### Check the end point and svc

```bash
# Ways to access  is using cluster_ip:8080  or loadbalancer:8080 or endpoint:80 or node:8443


root@k8s-ub-master:~# kubectl  get ep,svc
NAME               ENDPOINTS                                            AGE
endpoints/webapp   10.244.106.75:80,10.244.63.200:80,10.244.63.201:80   4m

NAME             TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)         AGE
service/webapp   LoadBalancer   10.106.145.15   192.168.52.16   8080:8443/TCP   4m

```