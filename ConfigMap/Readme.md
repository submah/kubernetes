<img src="../images/c4logo.png">


### `ConfigMaps`

#### Create the local directory
```
mkdir -p configure-pod-container/configmap/
```

### Download the sample files into **configure-pod-container/configmap/** directory
```
wget https://kubernetes.io/examples/configmap/game.properties -O configure-pod-container/configmap/game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O configure-pod-container/configmap/ui.properties
```


### Syntax 
#### kubectl create configmap <configmap-name> --from-file=</path/to/Directory-or-file>

#### Create configmap from Directory
```
kubectl create configmap game-config --from-file=configure-pod-container/configmap/
```

#### To Describe the Configmap
```
kubectl describe configmaps game-config 
```

#### Downlaod the sample file from Internet
```
curl -OL https://k8s.io/examples/pods/config/redis-config
```

#### Create configmap from File
```
kubectl create configmap example-redis-config --from-file=redis-config
```


#### Accessing ConfigMap in a pod
```yml
vi pod-accessing-configmap.yml

apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
   - name: redis
     image: redis
     volumeMounts:
     - mountPath: /redis-master
       name: config
  volumes:
  - name: config
    configMap:
      name: example-redis-config
      items:
      - key: redis-config
        path: redis.conf
```

#### To verify the data i.e. redis.conf is available inside the container or not ?
```
kubectl exec -it redis -- cat /redis-master/redis.conf
```

#### Creat configmap form literal values
```
kubectl create configmap special-config --from-literal=special.how=very
```

#### Accessing configmap in a pod from environment variable 
```yml
vi pod-accessing-confimap-using-env.yml

apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-container
    image: busybox
    command: [ "env" ]
    env:
      - name: SPECIAL_LEVEL_KEY
        valueFrom:
          configMapKeyRef:
             name: special-config
             key: special.how
  restartPolicy: Never

```

#### To verify execute the below command
```
kubectl logs test-pod | grep SPECIAL
```

