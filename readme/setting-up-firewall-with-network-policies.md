<img src="../images/c4logo.png">

In this session we are going to learn: 
- What is NetworkPolicy
- Creating default network policy for namespace
- Inter namesapce pods communication with NetworPolicy

### What is NetworkPolicy.
Network policies are Kubernetes resources that control the traffic between pods and/or network endpoints. They uses labels to select pods and specify the traffic that is directed toward those pods using rules. Most CNI plugins support the implementation of network policies, however, if they don't and we create a NetworkPolicy, then that resource will be ignored.

The most popular CNI plugins with network policy support are:

- Weave
- Calico
- Cilium
- Kube-router
- Romana

In Kubernetes, pods are capable of communicating with each other and will accept traffic from any source, by default. With NetworkPolicy we can add traffic restrictions to any number of selected pods, while other pods in the namespace (those that go unselected) will continue to accept traffic from anywhere. The NetworkPolicy resource has mandatory fields such as apiVersion, kind, metadata and spec. Its spec field contains all those settings which define network restrictions within a given namespace:

**podSelector** selects a group of pods for which the policy applies
**policyTypes** defines the type of traffic to be restricted (inbound, outbound, both)
**ingress** includes inbound traffic whitelist rules
**egress** includes outbound traffic whitelist rules

<img src="../images/NetworkPolicy.png">