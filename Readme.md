<img src="images/c4logo.png">

**LAB-01**
=======================================================================================================================================

__Master node must be configured with 2 CPU and 2GB of RAM__
### Installing Kubernetes on Ubuntu

#Update 
```
sudo apt-get update

``` 
### Downloading the gpg
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
### Installing Docker and Kubernetes packages
```syntax
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt-get update
sudo apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni 

```
### turning off swap
```
sudo swapoff -a

```

### Initializing Master
```
  sudo kubeadm init --apiserver-advertise-address **PROVIDE IP ADDRESS HERE** --pod-network-cidr=192.168.0.0/16
```
attach initializing-master image

### Initialization of the Nodes
After master being initialized, it should display the command which could be used on all worker/nodes to join the k8s cluster.
```
sudo kubeadm join 10.128.0.2:6443 --token m7qgmb.6gjki057ajpo49nq \
    --discovery-token-ca-cert-hash sha256:fc5d3ec0fe14b141cf87baa206082cf3a26b8c158c02e255f232e1768905b8b7
```
Note: Don't copy and paste the above command, use the output which you have received at the master initialization

### Settingup the Kubectl client configureation On Master node
```color
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### To Validate the Cluster Setup
```
sudo kubectl get nodes
```
add picture "kubectl-get-nodes"

If you see the above image you can find both the worker nodes are in NotReady state. It means Master node is unable to communicat to the worker node(s). In order make the communication we have to install the netwrok driver. we are going to install Weave

### Installing Weave CNI
```
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```
insert picture kubectl-get-nodes-ready

### Set up Visualiser
```
git clone https://github.com/submah/kubernetes.git
kubectl apply -f kubernetes-ops-view/deploy/
```

Visualiser is accessable over a service port whic we can get it for below command

```
kubectl get svc | grep -i kube-ops-view
```

insert picture kube-ops-view-port

```
http://<NODE_IP>:servie-port/#scale=2.0
```

Insert image viasualizer


**LAB-02**
=======================================================================================================================================
### Deploying Application with Kubernetes
> Launch vote application with kubernetes
```
kubectl run vote --image=c4clouds/vote:v1
```

### To view the pod
```
kubectl get pods

kubectl get deployments
```

### Scalability
Scale the vote app to run X number of pods

```
kubectl scale deployment vote --replicas=6

kubectl get pods
```

### Accessing the vote application 
```
kubectl expose deployment vote --type=NodePort --port 80

kubectl get svc
```

### Working with Service Discovery

Once you expose the vote applcation to nodeport you can view the application by navigaing to any NODEIP:SERVICEPORT
After getting the UI if you click on any of the button you will get error message. Inorder to resolve this follow the below command

```
kubectl  create deployment  redis  --image=redis:alpine

kubectl expose deployment redis --port 6379

kubectl  create deployment  worker --image=c4clouds/worker

kubectl  create deployment  db --image=postgres:9.4

kubectl expose deployment db --port 5432

kubectl create deployment  result --image=c4clouds/vote-result

kubectl expose deployment result --type=NodePort --port 80
```

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

