## COOKBOOK 15

## Set resource quota on namespace, and set limits on the resources. "Requests" specification is used at the time of pod placement time, where "Limits" specification used at the time of runtime. What is specified in "Requests" is guarenteed to get, but "Limits" are not guarenteed.

#### Create a resource quota on the CPU and memory resources, and number of pods and services

```bash
root@k8s-ub-master:~# cat resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-ns-quota
  namespace: my-ns
spec:
  hard:
    limits.cpu: 4
    limits.memory: "2Gi"
    requests.cpu: "500m"
    requests.memory: "512Mi"
    pods: 100
    services: 50

root@k8s-ub-master:~# kubectl describe resourcequota/my-ns-quota
Name:            my-ns-quota
Namespace:       my-ns
Resource         Used   Hard
--------         ----   ----
limits.cpu       900m   4
limits.memory    512Mi  2Gi
pods             1      100
requests.cpu     300m   500m
requests.memory  256Mi  512Mi
services         0      50

```


#### Create a limit on a particular resource in a namespace.

```bash

# Create the limit range definition for type pod

root@k8s-ub-master:~# cat limit-range.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: pod-limit
  namespace: my-ns
spec:
  limits:
  - type: Pod
    max:
      cpu: 200m
      memory: 256Mi
    min:
      cpu: 100m
      memory: 64Mi


root@k8s-ub-master:~# kubectl apply -f limit-range.yaml
limitrange/pod-limit created


# Describe the limits

root@k8s-ub-master:~# kubectl describe limits
Name:       pod-limit
Namespace:  my-ns
Type        Resource  Min   Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---    ---------------  -------------  -----------------------
Pod         cpu       100m  200m   -                -              -
Pod         memory    64Mi  256Mi  - 

```


