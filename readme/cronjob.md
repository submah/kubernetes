<img src="../images/c4logo.png">

In this session we are going to learn:

- Cron 
- Job

### To setup Cron

__File: cron.yml__

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: middle
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: middle
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo `date` Cron is running on every 2 Minutes.
          restartPolicy: OnFailure
```

### To setup job
A job in Kubernetes is a supervisor for pods carrying out batch processes, that is, a process that runs for a certain time to completion, for example a calculation or a backup operation.

Let’s create a job called countdown that supervises a pod counting from 9 down to 1:

__File: job.yml

```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown
spec:
  template:
    metadata:
      name: countdown
    spec:
      containers:
      - name: counter
        image: centos:7
        command:
         - "bin/bash"
         - "-c"
         - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
      restartPolicy: Never
```


