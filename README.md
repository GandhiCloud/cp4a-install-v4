# Cloud Pak for Applications V4.0 Installation

Given below the steps to install the Cloud Pak for applications v4.0 on top of the OCP 4.2 version.

This is a simplified version for IBMers to install in FYRE OCP4.2 cluster.

Most of the steps mentioned below to be done from your local system. Switch to infra node, when it is explicitly mentioned.

## References

The detailed steps about this instllation is available in the knowledge center at the location https://www.ibm.com/support/knowledgecenter/SSCSJL_4.x/install-icpa-cli.html

Another version of the installation steps are availbale in https://github.ibm.com/IBMCloudPak4Apps/icpa-install

## Prerequisites

OCP 4.2 cluster up and running. 

## Steps to install

### 1. Get the entitlement key

Get the entitlement key of the cloud pak for application from the below URL for the internal consumption.

https://github.ibm.com/CloudpakOpenContent/cloudpak-entitlement


### 2. Extract the installation configuration from the installer image

1. Replace the <<entitlement_key>> value and run the below export commands to set the entitled registry information.

```
export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=ekey
export ENTITLED_REGISTRY_KEY=<<entitlement_key>>
```

2. Docker login 

Do the docker login to access the entitled registry by running the below command

```
docker login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"

```

3. Create data directory

Create a "data" directory by running the below command

```
mkdir data
```

4. Extract the installation configuration

Extract the installation configuration files by running the below command 

```
docker run -v $PWD/data:/data:z -u 0 \
           -e LICENSE=accept \
           "$ENTITLED_REGISTRY/cp/icpa/icpa-installer:4.0.0" cp -r data/* /data
```
### 3. Update NFS folder permission

1. Goto OCP cluster webconsole.

2. Goto NFS PV.

3. Find the value of the nfs.path attribute. It could be like this.

```
path: /data/nfs1
```

4. Login to the infra node of your cluster with ssh.

```
ssh -v root@111.222.333.444
```


5. Run the below command to upgrade the rights to the nfs folder.

```
chmod -R 777 /data/nfs1
```
6. Exit from the infra node by executing the below command

```
exit
```

### 4. Login to OCP cluster

Login to your OCP cluster using the below command. You need to subsitute the value for <your_cluster_hostname>.

```
oc login https://<your_cluster_hostname> -u <username> -p <password>
```

### 5. Run the installer

Run the installer using the below command. 

```
docker run -v ~/.kube:/root/.kube:z -u 0 -t \
           -v $PWD/data:/installer/data:z \
           -e LICENSE=accept \
           -e ENTITLED_REGISTRY -e ENTITLED_REGISTRY_USER -e ENTITLED_REGISTRY_KEY \
           "$ENTITLED_REGISTRY/cp/icpa/icpa-installer:4.0.0" install
```

It may take more than 20 minutes time to complete the installation. At the end of the installation you will get the necessary URLs to access Kabanero, TA and Tekton.
