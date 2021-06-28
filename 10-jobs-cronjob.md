## COOKBOOK 10

## Create job and a cornjob


#### Create a job that runs for 3600 sec

```bash
root@k8s-ub-master:~# kubectl create job batch1 --image=jacdenn/rhel7-custom --dry-run=client -oyaml -- sleep 3600

apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: batch1
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - sleep
        - "3600"
        image: jacdenn/rhel7-custom
        name: batch1
        resources: {}
      restartPolicy: Never
status: {}

root@k8s-ub-master:~# kubectl get job
NAME     COMPLETIONS   DURATION   AGE
batch1   0/1           10s        10s

```


#### Create  batch jobs that can run with 3 replicas that runs for 360 sec

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: batch2
spec:
  parallelism: 3
  template:
    metadata:
    spec:
      containers:
      - command:
        - sleep
        - "3600"
        image: jacdenn/rhel7-custom
        name: batch2
      restartPolicy: Never

```

#### Create  batch jobs that can run with 4 replicas that runs for 30 sec, and with a backofflimit if it fails.

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: batch3
spec:
  parallelism: 2
  backoffLimit: 2
  template:
    metadata:
    spec:
      containers:
      - command:
        - sleep
        - "36i00"
        image: jacdenn/rhel7-custom
        name: batch2
      restartPolicy: Never

```


#### Create a cron job

```bash

# cron job that runs every 3 hours at 15th/25th and 35th minute

kubectl create cronjob cronbatch --image=jacdenn/rhel7-custom --schedule="15,25,55 */3 * * *" --dry-run=client -oyaml  -- sleep 300

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronbatch
spec:
  jobTemplate:
    metadata:
      name: cronbatch
    spec:
      template:
        metadata:
        spec:
          containers:
          - command:
            - sleep
            - "300"
            image: jacdenn/rhel7-custom
            name: cronbatch
            resources: {}
          restartPolicy: OnFailure
  schedule: 15,25,55 */3 * * *


# Check ths cronjobs
root@k8s-ub-master:~# kubectl get cj -owide
NAME        SCHEDULE             SUSPEND   ACTIVE   LAST SCHEDULE   AGE   CONTAINERS   IMAGES                 SELECTOR
cronbatch   15,25,55 */3 * * *   False     0        <none>          66s   cronbatch    jacdenn/rhel7-custom   <none>


```

```
# Cron format
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * <command to execute>
```

#### Suspend a cronjob using a patch

```bash
# create a patch file
root@k8s-ub-master:~# cat patch.yaml
spec:
  suspend: true

# Apply the patch
root@k8s-ub-master:~# kubectl patch cj/cronbatch --patch "$(cat patch.yaml)"
cronjob.batch/cronbatch patched


# Validate the change
rueroot@k8s-ub-master:~# kubectl get cj/cronbatch -o=jsonpath='{.spec.suspend}{"\n"}'
true


```