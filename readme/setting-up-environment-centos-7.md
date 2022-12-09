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


## Steps for Installing Kubernetes on CentOS 7

To use Kubernetes, you need to install a containerization engine. Currently, the most popular container solution is Docker. 
[Docker needs to be installed on CentOS](https://raw.githubusercontent.com/submah/docker-tutorials/master/docs/install_docker_centos7.md), both on the Master Node and the Worker Nodes.