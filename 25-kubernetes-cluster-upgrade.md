

#### Check the current versions

```bash

root@k8s-ub-master:~# kubectl get no -owide
NAME            STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-ub-master   Ready    master   47d   v1.18.2   192.168.52.133   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8
k8s-ub-node1    Ready    <none>   47d   v1.18.2   192.168.52.134   <none>        Ubuntu 20.04 LTS   5.4.0-28-generic   docker://19.3.8
k8s-ub-node2    Ready    <none>   47d   v1.18.2   192.168.52.135   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8

root@k8s-ub-master:~# kubelet --version
Kubernetes v1.18.2

root@k8s-ub-master:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:54:15Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}


root@k8s-ub-master:~# kubectl version --short
Client Version: v1.18.2
Server Version: v1.18.2

```


##### Upgrade the control plane  in k8s-ub-master

```bash

# Check the (next) stable verion 
# You can also run upgrade plan for a particular version , Example : kubeadm upgrade plan v1.18.3

root@k8s-ub-master:~# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.18.2
[upgrade/versions] kubeadm version: v1.18.2
[upgrade/versions] Latest stable version: v1.18.3
[upgrade/versions] Latest stable version: v1.18.3
[upgrade/versions] Latest version in the v1.18 series: v1.18.3
[upgrade/versions] Latest version in the v1.18 series: v1.18.3
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
Kubelet     3 x v1.18.2   v1.18.3
Upgrade to the latest version in the v1.18 series:
COMPONENT            CURRENT   AVAILABLE
API Server           v1.18.2   v1.18.3
Controller Manager   v1.18.2   v1.18.3
Scheduler            v1.18.2   v1.18.3
Kube Proxy           v1.18.2   v1.18.3
CoreDNS              1.6.7     1.6.7
Etcd                 3.4.3     3.4.3-0
You can now apply the upgrade by executing the following command:
        kubeadm upgrade apply v1.18.3
Note: Before you can perform this upgrade, you have to update kubeadm to v1.18.3.




# drain the master node

root@k8s-ub-master:~# kubectl drain k8s-ub-master --ignore-daemonsets
node/k8s-ub-master cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-w5wk6, kube-system/kube-proxy-6j6ll, metallb-system/speaker-zqbbh
evicting pod kube-system/calico-kube-controllers-77bbc67d68-ppwbh
evicting pod kube-system/coredns-66bff467f8-hcfth
evicting pod kube-system/coredns-66bff467f8-p4ws2
pod/coredns-66bff467f8-hcfth evicted
pod/coredns-66bff467f8-p4ws2 evicted
pod/calico-kube-controllers-77bbc67d68-ppwbh evicted
node/k8s-ub-master evicted


# Upgrade the kubeadm binaries to 1.18.3
root@k8s-ub-master:~# apt install kubeadm=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 36 not upgraded.
Need to get 8,163 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.18.3-00 [8,163 kB]
Fetched 8,163 kB in 1s (6,638 kB/s)
(Reading database ... 108014 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.18.3-00_amd64.deb ...
Unpacking kubeadm (1.18.3-00) over (1.18.2-00) ...
Setting up kubeadm (1.18.3-00) ...


# Perform the upgrade on control plane
root@k8s-ub-master:~# kubeadm upgrade apply v1.18.3
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.18.3"
[upgrade/versions] Cluster version: v1.18.2
[upgrade/versions] kubeadm version: v1.18.3
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
[upgrade/prepull] Prepulling image for component etcd.
[upgrade/prepull] Prepulling image for component kube-apiserver.
[upgrade/prepull] Prepulling image for component kube-controller-manager.
[upgrade/prepull] Prepulling image for component kube-scheduler.
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
[apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
[apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
[upgrade/prepull] Prepulled image for component etcd.
[upgrade/prepull] Prepulled image for component kube-scheduler.
[upgrade/prepull] Prepulled image for component kube-controller-manager.
[upgrade/prepull] Prepulled image for component kube-apiserver.
[upgrade/prepull] Successfully prepulled the images for all the control plane components
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.18.3"...
Static pod: kube-apiserver-k8s-ub-master hash: f1eba01cb687200bd7a5d6edb6708264
Static pod: kube-controller-manager-k8s-ub-master hash: 7e7a524bbe609be08615d12902ca2d8c
Static pod: kube-scheduler-k8s-ub-master hash: 155707e0c19147c8dc5e997f089c0ad1
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/etcd] Non fatal issue encountered during upgrade: the desired etcd version for this Kubernetes version "v1.18.3" is "3.4.3-0", but the current etcd version is "3.4.3". Won''t downgrade etcd, instead just continue
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213494313"
W0612 13:05:48.671217   70971 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-06-12-13-05-48/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-apiserver-k8s-ub-master hash: f1eba01cb687200bd7a5d6edb6708264
Static pod: kube-apiserver-k8s-ub-master hash: f1eba01cb687200bd7a5d6edb6708264
Static pod: kube-apiserver-k8s-ub-master hash: f1eba01cb687200bd7a5d6edb6708264
Static pod: kube-apiserver-k8s-ub-master hash: f1eba01cb687200bd7a5d6edb6708264
Static pod: kube-apiserver-k8s-ub-master hash: a012440f420cb855b90ecfb482ea2de6
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-06-12-13-05-48/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-controller-manager-k8s-ub-master hash: 7e7a524bbe609be08615d12902ca2d8c
Static pod: kube-controller-manager-k8s-ub-master hash: eea73215f8b541855dc3cc799dc5a6ce
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-06-12-13-05-48/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-scheduler-k8s-ub-master hash: 155707e0c19147c8dc5e997f089c0ad1
Static pod: kube-scheduler-k8s-ub-master hash: a8caea92c80c24c844216eb1d68fe417
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.18.3". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
'

# Perform upgrade of kubelet and kubectl
root@k8s-ub-master:~# apt install  kubelet=1.18.3-00 kubectl=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  cri-tools
Use 'apt autoremove' to remove it.
The following NEW packages will be installed:
  kubectl kubelet
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 28.3 MB of archives.
After this operation, 157 MB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.18.3-00 [8,821 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.18.3-00 [19.4 MB]
Fetched 28.3 MB in 1s (25.3 MB/s)
Selecting previously unselected package kubectl.
(Reading database ... 108011 files and directories currently installed.)
Preparing to unpack .../kubectl_1.18.3-00_amd64.deb ...
Unpacking kubectl (1.18.3-00) ...
Selecting previously unselected package kubelet.
Preparing to unpack .../kubelet_1.18.3-00_amd64.deb ...
Unpacking kubelet (1.18.3-00) ...
Setting up kubectl (1.18.3-00) ...
Setting up kubelet (1.18.3-00) ...


# Restart the kubelet
root@k8s-ub-master:~# systemctl restart kubelet
root@k8s-ub-master:~#


# Check the versions
root@k8s-ub-master:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:49:29Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
root@k8s-ub-master:~# kubectl version --short
Client Version: v1.18.3
Server Version: v1.18.3
root@k8s-ub-master:~# kubelet --version
Kubernetes v1.18.3
root@k8s-ub-master:~# kubectl get no -owide
NAME            STATUS                     ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-ub-master   Ready,SchedulingDisabled   master   47d   v1.18.3   192.168.52.133   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8
k8s-ub-node1    Ready                      <none>   47d   v1.18.2   192.168.52.134   <none>        Ubuntu 20.04 LTS   5.4.0-28-generic   docker://19.3.8
k8s-ub-node2    Ready                      <none>   47d   v1.18.2   192.168.52.135   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8


# Enable Scheduling on master node

root@k8s-ub-master:~# kubectl uncordon k8s-ub-master
node/k8s-ub-master uncordoned

root@k8s-ub-master:~# kubectl get no -owide
NAME            STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-ub-master   Ready    master   47d   v1.18.3   192.168.52.133   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8
k8s-ub-node1    Ready    <none>   47d   v1.18.2   192.168.52.134   <none>        Ubuntu 20.04 LTS   5.4.0-28-generic   docker://19.3.8
k8s-ub-node2    Ready    <none>   47d   v1.18.2   192.168.52.135   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8



```


