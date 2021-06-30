## Volume Mounts of emptyDir shared across containers in the pod

Example:
```
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: voltest-httpd
        volumeMounts:
        - name: volshare
          mountPath: /mydata-httpd
      - image: jacdenn/rhel7-custom
        name: voltest-custom
        volumeMounts:
        - name: volshare
          mountPath: /mydata-custom
          readOnly: true
      volumes:
      - name: volshare
        emptyDir: {}
```

```bash
kubectl exec -it voltest-d965d965f-8sxml -c voltest-custom -- df -h /mydata-custom
```

## Volume mounts with one container writing and made it available to both

Example:
```
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: voltest-httpd
        volumeMounts:
        - name: volshare
          mountPath: /mydata-httpd
          readOnly: true
      - image: jacdenn/rhel7-custom
        name: voltest-custom
        volumeMounts:
        - name: volshare
          mountPath: /mydata-custom
        command:
        - sh
        args:
        - -c
        - >-
          for each in $(seq 1 50000); do echo "$each $(date)"; sleep 1; done >>/mydata-custom/tail-logs
      volumes:
      - name: volshare
        emptyDir: {}
```
```
kubectl exec -it voltest-7d6b9b8f7c-sxq4p -c voltest-httpd -- tail -f  /mydata-httpd/tail-logs
```


## Create the volume with NFS
```
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: demo-nfs-app
        volumeMounts:
        - name: nfs-mount
          mountPath: /data
      volumes:
      - name: nfs-mount
        nfs:
          server: 192.168.52.131
          path: /nfsfileshare
```

## Create the PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01
  namespace: my-ns
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  nfs:
    path: /nfsfileshare
    server: 192.168.52.131

```

## Create the PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc01
  namespace: my-ns
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow

```

## Use PVC in the Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: vol-demo-nfs-pv
  name: vol-demo-nfs-pv
spec:
  replicas: 2
  selector:
    matchLabels:
      run: vol-demo-nfs-pv
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: vol-demo-nfs-pv
    spec:
      containers:
      - image: jacdenn/rhel7-httpd
        name: vol-demo-nfs-pv
        volumeMounts:
        - name: mydata
          mountPath: /usr/mware
      initContainers:
      - image: jacdenn/rhel7-httpd
        name: vol-demo-nfs-pv-init
        volumeMounts:
        - name: mydata
          mountPath: /usr/mware
        command:
        - sh
        args:
        - -c
        - >-
          echo "$(date) from container-init ">/usr/mware/by-init
      volumes:
      - name: mydata
        persistentVolumeClaim:
          claimName: pvc01

```




## Summary - Quick reference
```
    volumes:
    - emptyDir: {}
...
    volumes:
    - name: nfs-data
      nfs:
         server: 192.168.10.12
         path:  /abc
...
     volumes:
     - name: pvc-share
       persistentVolumeClaim:
           claimName: pvc001    
...
    volumes:
    - name: projected-vol
      projected:
         sources:
         - secret:
               name: credential
         - secret:
               name: salt
```