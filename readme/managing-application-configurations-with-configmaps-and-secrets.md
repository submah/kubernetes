<img src="../images/c4logo.png">


In this session we are going to learn about: 

- Introduction to ConfigMaps and Secrets
- Creating Config Map for Vote app
- Setting up Environment Specific Configs
- Adding Configs from Files
- Creating Secrets to Encrypt Database
- Setting Environment vars using Secrets

### Introduction to ConfigMaps and Secrets
*ConfigMaps*

ConfigMaps are Kubernetes objects that can draw configuration information from other sources such as directories or files. ConfigMaps are added to virtual directories called Volumes, which are mounted filesystems that share the lifetime of a Pod which encloses it. This enables containers to access information in ConfigMaps as if it were on a filesystem.

A powerful feature of ConfigMaps, is he ability to aggregate a variety of information from different file types into a config map. Furthermore, if we would like to update any of these files while our Pods are running, we can do so and have the changes be made available in realtime.

Benefits:
- Decouples configuration form pods and components.
- Store configuration data as key-value pairs 
**Note: You must create a ConfigMaps before referencing it in a Pod spec.**

*Creating ConfigMaps from:*
- Directories
- Files
- Literal Values

__file: vote-cm.yaml__

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: operation
data:
  OPTION_A: Visa
  OPTION_B: Mastercard
```
*To Create ConfigMap Object*

```
kubectl get cm
kubectl apply -f vote-cm.yaml
kubectl get cm
kubectl describe cm vote
```

**Note Inorder to use the configmap in your deployment you need to refer the configmap form the file.**

__file: vote-deploy.yaml__

```yml
spec:
      containers:
      - image: c4clouds/vote:v1
        imagePullPolicy: Always
        name: vote
        envFrom:
          - configMapRef:
              name: vote
        ports:
        - containerPort: 80
          protocol: TCP
        restartPolicy: Always
```

*To apply*
```
kubectl apply -f vote-deploy.yaml
```
```yml
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
  paused: false
  replicas: 6
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
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
          image: c4clouds/vote:v4
          #Config Maps goes here
          envFrom:
            - configMapRef:
                name: vote
          resources:
            limits:
              cpu: "200m"
              memory: "250Mi"
            requests: 
              cpu: "100m"
              memory: "50Mi"
          ports:
            - containerPort: 80
              protocol: TCP
```
### Setting up Environment Specific Configs
for setting up environemt specific configs create a new **NameSpace** and modify the files available on kuberetes-lab/voting-app-lab
provide namespace to the spec files.

### Adding Configs from Files

__file: redis.conf__

```r
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
```

*To Apply*
```
kubectl create configmap --from-file redis.conf redis
```