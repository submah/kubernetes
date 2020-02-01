<img src="images/c4logo.png">

**LAB-03**
=======================================================================================================================================
In this Lab we will be learning how to write pod spec with YAML syntax

[Click here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/) to see the Kubernetes API Reference Document

AKMS => Resource Configs Specs

```yaml
apiVersion: 
kind:
metadata:
spec:
```

### Writing Pod Spec
Refer to https://github.com/submah/kubernetes/tree/master/lab


Use the template to create you pod and execute the blow commad to view it.

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

### To Login to a pod
```
kubectl exec -it vote  sh
```

### Adding a Volume for data persistence
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
File: multi_container_pod.yml 

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

To Creat this pod

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
```
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

