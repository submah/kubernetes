<img src="../images/c4logo.png">

### Key Concepts of Kubernetes
In this lession we are going to learn about:
- Namespaces
- Pods
- Replica Sets and Deployments
- Service Discovery and Load Balancing
- Configmaps, Storage, Network, RBAC
- Statefulsets, Crons and Jobs
- Kubernetes Architecture

### Namespaces
Namespaces are Kubernetes objects which partition a single Kubernetes cluster into multiple virtual clusters. Each Kubernetes namespace provides the scope for Kubernetes Names it contains; which means that using the combination of an object name and a Namespace, each object gets an unique identity across the cluster.

By default, a Kubernetes cluster is created with the following three namespaces:

- default: By default all the resource created in Kubernetes cluster are created in the default namespace. By default the default namespace can allow applications to run with unbounded CPU and memory requests/limits (Until someone set resource quota for the default namespace).

- kube-public: Namespace for resources that are publicly readable by all users. This namespace is generally reserved for cluster usage.

- kube-system: It is the Namespace for objects created by Kubernetes systems/control plane.