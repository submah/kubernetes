<img src="../images/c4logo.png">

In this session we are going to learn about:

- Introduction to pod
- Writing pod Specification
- Launching and Operating Pods (Login to the pod, browsing the web UI of the pod)
- Attaching a volume to a Pod
- Launching Multi-Container Pods
- Connecting to Individual Containers
- Launching Replica Set and Fault Tolerance
- Deploying a worker app

### Introduction to pod
While Kubernetes is considered a container orchestration platform, its true building block is the Pod. According to the Kubernetes documentation “Pods are the smallest deployable units of computing that can be created and managed in Kubernetes”.

What exactly is a pod? Essentially, a pod is a layer of encapsulation for a set containers that are typically co-located together from an application perspective. For example, consider a Spring boot application running inside a container that is coupled with a nginx container as a network proxy. When scaling instances of our Spring Boot container, we would also like to scale instances of our nginx containers as well. Pods provide us this functionality.

Kubernetes Pods also utilize useful and interesting methods of container networking. One such method is ensuring that all containers in a pod operate on the same “network stack”. This implementation, conveniently, enables containers in a pod to communicate over localhost.

<img src="../images/Pod_Container_Network.png">

### Writing pod Specification
__file: vote-pod.yml__

```yml
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    app: python
    role: vote
    version: v1
spec:
  containers:
    - name: app
      image: c4clouds/vote:v1
      ports:
        - containerPort: 80
          protocol: TCP
```  

### Launching and Login to the pod
```
kubectl apply -f vote-pod.yml
```
[Output]
pod/vote created


### To view pods
```
kubectl get pods

kubectl get po

kubectl get pods -o wide

kubectl get pods vote
```
### To get detailed info about an Object in kubernetes
```
Syntax : kubectl describe ojbect_name name_of_the_ojbect
kubectl describe pod vote
```
### To view logs of a pod
```
kubectl logs vote
```

### To login to a pod
```
kubectl exec -it vote /bin/sh
```

### browsing the web UI of the pod
In order to browse the UI of the pod we have to create a service for the pod.
Now what is a Service in Kubernetes?

In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them. The set of Pods targeted by a Service is usually determined by a selector

__file: vote-service.yml__
```yml
apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
  type: NodePort
```
To creat a service

```
kubectl apply -f vote-service.yml

```

*Node-IP-Address:30000*

<img src="../images/vote-app-service-output.png">

### Attaching a volume to a Pod
to add a volume to the vote pod, open the vote.yml file and add the below lines.

vi vote.yml

```yml
  #align to name 
  volumeMounts:
    - mountPath: /test-vote-volume
      name: vote-volume
#Align to containers
volumes:
    - name: vote-volume
      hostPath:
        path: /vote-volume
```
To validate 

```
kubectl exec get po -o wide   #check on which node the pod is running

kubectl exec vote -- /bin/sh -c 'df /test-vote-volume'
```
**Note: to check whether container is ableto write data to the mountd volume or not**
```
kubectl exec vote -- /bin/sh -c 'echo "Checking volume from pod" > /test-vote-volume/test.txt'

kubectl exec vote -- /bin/sh -c 'cat /test-vote-volume/test.txt'
```
==================================================================================

Lets create a pod for database and attach a volume to it. To achieve this we will need to

create a volumes definition
attach volume to container using VolumeMounts property

Local host volumes are of two types:
  * emptyDir
  * hostPath

[Click here](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) for Reference document 

File: db-pod.yaml in you cloned repository

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    app: postgres
    role: database
    tier: back
spec:
  containers:
    - name: db
      env:
        - name: POSTGRES_PASSWORD
          value: root
      image: postgres:9.4
      ports:
        - containerPort: 5432
      volumeMounts:
      - name: db-data
        mountPath: /var/lib/postgresql/data
  volumes:
  - name: db-data
    hostPath:
      path: /var/lib/pgdata
      type: DirectoryOrCreate
```
### To create this pod
```
kubectl apply -f db-pod.yaml

kubectl describe pod db

