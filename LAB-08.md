<img src="images/c4logo.png">

# Application Routing with Ingress Controllers
As part of this implementation you are going to use Traefik as the ingress controller. Its a fast and lightweight ingress controller and also comes with great documentation and support.

There are commonly two ways you could deploy an ingress
 * Using Deployments with HA setup
 * Using DaemonSets which run on every node
 
We pick DaemonSet, which will ensure that one instance of traefik is run on every node. Also, we use a specific configuration hostNetwork so that the pod running traefik attaches to the network of underlying host, and not go through kube-proxy. This would avoid extra network hop and increase performance a bit.

Deploy ingress controller with daemonset as

```shell
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml

```

> To validate
```
kubectl get svc,ds,pods -n kube-system  --selector='k8s-app=traefik-ingress-lb'
```
> To Access 

```
Visit any of the nodes 8080 port e.g. http://IPADDRESS:8080 to see traefik's management UI.
```

### Setting up Named Based Routing for Vote App
We will direct all our request to the ingress controller now, but with differnt hostname e.g. vote.example.com or results.example.com. And it should direct to the correct service based on the host name.

In order to achieve this you, as a user would create a ingress object with a set of rules,

file: vote-ing.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: vote
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
    - host: vote.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: vote
              servicePort: 80
    - host: results.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: results
              servicePort: 80
```

> To Apply
```
kubectl get ing
kubectl apply -f vote-ing.yaml --dry-run
kubectl apply -f vote-ing.yaml
```

```
vote.example.com       -------+                        +----- vote:81
                              |     +-------------+    |
                              |     |   ingress   |    |
                              +===> |   node:80   | ===+
                              |     +-------------+    |
                              |                        |
  results.example.com  -------+                        +----- results:82
```

Create a local hosts file entry. On unix systems its in /etc/hosts file. On windows its at C:\Windows\System32\drivers\etc\hosts. You need admin access to edit this file.

For example, on a linux or osx, you could edit it as,

```
sudo vim /etc/hosts
```

And add an entry such as ,
```
xxx.xxx.xxx.xxx vote.example.com results.example.com
```
