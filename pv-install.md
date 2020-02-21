# Additional Pre-req for Transformation Advisor

## NFS Server setup.
You would need NFS server for Transformation Advisor (ta). <br/>
If you already have access to nfs server, please note down following two values  <br/>
server ip = x.x.x.x <br/>
path for ta volument = /nfsshare/ta <br/>

*Skip steps below, of you already have nfs server setup*

If you don't have existing nfs server, you can setup one quickly with following instructions.

```
yum install -y nfs-utils
mkdir /nfsshare
chmod -R 755 /nfsshare
chown nfsnobody:nfsnobody /nfsshare
systemctl enable rpcbind
systemctl enable nfs-server
systemctl start rpcbind
systemctl start nfs-server

```

Export the share

vi /etc/exports and add following line
```
/nfsshare *(rw,sync,no_subtree_check,no_root_squash,no_all_squash)
```

Restart nfs-Server
```
systemctl restart rpcbind
systemctl restart nfs-server

```
Add share for Transformation Advisor
```
mkdir /nfsshare/ta
chmod 755 /nfsshare/ta

```


As mentioned in knowledgecenter, create a PV.
Easiest way is to create a new pv.yaml file and paste the content.
```
apiVersion: v1
kind: PersistentVolume
metadata:
 name: tanfspv
 labels:
   pvc_for_app: "ta"
spec:
 capacity:
   storage: 8Gi
 accessModes:
 - ReadWriteOnce
 nfs:
   path: /nfsshare/ta
   server: x.x.x.x
 persistentVolumeReclaimPolicy: Recycle

 ```

Run folliwng commands
```
oc create -f pv.yaml

```
