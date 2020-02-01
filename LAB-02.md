<img src="images/c4logo.png">

**LAB-02**
=======================================================================================================================================
### Deploying Application with Kubernetes
> Launch vote application with kubernetes
```
kubectl run vote --image=c4clouds/vote:v1
```

### To view the pod
```
kubectl get pods

kubectl get deployments
```

### Scalability
Scale the vote app to run X number of pods

```
kubectl scale deployment vote --replicas=6

kubectl get pods
```

### Accessing the vote application 
```
kubectl expose deployment vote --type=NodePort --port 80

kubectl get svc
```

### Working with Service Discovery

Once you expose the vote applcation to nodeport you can view the application by navigaing to any NODEIP:SERVICEPORT
After getting the UI if you click on any of the button you will get error message. Inorder to resolve this follow the below command

```
kubectl  create deployment  redis  --image=redis:alpine

kubectl expose deployment redis --port 6379

kubectl  create deployment  worker --image=c4clouds/worker

kubectl  create deployment  db --image=postgres:9.4

kubectl expose deployment db --port 5432

kubectl create deployment  result --image=c4clouds/vote-result

kubectl expose deployment result --type=NodePort --port 80
```

