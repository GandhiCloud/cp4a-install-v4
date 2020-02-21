# Additional Pre-req for Transformation Advisor

The prequisite for the Transformation Advisor is given below.

This documentation is already available in the IBM Knowledge Center. https://www.ibm.com/support/knowledgecenter/SSCSJL_4.x/install-prerequisites-ta.html

## NFS Server

## 1. Verify NFS Server setup.

You would need NFS server for Transformation Advisor (ta).

If you already have access to nfs server, please note down following two values.

server ip = x.x.x.x <br/>
path for ta volume = /nfsshare/ta <br/>

*Skip steps below, of you already have nfs server setup*

### 1. Install NFS Server setup.

You would need NFS server for Transformation Advisor (ta). If you have existing NFS server, you can skip this step and goto next step.

If you don't have existing nfs server, you can setup one quickly with following instructions.

1. Run the below commands.

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

2. Export the share by doing below steps.

2.1) Open vi editor

```
vi /etc/exports
```

2.2) Add the below line, save and exit the vi editor.

```
/nfsshare *(rw,sync,no_subtree_check,no_root_squash,no_all_squash)
```

3. Restart nfs-Server by running the below command.

```
systemctl restart rpcbind
systemctl restart nfs-server
```

## 2. Verify Share exists for Transformation Advisor

Lets assume your NFS share is `/nfsshare`.

Check whether you have any share(folder) avaialble for TA like `/nfsshare/ta`  or `/nfsshare/ta_share`.

If exists, note the path of share folder name. (Lets assume it is `/nfsshare/ta`)

If it doesn't exists then do the following to create a folder.
```
mkdir /nfsshare/ta
chmod 755 /nfsshare/ta
```

## 3. Find the NFS Server IP

Note down the IP address of the NFS Server. (Lets assume it is `111.222.333.444`)

## 4. Create PV

We need to create a PV in your cluster by doing below.

Given below the PV Conent.

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
 - ReadWriteMany
 nfs:
   path: <<NFS_SERVER_TA_PATH>>
   server: <<NFS_SERVER_IP>>
 persistentVolumeReclaimPolicy: Recycle
 ```

1. Replace the server ip <<NFS_SERVER_IP>> with the NFS server ip, we noted above. (Our Assumption value `111.222.333.444`)

2. Replace the <<NFS_SERVER_TA_PATH>> with the NFS share folder name. (Our Assumption value `/nfsshare/ta`)

3. Copy the PV content into a file called pv.yaml

4. Run the below command to create the pv in cluster.
```
oc create -f pv.yaml
```
