<img src="../images/c4logo.png">

### Introduction to Kubernetes
In this section we are going to learn about:
* The need for a Container Orchestration Engine
* Battles of COEs, which one to choose
* Key Features of a COE.
* What makes Kubernetes the defacto COE choice.
* Negatives of using Kubernetes

### The need for a Container Orchestration Engine
- What is Container Orchestration?
  Container orchestration is the automation of all aspects of coordinating and managing containers. Container orchestration is focused on managing the life cycle of containers and their dynamic environments.

- Why Do We Need Container Orchestration?
  Container orchestration is used to automate the following tasks at scale:
  
    - [X] **Configuring and scheduling of containers**
    - [X] **Provisioning and deployments of containers**
    - [X] **Availability of containers**
    - [X] **The configuration of applications in terms of the containers that they run in**
    - [X] **Scaling of containers to equally balance application workloads across infrastructure**
    - [X] **Allocation of resources between containers**
    - [X] **Load balancing, traffic routing and service discovery of containers**
    - [X] **Health monitoring of containers**
    - [X] **Securing the interactions between containers.**

### Battles of COEs, which one to choose
Here we are considering Kubernetes. Kubernetes is based on Google’s experience of running workloads at huge scale in production over the past fifteen years. It’s not an open sourcing of Borg, their internal container orchestration system, but draws on lessons Google have learned from running Borg.

Where Docker Swarm extends single host Docker, Kubernetes’ starting point is the cluster itself.

### Key Features of a COE
If you’re considering Kubernetes, now’s the time to think about how willing you are to step away from the Docker way.
Here’s what makes up a Kubernetes cluster:

- **Master:** by default, a single master handles API calls, assigns workloads and maintains configuration state.
- **Minions:** the servers that run workloads and anything else that’s not on the master.
- **Pods:** units of compute power, made-up of one or a handful of containers deployed on the same host, that together perform a task,  have a single IP address and flat networking within the pods.
- **Services:** front end and load balancer for pods, providing a floating IP for access to the pods that power the service, meaning that changes can happen at the pod-level while maintaining a stable interface.
- **Replication controllers:** responsible for maintaining X replicas of the required pods.
- **Labels:** key-value tags (e.g. “Name”) that you and the system use to identify pods, replication controllers and services.

### What makes Kubernetes the defacto COE choice
Benefits of Kubernetes for companies:
- Control and automate deployments and updates
- Save money by optimizing infrastructural resources thanks to the more efficient use of hardware Orchestrate containers on multiple hosts
- Solve many common problems deriving by the proliferation of containers by organizing them in “pods” (see the last post!)
- Scale resources and applications in real time
- Test and autocorrection of applications

### Negatives of using Kubernetes
- Kubernetes can be an overkill for simple applications

- Kubernetes is very complex and can reduce productivity

- The transition to Kubernetes can be cumbersome

- Kubernetes can be more expensive than its alternatives

