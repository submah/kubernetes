<img src="images/c4logo.png">

**LAB-04**
=======================================================================================================================================

### Creating a Namespace and switching to it
Check current config
```
kubectl config view
```

### Creating a namespace
Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.

Lets create a namespace called **operation**
```
kubectl get ns

kubectl create namespace operation

kubectl get ns
```

### To switch from one context to other use below command
```
kubectl config --help

kubectl config get-contexts

kubectl config current-context

kubectl config set-context --help

kubectl config set-context --current --namespace=operation

kubectl config get-contexts

kubectl config view
```


### Adding ReplicaSet Configurations
Lets now write the spec for the Rplica Set. This is going to mainly contain,

 * replicas
 * selector
 * template (pod spec )
 * minReadySeconds
> edit file: vote-rs.yaml
```yaml
apiVersion: xxx
kind: xxx
metadata:
  xxx
spec:
  xxx
  template:
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
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```              

Above file already containts the spec that you had written for the pod. You would observe its already been added as part of spec.template for replicaset.

Lets now add the details specific to replicaset.

> file: vote-rs.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
spec:
  replicas: 4
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3, v4, v5]}
  template:
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
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```   

To apply the changes

```
kubectl apply -f vote-rs.yaml --dry-run

kubectl apply -f vote-rs.yaml

kubectl get rs

kubectl describe rs vote

kubectl get pods

kubectl get pods --show-labels
```

### Checking High Availability
Try deleting pods created by the replicaset,

```
kubectl get pods

kubectl delete pods vote-xxxx vote-yyyy
```

### Scalability
Scaling up application is as easy as running,
```
kubectl scale --replicas=8 rs/vote

kubectl get pods --show-labels
```