kubectl get events
```

### Creating Multi Container Pods
__File: multi_container_pod.yml__

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    tier: front
    app: nginx
    role: ui
spec:
  containers:
    - name: nginx
      image: nginx:stable-alpine
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - name: data
          mountPath: /var/www/html-sample-app

    - name: sync
      image: c4clouds/sync:v2
      volumeMounts:
        - name: data
          mountPath: /var/www/app

  volumes:
    - name: data
      emptyDir: {}
```

### To Creat this pod

```
kubectl apply -f multi_container_pod.yml
```

### To Login to the pod

```
kubectl exec -it web  sh  -c nginx
kubectl exec -it web  sh  -c sync

```

### Adding Resource requests and limits
We can control the amount of resource requested and also put a limit on the maximum a container in a pod could take up. This can be done by adding to the existing pod spec as below. [Click Here to Refer the official guide](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)

> File: vote-pod.yaml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: vote
  labels:
    app: python
    role: vote
    version: v1
spec:
  containers:
    - name: app
      image: c4clouds/vote:v1
      resources:
        requests:
          memory: "64Mi"
          cpu: "50m"
        limits:
          memory: "128Mi"
          cpu: "250m"
```
### To apply the changes execute the below command
```
kubectl apply -f vote-pod.yaml
```
>[Sample output]
```json
The Pod "vote" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
{"Volumes":[{"Name":"default-token-snbj4","HostPath":null,"EmptyDir":null,"GCEPersistentDisk":null,"AWSElasticBlockStore":null,"GitRepo":null,"Secret":{"SecretName":"default-token-snbj4","Items":null,"DefaultMode":420,"Optional":null},"NFS":null,"ISCSI":null,"Glusterfs":null,"PersistentVolumeClaim":null,"RBD":null,"Quobyte":null,"FlexVolume":null,"Cinder":null,"CephFS":null,"Flocker":null,"DownwardAPI":null,"FC":null,"AzureFile":null,"ConfigMap":null,"VsphereVolume":null,"AzureDisk":null,"PhotonPersistentDisk":null,"Projected":null,"PortworxVolume":null,"ScaleIO":null,"StorageOS":null}],"InitContainers":null,"Containers":[{"Name":"app","Image":"c4clouds/vote:v1","Command":null,"Args":null,"WorkingDir":"","Ports":null,"EnvFrom":null,"Env":null,"Resources":{"Limits":
```

From the above output, its clear that not all the fields are mutable(except for a few e.g labels). Container based deployments primarily follow concept of immutable deployments. So to bring your change into effect, you need to re create the pod as,

```
kubectl delete pod vote

kubectl apply -f vote-pod.yaml

kubectl describe pod vote
```
## Exercise
```
*Define the value of **cpu.request** > **cpu.limit** Try to apply and observe.
* Define the values for **memory.request** and **memory.limit** higher than the total system memory. Apply and observe the deployment and pods.
```

### Launching Replica Set and Fault Tolerance
Lets write the spec for the Rplica Set. This is going to mainly contain,

* replicas
* selector
* template (pod spec )
* minReadySeconds

__file: vote-rs.yml__

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
spec:
  replicas: 2
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3, v4, v5]}
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: c4clouds/vote:v1
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```          

*To apply the changes*

```
kubectl apply -f vote-rs.yaml --dry-run

kubectl apply -f vote-rs.yaml

kubectl get rs

kubectl describe rs vote

kubectl get pods

kubectl get pods --show-labels
```

*Checking High Availability*
Try deleting pods created by the replicaset,

```
kubectl get pods

kubectl delete pods vote-xxxx vote-yyyy
```
*Scalability*

Scaling up application is as easy as running,

```
kubectl scale --replicas=8 rs/vote

kubectl get pods --show-labels
```
### Deploying a worker app
We are going to write deployment spec for deploying the worker app.
Please refer to the [deployment documentation](https://github.com/submah/kubernetes/blob/master/readme/key-concepts-of-kubernetes.md)

__file: worker-deployment.yml__

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: operation
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  revisionHistoryLimit: 4
  paused: false
  replicas: 2
  minReadySeconds: 20
  selector:
    matchLabels:
      role: worker
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
  template:
    metadata:
      name: worker
      labels:
        app: java
        role: worker
        version: v2
    spec:
      containers:
        - name: app
          image: c4clouds/worker:latest
```
*To apply*
```
kubectl apply -f worker-deployment.yml
```
