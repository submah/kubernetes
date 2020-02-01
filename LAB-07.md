<img src="images/c4logo.png">

### Manage your configuration with ConfigMaps
Configmap is one of the ways to provide configurations to your application

Injecting env variables with configmaps
Create our configmap for vote app

file: projects/operation/dev/vote-cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vote
  namespace: operation
data:
  OPTION_A: Visa
  OPTION_B: Mastercard
```  

To Create ConfigMap Object

```
kubectl get cm
kubectl apply -f vote-cm.yaml
kubectl get cm
kubectl describe cm vote
```

**Note Inorder to use the configmap in your deployment you need to refer the configmap form the file.

>file: vote-deploy.yaml

```yaml
spec:
      containers:
      - image: c4clouds/vote
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

> To apply

```
kubectl apply -f vote-deploy.yaml
```

# Steps to set up NFS based Persistent Volumes
### Creating a Persistent Volume Claim
switch to project directory

cd /projects/operation/dev/

file: db-pvc.yaml

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs
```

create the Persistent Volume Claim and validate

```
kubectl get pvc


kubectl apply -f db-pvc.yaml

kubectl get pvc,pv
```

Now, to use this PVC, db deployment needs to be updated with volume and volumeMounts configs as given in example below

>file: db-deploy-pvc.yaml

```yaml
spec:
   containers:
   - image: postgres:9.4
     imagePullPolicy: Always
     name: db
     ports:
     - containerPort: 5432
       protocol: TCP
     #mount db-vol to postgres data path
     volumeMounts:
     - name: db-vol
       mountPath: /var/lib/postgresql/data
   #create a volume with pvc
   volumes:
   - name: db-vol
     persistentVolumeClaim:
       claimName: db-pvc
```

>Apply db-deploy-pcv.yaml as

```
kubectl apply -f db-deploy-pvc.yaml

kubectl get pod -o wide --selector='role=db'

kubectl get pvc,pv
```
# Set up NFS Provisioner in kubernetes
Change into nfs provisioner installation dir

```
cd k8s-code/storage
kubectl apply -f nfs
```

```
kubectl get storageclass
kubectl get pods
kubectl logs -f nfs-provisioner-0
```
#### Check the output with below command
```
kubectl get pvc,pv
kubectl get pods
```


 

