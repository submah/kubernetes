<img src="../images/c4logo.png">

In this session we are going to learn about: 

- How to setup NFS on Ubuntu 
- Create Persistent volume (PV)
- Create Persistent Volume Clame (PVC)
- Persistnt volume claim for storing the data


### How to setup NFS on Ubuntu

*Install NFS*
```
apt-get update 

apt-get install nfs-kernel-server -y
```

*Create a directory for sharing over NFS*

```
sudo mkdir -p /mnt/sharedfolder 
sudo chown nobody:nogroup /mnt/sharedfolder 
sudo chmod 777 /mnt/sharedfolder
```
*To export the directoy over NFS*
To export the directoy over NFS open the /etc/exports file and provide the path and the server who want to access the sharedfolder 

```
vi /etc/exports

/mnt/sharedfolder **client_IP_1**(rw,sync,no_subtree_check) 
/mnt/sharedfolder **client_IP_2**(rw,sync,no_subtree_check)

exportfs -a

systemctl restart nfs-kernel-server
```

*Install the NFS Client on the Client Systems*

```
apt update

apt install nfs-common
```

*To verify*
```
showmount -e 192.168.50.53 # IP address fo the NFS server
```

### Create Persistent volume (PV)
Create a persistent volume on Kubernetes mster node

__File: create-persistent-volume.yml__

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 5Gi # GB in Size
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: nfs
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /mnt/sharedfolder #NFS Share Path
    server: 192.168.50.53   #NFS share server IP Address
```

*To apply the spec*

```
kubectl apply -f create-persistent-volume.yml
```

### kubectl apply -f create-persistent-volume.yml

Create a Persistent volumeclaim on masternode

__File: create-persistent-volumeclaim.yml__

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassname: nfs
  resources:
    requests:
      storage: 1Gi #Mention the claim volume size in GB, MB or KB
```      

### Persistnt volume claim for storing the data

create a pod and use the persistnt volume claim for storing the data

__File: persistent-volumeclaim-pod.yml__

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: demo-pvc #persistent volume name
  volumes:
    - name: demo-pvc #Persistent volume name
      persistentVolumeClaim:
        claimName: demo-pvc #Persistent volime claim name
```
*To apply the spec*

```
kubectl apply -f persistent-volumeclaim-pod.yml
```

### To check the PVC

```
kubectl exec -it nginx-pod -- /bin/sh -c "echo 'I hope this is persistent' > /var/www/html/persistent" 

kubectl exec -it nginx-pod -- cat /var/www/html/persistent
```