##### Upgrade the worker node k8s-ub-node1

```bash
# Drain k8s-ub-node1

root@k8s-ub-master:~# kubectl drain k8s-ub-node1 --ignore-daemonsets
node/k8s-ub-node1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-wwl8p, kube-system/kube-proxy-z95l4, metallb-system/speaker-6glmt
evicting pod kube-system/calico-kube-controllers-77bbc67d68-6kf22
evicting pod metallb-system/controller-57f648cb96-95kq9
evicting pod kube-system/coredns-66bff467f8-4jgll
evicting pod my-ns/newapp-6d988c8f46-nt4nv
evicting pod my-ns/newapp-6d988c8f46-zwf9n
pod/controller-57f648cb96-95kq9 evicted
pod/calico-kube-controllers-77bbc67d68-6kf22 evicted
pod/coredns-66bff467f8-4jgll evicted
pod/newapp-6d988c8f46-nt4nv evicted
pod/newapp-6d988c8f46-zwf9n evicted
node/k8s-ub-node1 evicted


# Login to the worker node and perform the upgrade of components in it

# Install the kubeadm binaries

root@k8s-ub-node1:~# apt install kubeadm=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-5.4.0-26 linux-headers-5.4.0-26-generic linux-image-5.4.0-26-generic linux-modules-5.4.0-26-generic linux-modules-extra-5.4.0-26-generic
Use 'apt autoremove' to remove them.
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 34 not upgraded.
Need to get 8,163 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.18.3-00 [8,163 kB]
Fetched 8,163 kB in 1s (10.7 MB/s)
(Reading database ... 144511 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.18.3-00_amd64.deb ...
Unpacking kubeadm (1.18.3-00) over (1.18.2-00) ...
Setting up kubeadm (1.18.3-00) ...
root@k8s-ub-node1:~#


# Perform the upgrade of worker

root@k8s-ub-node1:~# kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
root@k8s-ub-node1:~#


# Install the kubelet binaries

root@k8s-ub-node1:~# apt install kubelet=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-headers-5.4.0-26 linux-headers-5.4.0-26-generic linux-image-5.4.0-26-generic linux-modules-5.4.0-26-generic linux-modules-extra-5.4.0-26-generic
Use 'apt autoremove' to remove them.
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 33 not upgraded.
Need to get 19.4 MB of archives.
After this operation, 8,192 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.18.3-00 [19.4 MB]
Fetched 19.4 MB in 1s (18.7 MB/s)
(Reading database ... 144511 files and directories currently installed.)
Preparing to unpack .../kubelet_1.18.3-00_amd64.deb ...
Unpacking kubelet (1.18.3-00) over (1.18.2-00) ...
Setting up kubelet (1.18.3-00) ...
root@k8s-ub-node1:~#


# Restart kubelet
root@k8s-ub-node1:~# systemctl restart kubelet
root@k8s-ub-node1:~#

# Check versions
root@k8s-ub-node1:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:49:29Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
root@k8s-ub-node1:~# kubelet --version
Kubernetes v1.18.3


# Uncordon the node 
root@k8s-ub-master:~# kubectl uncordon k8s-ub-node1
node/k8s-ub-node1 uncordoned
root@k8s-ub-master:~# kubectl get no -owide
NAME            STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-ub-master   Ready    master   47d   v1.18.3   192.168.52.133   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8
k8s-ub-node1    Ready    <none>   47d   v1.18.3   192.168.52.134   <none>        Ubuntu 20.04 LTS   5.4.0-28-generic   docker://19.3.8
k8s-ub-node2    Ready    <none>   47d   v1.18.2   192.168.52.135   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8


```


