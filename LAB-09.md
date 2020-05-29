<img src="images/c4logo.png">

### Kubernetes Access Control: Authentication and Authorization

# Role Based Access Control (RBAC)

|Group	|User	|Namespaces	|Resources	|Access Type (verbs)|
|-------|-----|-----------|-----------|-------------------|
|Ops	|Maya	|all	|all	|get, list, watch, update, patch, create, delete, deletecollection
|Dev	|Kim	|operation	|deployments, statefulsets, services, pods, configmaps, secrets, replicasets, ingresses, endpoints, cronjobs, jobs, persistentvolumeclaims	|get, list , watch, update, patch, create


In this lab we are going to do,
  * Create users and groups and setup certs based authentication
  * Create service accounts for applications
  * Create Roles and ClusterRoles to define authorizations
  * Map Roles and ClusterRoles to subjects i.e. users, groups and service accounts using RoleBingings and ClusterRoleBindings.

> Kubernetes API can be accessed by,

  *  Users
  * Service Accounts
  

### Creating Kubernetes Users and Groups
Generate the user's private key

```bash
mkdir -p  ~/.kube/users
cd ~/.kube/users

openssl genrsa -out kim.key 2048
```

Output
```console
root@k8s-master:~/.kube/users# openssl genrsa -out kim.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.................................................................................................................
.......................+++++
.......+++++
e is 65537 (0x010001)
```

Certification Signing Request (CSR) for each of the users. When you generate the csr make sure you also provide
  * CN: This will be set as username
  * O: Org name. This is actually used as a group by kubernetes while authenticating/authorizing users. You could add as many as youneed

example:
```
openssl req -new -key kim.key -out kim.csr -subj "/CN=kim/O=dev/O=example.org"
```
** Note: If you are getting error like Can't load /root/.rnd into RNG plese follow the below setps **
```console
vi /etc/ssl/openssl.cnf

Comment line# 13 
#RANDFILE               = $ENV::HOME/.rnd
```

After generated the CSR file, now you have to signed the certificate by the **Certificate Authority (CA)** which is Kubernetes Master.
  * Certificate : ca.crt (kubeadm) 
  * Pricate Key : ca.key (kubeadm) 

You would typically find it the following paths
** /etc/kubernetes/pki **

Signing the CSR
```
openssl x509 -req -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 730 -in kim.csr -out kim.crt
```

### Setting up User configs with kubectl
In order to configure the users that you created above, following steps need to be performed with kubectl

* Add credentials in the configurations
* Set context to login as a user to a cluster
* Switch context in order to assume the user's identity while working with the cluster

to add credentials,

```
kubectl config set-credentials kim --client-certificate=/root/.kube/users/kim.crt --client-key=/root/.kube/users/kim.key
```

```
kubectl config get-contexts
```

To set context for kubernetes cluster,
```
kubectl config set-context kim-kubernetes --cluster=kubernetes  --user=kim --namespace=operation
```

output
```console
Context "kim-kubernetes" created.
```

You can varify with 
```console
# kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          kim-kubernetes                kubernetes   kim                operation
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
```
and
```yaml
root@k8s-master:~/.kube/users# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.128.0.2:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: operation
    user: kim
  name: kim-kubernetes
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kim
  user:
    client-certificate: users/kim.crt
    client-key: users/kim.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
To switch to different context
```console
kubectl config use-context kim-kubernetes

kubectl config get-contexts

CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kim-kubernetes                kubernetes   kim                operation
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
```
To see the running pod's
```
kubectl get pods
```

output
```console
root@k8s-master:~/.kube/users# kubectl get pod
Error from server (Forbidden): pods is forbidden: User "kim" cannot list resource "pods" in API group "" in the namespace "operation"
```

### Define authorisation rules with Roles and ClusterRoles
Whats the difference between Roles and ClusterRoles ??

* Role is limited to a namespace (Projects/Orgs/Env)
* ClusterRole is Global

### Create a Role and Rolebinding for dev group with the authorizations defined in the table above. Once applied, test it

>Switched to context "kubernetes-admin@kubernetes".
```
kubectl config use-context kubernetes-admin@kubernetes
```
file: kim-role.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  namespace: operation
  name: kim-operation
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch", "update", "patch", "create"]
```

```
kubectl apply -f kim-role.yml
```
file: kim-rolebinding.yml

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kim-operation-rolebinding
  namespace: operation
subjects:
- kind: Group
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: kim-operation
  apiGroup: rbac.authorization.k8s.io
```  

```
kubectl apply -f kim-rolebinding.yml
```


### Exercise
Create a ClusterRole and ClusterRolebinding for ops group with the authorizations defined in the table above. Once applied, test it

### Solution
file: maya-clusterrole.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: maya-clusterrole
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch", "update", "patch", "create"]
```

file: maya-clusterrole-binding.yml

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: maya-cluster-rolebinding
subjects:
- kind: Group
  name: ops
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: maya-clusterrole
  apiGroup: rbac.authorization.k8s.io
```  
