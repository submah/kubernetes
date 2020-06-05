<img src="../images/c4logo.png">

In this session we are going to learn about: 

- How to setup NFS on Ubuntu 
- Create Persistent volume (PV)
- Create Persistent Volume Clame (PVC)


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
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /mnt/sharedfolder #NFS Share Path
    server: 192.168.50.53   #NFS share server IP Address
```

*To apply the spec*

```
kubectl apply -f create-persistent-volume.yml
```


