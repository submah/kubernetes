<img src="images/c4logo.png">

**LAB-01**
=======================================================================================================================================

## Setup Kubernetes Cluster on Centos 7
Prerequisites

- 2048MB RAM + 2 CPU + 20 GB HD (minimum) / node
- Internet access / eth0 interface
- Multiple Linux servers running CentOS 7 (1 Master Node, Multiple Worker Nodes)
- A user account on every system with sudo or root privileges
- The yum package manager, included by default Command-line/terminal window


## Steps for Installing Kubernetes on CentOS 7  [In All Nodes]

To use Kubernetes, you need to install a containerization engine. Currently, the most popular container solution is Docker. 
[Docker needs to be installed on CentOS](https://raw.githubusercontent.com/submah/docker-tutorials/master/docs/install_docker_centos7.md), both on the Master Node and the Worker Nodes.

## Step 1: Configure Kubernetes Repository
Kubernetes packages are not available from official CentOS 7 repositories. This step needs to be performed on the Master Node, and each Worker Node you plan on utilizing for your container setup. Enter the following command to retrieve the Kubernetes repositories.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
## Step 2: Install kubelet, kubeadm, and kubectl [On All Nodes]
These 3 basic packages are required to be able to use Kubernetes. Install the following package(s) on each node:

```
sudo yum install -y kubelet kubeadm kubectl
sudo systemctl enable kubelet
sudo systemctl start kubelet

```

## Step 4: stop  Firewall [On All Nodes]
```
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

## Step 5: Update Iptables Settings

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

```

## Step 6: Disable SELinux

```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#Reboot the nodes
init 6
```
==============================================================================

## Step 1: Create Cluster with kubeadm [On Master Node]

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

## Step 2: Manage Cluster as Regular User [On Master Node]

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Step 3: Set Up Pod Network [On Master Node]

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## Step 4: Check Status of Cluster [On Master Node]
```
sudo kubectl get nodes
```

## Step 5: Join Worker Node to Cluster [On Worker Nodes]
copy the output of [kubeadm init] command and paste, hit enter to execute and join the worker nod to cluster
