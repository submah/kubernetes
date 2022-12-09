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

## Step 5: Update Iptables Settings [On All Nodes]

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

```

## Step 6: Disable SELinux [On All Nodes]

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
<img src="../images/get-nodes.png">

## Step 4: Check Status of Cluster [On Master Node]
```
sudo kubectl get nodes
```

## Step 5: Join Worker Node to Cluster [On Worker Nodes]
After master being initialized, it should display the command which could be used on all worker/nodes to join the k8s cluster.
```
sudo kubeadm join 10.128.0.2:6443 --token m7qgmb.6gjki057ajpo49nq \
    --discovery-token-ca-cert-hash sha256:fc5d3ec0fe14b141cf87baa206082cf3a26b8c158c02e255f232e1768905b8b7
```
Note: Don't copy and paste the above command, use the output which you have received at the master initialization [kubeadm init]

### Launching Kubernetes Dashboard
```yml
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

#Open the file and update the service to
ports:
  - nodePort: 32414
    port: 443
    targetPort: 8443
type: NodePort

#To apply 
kubectl apply -f recommended.yaml
```

### Creating a Service Account
We are creating Service Account with name admin-user in namespace kubernetes-dashboard first.
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

### Creating a ClusterRoleBinding
the ClusterRole cluster-admin already exists in the cluster. We can use it and create only ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.
```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

In order to access the Kubernetes dashboard https://Node-IP-Address:32414 

[output]

<img src="../images/kubernetes-dashboard-token.png">

__Getting a Bearer Token__
```yml
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
#Copy and paste the token on the dashboard browser window
```
```
kubectl -n kubernetes-dashboard create token admin-user
```
<img src="../images/kubernetes-dashboard.png">


### Set up Visualiser
```
git clone https://github.com/submah/kubernetes.git
kubectl apply -f kubernetes/kubernetes-ops-view/deploy
```
[Output]
```
serviceaccount/kube-ops-view created
clusterrole.rbac.authorization.k8s.io/kube-ops-view created
clusterrolebinding.rbac.authorization.k8s.io/kube-ops-view created
deployment.apps/kube-ops-view created
ingress.extensions/kube-ops-view created
deployment.apps/kube-ops-view-redis created
service/kube-ops-view-redis created
service/kube-ops-view created
```

Visualiser is accessable over a service port whic we can get it for below command

```
kubectl get svc | grep -i kube-ops-view
```


```
http://<NODE_IP>:servie-port/#scale=2.0
```

<img src="../images/viasualizer.PNG">


