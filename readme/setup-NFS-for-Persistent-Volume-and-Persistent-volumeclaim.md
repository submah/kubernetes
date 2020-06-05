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

