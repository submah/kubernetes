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

### To view the network packet routing execute the below command,
```
iptables -nvL -t nat  
iptables -nvL -t nat  | grep 30000
```
Any network trafic that comes to port 30000 get forwarded to the service

```
iptables -nvL -t nat  | grep SERVICE-NAME  -A 5
```

### Accessing Application with ExternalIP
```
kubectl  get svc

kubectl edit svc vote
```
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
  externalIPs:
    - xx.xx.xx.xx
    - yy.yy.yy.yy
```
**Note: replace xx.xx.xx.xx and yy.yy.yy.yy IP addresses of the nodes on two of the kubernetes hosts.

```
kubectl  get svc
kubectl apply -f vote-svc.yaml
kubectl  get svc
kubectl describe svc vote
```

### Internal Service Discovery
To understand Internal Service Discovery browse the vote application and try to click on any one option, is it working? 
To check why the application is not working follow the below steps

```
kubectl get pod
kubectl exec VOTE-POD-NAME nslookup redis
```
To resolve this let's create a service for redis

```
kubectl apply -f redis-svc.yaml

kubectl get svc

kubectl describe svc redis
```

### Creating Endpoints for Redis
Service is been created, but you still need to launch the actual pods running redis application

```
kubectl apply -f redis-deploy.yaml
kubectl describe svc redis
```
