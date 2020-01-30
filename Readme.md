<img src="images/c4logo.png">

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
``` 
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt-get update
sudo apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni 

```
### turning off swap
```
swapoff -a

```

### Initializing Master
```
  sudo kubeadm init --apiserver-advertise-address **PROVIDE IP ADDRESS HERE** --pod-network-cidr=192.168.0.0/16
```
attach initializing-master image

### Initialization of the Nodes
After master being initialized, it should display the command which could be used on all worker/nodes to join the k8s cluster.
```
kubeadm join 10.128.0.2:6443 --token m7qgmb.6gjki057ajpo49nq \
    --discovery-token-ca-cert-hash sha256:fc5d3ec0fe14b141cf87baa206082cf3a26b8c158c02e255f232e1768905b8b7
```
Note: Dont copy and paste the above command, user the out which you have received at the master initialization

