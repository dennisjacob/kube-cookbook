## COOKBOOK 11

### Static pods are not scheduled by the kube-scheduler or managed by the kube-replication-controllers. It is managed by the local kubelet on a node. A mirror pod is created by the kube-api server, though. When a k8s cluster deployed using the kubeadm, the components like kube-apiserver, kube-scheduler, kube-controller-manager and etcd are created as static pods.

#### Find the location of the static pod

```bash
root@k8s-ub-master:/etc/systemd/system/kubelet.service.d# grep '\-\-config' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"

root@k8s-ub-master:/etc/systemd/system/kubelet.service.d# grep -i static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests

root@k8s-ub-master:/etc/systemd/system/kubelet.service.d# cd /etc/kubernetes/manifests
root@k8s-ub-master:/etc/kubernetes/manifests# ls -trl
total 16
-rw------- 1 root root 3231 Apr 25 18:10 kube-controller-manager.yaml
-rw------- 1 root root 3378 Apr 25 18:10 kube-apiserver.yaml
-rw------- 1 root root 1120 Apr 25 18:10 kube-scheduler.yaml
-rw------- 1 root root 1870 Apr 25 18:10 etcd.yaml
root@k8s-ub-master:/etc/kubernetes/manifests#

```

#### Creating a test pod

```bash
root@k8s-ub-master:/etc/kubernetes/manifests# kubectl run static-test-pod --image=jacdenn/rhel7-custom --env STATIC_POD=yes -l static=yes --dry-run=client -oyaml --command -- sleep infinity >static-test-pod.yaml
root@k8s-ub-master:/etc/kubernetes/manifests#

root@k8s-ub-master:/etc/kubernetes/manifests# ls -l static-test-pod.yaml
-rw-r--r-- 1 root root 361 Jun  6 03:59 static-test-pod.yaml


# Check the static pod. If not specified, it creates the pod in default namespace

root@k8s-ub-master:/etc/kubernetes/manifests# kubectl get po -n default
NAME                            READY   STATUS    RESTARTS   AGE
static-test-pod-k8s-ub-master   1/1     Running   0          90s

```
