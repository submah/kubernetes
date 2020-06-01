<img src="../images/c4logo.png">

In this session we are going to learn about:

- Introduction to pod
- Writing pod Specification
- Launching and Operating Pods (Login to the pod, browsing the web UI of the pod)
- Attaching a volume to a Pod
- Launching Multi-Container Pods
- Connecting to Individual Containers
- Launching Replica Set and Fault Tolerance
- Solution part - Deploying a worker app

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