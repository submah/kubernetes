##################################################################################################
#Kubernetes setup NFS for Persistent Volume and Persistent volumeclaim 
##################################################################################################

#Install NFS on a ubuntu server 
apt-get update
apt install nfs-kernel-server

#Create a directory for sharing over NFS
sudo mkdir -p /mnt/sharedfolder
sudo chown nobody:nogroup /mnt/sharedfolder
sudo chmod 777 /mnt/sharedfolder

#To export the directoy over NFS open the /etc/exports file and provide the path and the server who want to access the sharedfolder
vi /etc/exports

/mnt/sharedfolder 192.168.50.51(rw,sync,no_subtree_check)
/mnt/sharedfolder 192.168.50.52(rw,sync,no_subtree_check)

exportfs -a

#To verify login to the client machine and execute the below command

showmount -e 192.168.50.53  # IP address fo the NFS server


##################################################################################################
#Create a persistent volume on Kubernetes mster node
##################################################################################################
kubectl apply -f create-persistent-volume.yml


##################################################################################################
#Create a Persistent volumeclaim 
##################################################################################################

kubectl apply -f  create-persistent-volumeclaim.yml


####################################################################################################
#create a pod and use the persistnt volume claim for storing the data
####################################################################################################
kubectl apply -f  persistent-volumeclaim-pod.yml



#To check the pvc 
kubectl exec -it nginx-pod -- /bin/sh -c "echo 'I hope this is persistent' > /var/www/html/persistent"
kubectl exec -it nginx-pod -- cat /var/www/html/persistent

