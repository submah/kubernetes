<img src="images/c4logo.png">

### Load Balancing and Service Discovery
In this alb you are going to learn how Loadbalancing work and how service discovery work which is an Internal features offered by Kubernetes.

<img src="images/service-discovery.png">

### Publishing external facing app with NodePort
In Kubernetes you can find 4 types of services which are

  * ClusterIP
  * NodePort
  * LoadBalancer
  * ExternalName
  
  We are going to create a Service with NodePort by which we weill be accessing our vote application
  
  File: vote-svc.yaml
  
  ```yaml
  apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30000
  type: NodePort
  ```
  
  To crea a service with above file use the below command
  
  ```
kubectl apply -f vote-svc.yaml
kubectl get svc
kubectl describe service vote
```

**Note: You can access your vote application with NotePort 3000**
  
