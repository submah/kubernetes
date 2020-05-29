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

<img src="../images/namespaces.png">

### Pod
While Kubernetes is considered a container orchestration platform, its true building block is the Pod. According to the Kubernetes documentation “Pods are the smallest deployable units of computing that can be created and managed in Kubernetes”.

What exactly is a pod? Essentially, a pod is a layer of encapsulation for a set containers that are typically co-located together from an application perspective. For example, consider a Spring boot application running inside a container that is coupled with a nginx container as a network proxy. When scaling instances of our Spring Boot container, we would also like to scale instances of our nginx containers as well. Pods provide us this functionality.

Kubernetes Pods also utilize useful and interesting methods of container networking. One such method is ensuring that all containers in a pod operate on the same “network stack”. This implementation, conveniently, enables containers in a pod to communicate over localhost.

<img src="../images/Pod_Container_Network.png">

### Replica Sets and Deployments
- What are Replica Sets?
We have already learned that one of the key advantages of Pods is that they allow developers/administrator to group sets of containers as an application unit and easily begin orchestrating them as workloads. Following the creation of a pod’s template, instances of this pod can then be scaled horizontally to make a Developer’s multi container applications more highly available. To manage the scaling of pods, Kubernetes uses an API object called a **ReplicaSet**.

According to the [Kubernetes Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), Replica Sets ensure "that a specified number of pod replicas are running at any given time." As a side note, the documentation extensively focuses on pointing out that ReplicaSets

A ReplicaSet is considered to be a type of Kubernetes API Object. Therefore, just like any other member of this family, ReplicaSets require values for the fields of apiVersion, kind, and metadata to uniquely identify a type of API object across Kubernetes versions.

A ReplicaSet also has an additional field that needs to be set. This is the spec.template, which is a template for a Kubernetes Pod without the apiVersion and kind, which are already present in the ReplicaSet. The Replica Set then uses this template, to create pods which it manages.

<img src="../images/ReplicaSet.png">

