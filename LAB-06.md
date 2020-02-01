<img src="images/c4logo.png">

### Release Strategy with Deployment
A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

### Deployment provides three features,

  * Availability: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Scalability: Updating the replica count, allows you to scale in and out, and its the responsibility of the deployment to 
    provide with that scalability. You could scale manually or use horizontalPodAutoscaler to do it automatically.
  * Update Strategy: Define a release strategy and update the pods accordingly.
  
 ```
 cd projects/operation/dev/
 cp vote-rs.yaml vote-deploy.yaml
 ```
 file: file: vote-deploy.yaml
 
 ```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  revisionHistoryLimit: 4
  replicas: 12
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

#### To create Deployment execute the below command
```
kubectl apply -f vote-deploy.yaml
```

### Rolling out a new version
Now, update the deployment spec to use a new version of the image.

file: vote-deploy.yaml
```yaml
template:
  metadata:
    labels:
      version: v2   
  spec:
    containers:
      - name: app
        image: c4clouds/vote:v2
```

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status deployment/vote
```

### Breaking a Rollout
file: vote-deploy.yaml

```yaml
spec:
  containers:
    - name: app
      image: c4clouds/vote:fake-version
```

```
kubectl apply -f vote-deploy.yaml

kubectl rollout status
```

### Undo and Rollback
```
kubectl rollout history deploy/vote

kubectl rollout history deploy/vote --revision=xx
```

```
kubectl rollout undo deploy/vote --to-revision=2
```
