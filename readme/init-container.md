<img src="../images/c4logo.png">

In this session we  are going to learn about:

- What is Init Container
- Create Init Container with Deployent

### What is Init Container
Init Containers are containers that run before the main container runs with your containerized application.

It’s sometimes necessary to prepare a container running in a pod. For example, you might want to wait for a service being available, want to configure things at runtime, or init some data in a database. In all of these cases, init containers are useful. Note that Kubernetes will execute all init containers (and they must all exit successfully) before the main container(s) are executed.

So let’s create an deployment consisting of an init container that writes a message into a file at /ic/this and the main (long-running) container reading out this file, then:


### Create Init Container with Deployent

__File: init-container.yml__

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic
  template:
    metadata:
      labels:
        app: ic
    spec:
      initContainers:
      - name: msginit
        image: centos:7
        command:
        - "bin/bash"
        - "-c"
        - "echo INIT_DONE > /ic/this"
        volumeMounts:
        - mountPath: /ic
          name: msg
      containers:
      - name: main
        image: centos:7
        command:
        - "bin/bash"
        - "-c"
        - "while true; do cat /ic/this; sleep 5; done"
        volumeMounts:
        - mountPath: /ic
          name: msg
      volumes:
      - name: msg
        emptyDir: {}
```
        