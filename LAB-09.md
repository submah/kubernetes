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

