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