##### Upgrade the worker node k8s-ub-node2

```bash
# Drain k8s-ub-node2

root@k8s-ub-master:~# kubectl drain k8s-ub-node2 --ignore-daemonsets
node/k8s-ub-node2 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-zfpp7, kube-system/kube-proxy-bq5pd, metallb-system/speaker-wvpww
evicting pod kube-system/coredns-66bff467f8-f9276
evicting pod ingress-nginx/nginx-ingress-controller-d7db4d9f7-t97cc
evicting pod my-ns/newapp-6d988c8f46-dn95f
evicting pod metallb-system/controller-57f648cb96-7bv92
evicting pod my-ns/newapp-6d988c8f46-lphcq
evicting pod my-ns/newapp-6d988c8f46-mcxqm
evicting pod my-ns/newapp-6d988c8f46-p45rw
pod/controller-57f648cb96-7bv92 evicted
pod/coredns-66bff467f8-f9276 evicted
I0612 13:24:35.275270  108374 request.go:621] Throttling request took 1.042099628s, request: GET:https://192.168.52.133:6443/api/v1/namespaces/ingress-nginx/pods/nginx-ingress-controller-d7db4d9f7-t97cc
pod/nginx-ingress-controller-d7db4d9f7-t97cc evicted
pod/newapp-6d988c8f46-dn95f evicted
pod/newapp-6d988c8f46-lphcq evicted
pod/newapp-6d988c8f46-mcxqm evicted
pod/newapp-6d988c8f46-p45rw evicted
node/k8s-ub-node2 evicted


# Login to the worker node and perform the upgrade of components in it

# Install the kubeadm binaries

root@k8s-ub-node2:~# apt install kubeadm=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 34 not upgraded.
Need to get 8,163 kB of archives.
After this operation, 0 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.18.3-00 [8,163 kB]
Fetched 8,163 kB in 1s (7,924 kB/s)
(Reading database ... 108174 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.18.3-00_amd64.deb ...
Unpacking kubeadm (1.18.3-00) over (1.18.2-00) ...
Setting up kubeadm (1.18.3-00) ...
root@k8s-ub-node2:~#

# Perform the upgrade of worker

root@k8s-ub-node2:~# kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.


# Install the kubelet binaries

root@k8s-ub-node2:~# apt install kubelet=1.18.3-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 33 not upgraded.
Need to get 19.4 MB of archives.
After this operation, 8,192 B of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.18.3-00 [19.4 MB]
Fetched 19.4 MB in 1s (19.9 MB/s)
(Reading database ... 108174 files and directories currently installed.)
Preparing to unpack .../kubelet_1.18.3-00_amd64.deb ...
Unpacking kubelet (1.18.3-00) over (1.18.2-00) ...
Setting up kubelet (1.18.3-00) ...


# Restart kubelet

root@k8s-ub-node2:~# systemctl restart kubelet
root@k8s-ub-node2:~#


# Check versions

root@k8s-ub-node2:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.3", GitCommit:"2e7996e3e2712684bc73f0dec0200d64eec7fe40", GitTreeState:"clean", BuildDate:"2020-05-20T12:49:29Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
root@k8s-ub-node2:~# kubelet --version
Kubernetes v1.18.3


#Uncordon  the worker

root@k8s-ub-master:~# kubectl uncordon k8s-ub-node2
node/k8s-ub-node2 uncordoned

root@k8s-ub-master:~# kubectl get no -owide
NAME            STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION     CONTAINER-RUNTIME
k8s-ub-master   Ready    master   47d   v1.18.3   192.168.52.133   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8
k8s-ub-node1    Ready    <none>   47d   v1.18.3   192.168.52.134   <none>        Ubuntu 20.04 LTS   5.4.0-28-generic   docker://19.3.8
k8s-ub-node2    Ready    <none>   47d   v1.18.3   192.168.52.135   <none>        Ubuntu 20.04 LTS   5.4.0-26-generic   docker://19.3.8


```