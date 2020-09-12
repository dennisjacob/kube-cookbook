
## Create a user with specific roles to do operations on cluster


### Objective
1. Create the user with specific roles
2. Attach the user to a role binding
3. Use certificates for authentication


#### create the private key ( using default options )
```bash
root@k8s-ub-master:~/certs# openssl genrsa -out user1.key
Generating RSA private key, 2048 bit long modulus (2 primes)
..................+++++
........................................................................+++++
e is 65537 (0x010001)
```

#### Create the certificate signing request
```bash
root@k8s-ub-master:~/certs# openssl req -new -key user1.key -out user1.csr -subj="/CN=user1"
```


#### Create the certificate signed by the CA cert used in the k8s cluster
```bash
root@k8s-ub-master:~/certs# openssl x509 -req  -in user1.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -CAserial serial_file -out user1.crt
Signature ok
subject=CN = user1
Getting CA Private Key

root@k8s-ub-master:~/certs# ls -trl
-rw------- 1 root root 1675 Jun  3 12:39 user1.key
-rw-r--r-- 1 root root  887 Jun  3 12:39 user1.csr
-rw-r--r-- 1 root root  989 Jun  3 12:39 user1.crt
-rw-r--r-- 1 root root   41 Jun  3 12:39 serial_file
```

#### Verifying the certs that were created
```bash
root@k8s-ub-master:~/certs# openssl x509 -in user1.crt -subject -dates -issuer -noout
subject=CN = user1
notBefore=Jun  3 12:38:33 2020 GMT
notAfter=Jul  3 12:38:33 2020 GMT
issuer=CN = kubernetes
```

#### Create a role mwdevops with [create get list patch update watch ] access to pod and [create get list patch update watch] access to svc. Check the verbs available using commands below first.

```bash
root@k8s-ub-master:~/certs# kubectl api-resources -owide | grep -wE 'po|svc'
pods                              po                                          true         Pod                              [create delete deletecollection get list patch update watch]
services                          svc                                         true         Service                          [create delete get list patch update watch]

# Test a dry-run
root@k8s-ub-master:~/certs# kubectl create role mwdevops --verb=create,get,list,patch,update,watch --resource=pods,svc --dry-run=client -oyaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: mwdevops
  namespace: my-ns
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - create
  - get
  - list
  - patch
  - update
  - watch


```


#### Bind the role mwdevops to user1 
```bash
# Perform a dry-run
root@k8s-ub-master:~/certs# kubectl create rolebinding mwdevops-rb --role=mwdevops --user=user1 --dry-run=client -oyaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: my-ns
  name: mwdevops-rb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: mwdevops
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: user1
```


#### Validate the role and rolebinding
```bash

root@k8s-ub-master:~/certs# kubectl get role,rolebinding -n my-ns
NAME                                      CREATED AT
role.rbac.authorization.k8s.io/mwdevops   2020-06-03T12:51:51Z

NAME                                                ROLE            AGE
rolebinding.rbac.authorization.k8s.io/mwdevops-rb   Role/mwdevops   20s
```

#### Check the current contexts created on the local
```bash
root@k8s-ub-master:~/certs# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   my-ns
```

#### Add the user to the local kube/config
```bash
root@k8s-ub-master:~/certs# kubectl config set-credentials user1 --client-certificate=/root/certs/user1.crt --client-key=/root/certs/user1.key
User "user1" set.

# Check the access using the auth can-i
root@k8s-ub-master:~/certs# kubectl auth can-i  get svc -n my-ns --as user1
yes
root@k8s-ub-master:~/certs# kubectl auth can-i  get po -n my-ns --as user1
yes
root@k8s-ub-master:~/certs# kubectl auth can-i  get deploy -n my-ns --as user1
no
```

#### Set the contexts
```bash
root@k8s-ub-master:~/certs# kubectl config set-context user1@kubernetes --cluster=kubernetes --user=user1 --namespace my-ns
Context "user1@kubernetes" created.

# Check the contexts again
root@k8s-ub-master:~/certs# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   my-ns
          user1@kubernetes              kubernetes   user1              my-ns
root@k8s-ub-master:~/certs#
```

#### Switch the context now
```bash
root@k8s-ub-master:~/certs# kubectl config use-context user1@kubernetes
Switched to context "user1@kubernetes".
root@k8s-ub-master:~/certs#
```

#### Check the default/current context again
```bash
root@k8s-ub-master:~/certs# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   my-ns
*         user1@kubernetes              kubernetes   user1              my-ns
```

#### Validate the user access
```bash
root@k8s-ub-master:~/certs# kubectl get po,svc
NAME                          READY   STATUS    RESTARTS   AGE
pod/newapp-6d988c8f46-lphcq   1/1     Running   1          32d
pod/newapp-6d988c8f46-mcxqm   1/1     Running   1          32d
pod/newapp-6d988c8f46-nt4nv   1/1     Running   1          32d
pod/newapp-6d988c8f46-zwf9n   1/1     Running   1          32d
root@k8s-ub-master:~/certs# kubectl get deploy
Error from server (Forbidden): deployments.apps is forbidden: User "user1" cannot list resource "deployments" in API group "apps" in the namespace "my-ns"
root@k8s-ub-master:~/certs# kubectl delete po/newapp-6d988c8f46-lphcq
Error from server (Forbidden): pods "newapp-6d988c8f46-lphcq" is forbidden: User "user1" cannot delete resource "pods" in API group "" in the namespace "my-ns"
root@k8s-ub-master:~/certs#
```


