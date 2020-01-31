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

**LAB-02**
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
