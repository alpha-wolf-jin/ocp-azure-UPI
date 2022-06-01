# ocp-azure-UPI

## Azure environment details
Azure Blank Open Environment
```
Resource Group: openenv
Application: openenv
Application/Client/Service Principal ID: XXXXXXX
Password: XXXXXXXXXXXXXXXXXXX
Tenant ID: XXXXXXXXXXXXXXXXXXXXXXXX
Subscription ID: XXXXXXXXXXXXXXXXX
```

Using this preconfigured service principal for ARO cluster
```
Resource Group: XXXXXXXXXXXXXX
Client ID: XXXXXXXXXXXXXXXXX
Client Secret: XXXXXXXXXXXXXXXXXXX
```

```
In order to conserve resources, we cannot archive or restore any data in this environment. All data will be lost upon expiration.
Here is some important information about your environment:
The resource group 'openenv-dk7fm' was created in our Azure organization.
If you have previously accepted an invitation your environment should be available at https://portal.azure.com
Your Azure account should be SSO enabled, so please log into the portal with your Red Hat Credentials
Your account now has full access to this resource group.  All resources must be in this resource group or you will see permission denied errors.
When this OPEN environment is deleted, the resource group will be removed and all data will be irrecovably removed.
Please regularly back up your data and script your changes in case you need to rebuild.
Please see this page for more information: https://www.opentlc.com/azure/openenv_documentation.html

Your Azure environment details:
Resource Group: openenv-dk7fm
Application: openenv-dk7fm
Application/Client/Service Principal ID: 53 
Password: sNIMx55Ypqavw 
Tenant ID: 1ce7 
Subscription ID: ede7 

Azure CLI quickstart:
export GUID=dk7fm
export CLIENT_ID=5331 
export PASSWORD=sNIMx5 
export TENANT=1ce7 
export SUBSCRIPTION=ede7 
export RESOURCEGROUP=openenv-dk7fm

curl -L https://aka.ms/InstallAzureCli | bash
az login --service-principal -u $CLIENT_ID -p $PASSWORD --tenant $TENANT

See https://docs.microsoft.com/en-us/cli/azure/install-azure-cli for more info on installing the azure CLI
See https://docs.microsoft.com/en-us/cli/azure/ for full documentation of the azure CLI

When creating ARO clusters, you must specify the following credentials in the az aro create command using this preconfigured service principal:
Resource Group: openenv-dk7fm
Client ID: 9b1e8 
Client Secret: 36q 

Example ARO installation commands (log in using az login command from above):
az network vnet create --resource-group openenv-dk7fm --name aro-vnet-dk7fm --address-prefixes 10.0.0.0/22
az network vnet subnet create --resource-group openenv-dk7fm --vnet-name aro-vnet-dk7fm --name master-subnet --address-prefixes 10.0.0.0/23 --service-endpoints Microsoft.ContainerRegistry
az network vnet subnet create --resource-group openenv-dk7fm --vnet-name aro-vnet-dk7fm --name worker-subnet --address-prefixes 10.0.2.0/23 --service-endpoints Microsoft.ContainerRegistry
az network vnet subnet update --name master-subnet --resource-group openenv-dk7fm --vnet-name aro-vnet-dk7fm --disable-private-link-service-network-policies true
az aro create --resource-group openenv-dk7fm --name aro-cluster-dk7fm --vnet aro-vnet-dk7fm --master-subnet master-subnet --worker-subnet worker-subnet --client-id 9b1e8c2f  --client-secret '36qFl4X' --pull-secret @/path/to/pull-secret.txt
```

## Prepare the commands and pull secret

The below commands werer installed.

- oc command
- openshift-install command 
  - `wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz`
- oc command 
  - `wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz`
- pull secet is stored in the /root/pull-secet.txt
- Create defautl dir for cluster installatin
  - `mkdir aro06 ; cd aro06`
- Set Git Credentail cache
  - `git config --global credential.helper 'cache --timeout 7200'`

```
# tar xvf openshift-client-linux.tar.gz
# tar xvf openshift-install-linux.tar.gz
# mv ./{oc,openshift-install} /usr/local/sbin/.
[root@localhost aro08]# oc version
Client Version: 4.10.11
# openshift-install version
openshift-install 4.10.11
```

Please refer to the below link for those tasks:
[OpenShift Pages](https://docs.openshift.com/container-platform/4.10/installing/installing_azure/installing-azure-user-infra.html)

## Test azure cli login

### Add the evironment variable to `~/.bashrc`
```
# vi ~/.bashrc 
...
export GUID=dk7fm
export CLIENT_ID=5331 
export PASSWORD=sNIMx5 
export TENANT=1ce7 
export SUBSCRIPTION=ede7 
export RESOURCEGROUP=openenv-dk7fm

# . ~/.bashrc 

[root@localhost aro06]# az login --service-principal -u $CLIENT_ID -p $PASSWORD --tenant $TENANT
```
> Remove the big file from commit
```
...
remote: error: File openshift-install is 590.87 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/alpha-wolf-jin/ocp-azure-UPI.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/alpha-wolf-jin/ocp-azure-UPI.git'

[root@localhost aro06]# git reset --soft HEAD~1

[root@localhost aro06]# git reset HEAD openshift-install
Unstaged changes after reset:
D	openshift-install

[root@localhost aro06]# git rm --cached openshift-install
rm 'openshift-install'

[root@localhost aro06]# git commit --amend
...
 1 file changed, 10 insertions(+), 2 deletions(-)

[root@localhost aro06]# git push -u origin main
```


## 01 Create public DNS zone from  azure portal


[Azure Portal](https://portal.azure.com)

### Click the resouce group `openenv-g96kt`
![Once Login Azure portal](images/azure-rg-01.png)

### Click `Create` to create the public DNS ZONE
![Create DNS ZONE](images/azure-dns-zone-01.png)


### Specify the RG and DOMAIN name
![Specify Domain](images/azure-dns-zone-02.png)


### New DOMAIN name
![Once Login Azure portal](images/azure-dns-zone-03.png)


### DOMAIN is empty
![Once Login Azure portal](images/azure-dns-zone-04.png)

**CLI creates public DNS zone**
```
az network dns zone create -g ${RESOURCE_GROUP} -n example.opentlc.com
```
### Update the name server on bastion

```
[root@localhost ~]# ping -c 1 ns1-07.azure-dns.com.
PING ns1-07.azure-dns.com (40.90.4.7) 56(84) bytes of data.

[root@localhost ~]# vi /etc/resolv.conf 
# Generated by NetworkManager
nameserver 40.90.4.7
nameserver 8.8.8.8

[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-enp1s0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=enp1s0
DEVICE=enp1s0
ONBOOT=yes
IPADDR=192.168.122.36
PREFIX=24
GATEWAY=192.168.122.1
DNS1=40.90.4.7
DNS2=8.8.8.8

[root@localhost ~]# dig  example.opentlc.com -t NS

; <<>> DiG 9.11.26-RedHat-9.11.26-6.el8 <<>> example.opentlc.com -t NS
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39471
;; flags: qr aa rd; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;example.opentlc.com.		IN	NS

;; ANSWER SECTION:
example.opentlc.com.	172800	IN	NS	ns1-07.azure-dns.com.
example.opentlc.com.	172800	IN	NS	ns2-07.azure-dns.net.
example.opentlc.com.	172800	IN	NS	ns3-07.azure-dns.org.
example.opentlc.com.	172800	IN	NS	ns4-07.azure-dns.info.

;; Query time: 32 msec
;; SERVER: 40.90.4.7#53(40.90.4.7)
;; WHEN: Sat Apr 09 16:27:29 +08 2022
;; MSG SIZE  rcvd: 182
```

## 02 Create Configuration file

**Create OCP configation file**

```
[root@localhost aro06]# openshift-install create install-config
? SSH Public Key /root/.ssh/id_ed25519.pub
? Platform azure
? azure subscription id <Subscription ID>
? azure tenant id <Tenant ID>
? azure service principal client id <preconfigured service principal Client ID>
? azure service principal client secret [? for help] <preconfigured service principal Client Secret>
INFO Saving user credentials to "/root/.azure/osServicePrincipal.json" 
INFO Credentials loaded from file "/root/.azure/osServicePrincipal.json" 
? Region eastus
? Base Domain example.opentlc.com
? Cluster Name aro
? Pull Secret [? for help] 
INFO Install-Config created in: .
```
> if failed, remove previous conf file
`# rm -f /root/.azure/osServicePrincipal.json`

**Customize configation file**

You can customize the configuartion file. Below command can help to understand how to customize it.
`[root@localhost aro06]# openshift-install explain installconfig.platform`

I did not modify the fconfiguartion file.
```
[root@localhost aro06]# cat install-config.yaml
apiVersion: v1
baseDomain: example.opentlc.com
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: {}
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform: {}
  replicas: 3
metadata:
  creationTimestamp: null
  name: aro
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  azure:
    baseDomainResourceGroupName: openenv-g96kt
    cloudName: AzurePublicCloud
    outboundType: Loadbalancer
    region: eastus
publish: External
pullSecret: 
sshKey: |
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIONeWKSpAxbJrkkThCxUjlVe80jSz2y9hIpDLpx43AyY root@localhost.localdomain
```



**Set the env variable based on configuration file**



```
[root@localhost aro06]# yq -r .metadata.name install-config.yaml
aro

[root@localhost aro06]# export CLUSTER_NAME=aro

[root@localhost aro06]# yq -r .platform.azure.region install-config.yaml
eastus

[root@localhost aro06]# export AZURE_REGION=eastus

[root@localhost aro06]# yq -r .baseDomain install-config.yaml
example.opentlc.com

[root@localhost aro06]# export BASE_DOMAIN=example.opentlc.com

[root@localhost aro06]# export BASE_DOMAIN_RESOURCE_GROUP=`yq -r .platform.azure.baseDomainResourceGroupName install-config.yaml`

[root@localhost aro06]# echo $BASE_DOMAIN_RESOURCE_GROUP
openenv-g96kt

```
**backup configuration file**

The original configuration file will be consumed druing creation manifest files.

`[root@localhost aro06]# cp install-config.yaml ~/backup/install-config.yaml-9-April`



## 03 Create manifest files

**Create OCP configation file**

```
[root@localhost aro06]# openshift-install create manifests
INFO Credentials loaded from file "/root/.azure/osServicePrincipal.json" 
INFO Consuming Install Config from target directory 
INFO Manifests created in: manifests and openshift 
[root@localhost aro06]# tree
.
├── images
│   ├── azure-dns-zone-01.png
│   ├── azure-dns-zone-02.png
│   ├── azure-dns-zone-03.png
│   ├── azure-dns-zone-04.png
│   ├── azure-portal-01.png
│   ├── azure-rg-01.png
│   └── test
├── manifests
│   ├── cloud-provider-config.yaml
│   ├── cluster-config.yaml
│   ├── cluster-dns-02-config.yml
│   ├── cluster-infrastructure-02-config.yml
│   ├── cluster-ingress-02-config.yml
│   ├── cluster-network-01-crd.yml
│   ├── cluster-network-02-config.yml
│   ├── cluster-proxy-01-config.yaml
│   ├── cluster-scheduler-02-config.yml
│   ├── cvo-overrides.yaml
│   ├── kube-cloud-config.yaml
│   ├── kube-system-configmap-root-ca.yaml
│   ├── machine-config-server-tls-secret.yaml
│   └── openshift-config-secret-pull-secret.yaml
├── openshift
│   ├── 99_cloud-creds-secret.yaml
│   ├── 99_kubeadmin-password-secret.yaml
│   ├── 99_openshift-cluster-api_master-machines-0.yaml
│   ├── 99_openshift-cluster-api_master-machines-1.yaml
│   ├── 99_openshift-cluster-api_master-machines-2.yaml
│   ├── 99_openshift-cluster-api_master-user-data-secret.yaml
│   ├── 99_openshift-cluster-api_worker-machineset-0.yaml
│   ├── 99_openshift-cluster-api_worker-machineset-1.yaml
│   ├── 99_openshift-cluster-api_worker-machineset-2.yaml
│   ├── 99_openshift-cluster-api_worker-user-data-secret.yaml
│   ├── 99_openshift-machineconfig_99-master-ssh.yaml
│   ├── 99_openshift-machineconfig_99-worker-ssh.yaml
│   ├── 99_role-cloud-creds-secret-reader.yaml
│   └── openshift-install-manifests.yaml
└── README.md

3 directories, 36 files


```

**Update the resoure group**

> `Auto generated route group name <infrastructureName>-rg : aro-pxn45-rg`

> I used predefined resource group, instead of default resource group.

```
# export INFRA_ID=`yq -r '.status.infrastructureName' manifests/cluster-infrastructure-02-config.yml`

# echo $INFRA_ID-rg
aro-pxn45-rg

# sed -i "s/${INFRA_ID}-rg/${BASE_DOMAIN_RESOURCE_GROUP}/g" manifests/*

# sed -i "s/${INFRA_ID}-rg/${BASE_DOMAIN_RESOURCE_GROUP}/g" openshift/*

# grep -R ${INFRA_ID}-rg ./*
# 


```




**Delete machines & machineset configuration files for master and worker**

```
[root@localhost aro06]# ls openshift/
99_cloud-creds-secret.yaml                             99_openshift-cluster-api_worker-machineset-1.yaml
99_kubeadmin-password-secret.yaml                      99_openshift-cluster-api_worker-machineset-2.yaml
99_openshift-cluster-api_master-machines-0.yaml        99_openshift-cluster-api_worker-user-data-secret.yaml
99_openshift-cluster-api_master-machines-1.yaml        99_openshift-machineconfig_99-master-ssh.yaml
99_openshift-cluster-api_master-machines-2.yaml        99_openshift-machineconfig_99-worker-ssh.yaml
99_openshift-cluster-api_master-user-data-secret.yaml  99_role-cloud-creds-secret-reader.yaml
99_openshift-cluster-api_worker-machineset-0.yaml      openshift-install-manifests.yaml

[root@localhost aro06]# rm -f ./openshift/99_openshift-cluster-api_master-machines-*.yaml 
[root@localhost aro06]# rm -f ./openshift/99_openshift-cluster-api_worker-machineset-*.yaml

[root@localhost aro06]# ls ./openshift
99_cloud-creds-secret.yaml                             99_openshift-machineconfig_99-master-ssh.yaml
99_kubeadmin-password-secret.yaml                      99_openshift-machineconfig_99-worker-ssh.yaml
99_openshift-cluster-api_master-user-data-secret.yaml  99_role-cloud-creds-secret-reader.yaml
99_openshift-cluster-api_worker-user-data-secret.yaml  openshift-install-manifests.yaml

```

Confirmed that there is no more `aro-clmlm-rg` in `./manifests/` dir
```
[root@localhost aro06]# grep -R aro-clmlm-rg ./openshift/
[root@localhost aro06]# 

```

**Set env variable based on manifest files**
```
[root@localhost aro06]# export INFRA_ID=`yq -r '.status.infrastructureName' manifests/cluster-infrastructure-02-config.yml`
[root@localhost aro06]# echo $INFRA_ID
aro-clmlm

[root@localhost aro06]# export RESOURCE_GROUP=`yq -r '.status.platformStatus.azure.resourceGroupName' manifests/cluster-infrastructure-02-config.yml`
[root@localhost aro06]# echo $RESOURCE_GROUP
openenv-g96kt

```
## 04 Create Ignite files

```
[root@localhost aro06]# openshift-install create ignition-configs
INFO Consuming Common Manifests from target directory 
INFO Consuming OpenShift Install (Manifests) from target directory 
INFO Consuming Master Machines from target directory 
INFO Consuming Openshift Manifests from target directory 
INFO Consuming Worker Machines from target directory 
INFO Ignition-Configs created in: . and auth      

[root@localhost aro06]# tree
.
├── auth
│   ├── kubeadmin-password
│   └── kubeconfig
├── bootstrap.ign
├── images
│   ├── azure-dns-zone-01.png
│   ├── azure-dns-zone-02.png
│   ├── azure-dns-zone-03.png
│   ├── azure-dns-zone-04.png
│   ├── azure-portal-01.png
│   ├── azure-rg-01.png
│   └── test
├── master.ign
├── metadata.json
├── README.md
└── worker.ign

2 directories, 14 files

```

**Verify the env varaible**

```
[root@localhost aro06]# echo $CLUSTER_NAME
aro

[root@localhost aro06]# echo $AZURE_REGION
eastus

[root@localhost aro06]# echo $AZURE_REGION
eastus

[root@localhost aro06]# echo $BASE_DOMAIN_RESOURCE_GROUP
openenv-g96kt

[root@localhost aro06]# echo $INFRA_ID
aro-clmlm

[root@localhost aro06]# echo $RESOURCE_GROUP
openenv-g96kt

[root@localhost aro06]# export SSH_KEY=`cat ~/.ssh/id_ed25519.pub`
[root@localhost aro06]# echo $SSH_KEY
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIONeWKSpAxbJrkkThCxUjlVe80jSz2y9hIpDLpx43AyY root@localhost.localdomain

```

## 05 Create an Azure identity for predefined the resource group

**Create an Azure identity for the resource group**
```
[root@localhost aro06]# echo az identity create -g $RESOURCE_GROUP -n ${INFRA_ID}-identity
az identity create -g openenv-g96kt -n aro-clmlm-identity

[root@localhost aro06]# az identity create -g $RESOURCE_GROUP -n ${INFRA_ID}-identity

```

**Grant the Contributor role to the Azure identity**
```
[root@localhost aro06]# export PRINCIPAL_ID=`az identity show -g $RESOURCE_GROUP -n ${INFRA_ID}-identity --query principalId --out tsv`

[root@localhost aro06]# export RESOURCE_GROUP_ID=`az group show -g $RESOURCE_GROUP --query id --out tsv`

[root@localhost aro06]# echo az role assignment create --assignee "$PRINCIPAL_ID" --role 'Contributor' --scope "$RESOURCE_GROUP_ID"
az role assignment create --assignee 3cc79d25-f1d5-482f-828c-092671226a6a --role Contributor --scope /subscriptions/ede7f891-835c-4128-af5b-0e53848e54e7/resourceGroups/openenv-g96kt

[root@localhost aro06]# az role assignment create --assignee "$PRINCIPAL_ID" --role 'Contributor' --scope "$RESOURCE_GROUP_ID"

```

**Newly ceated Azure identity on Azure portal**
![Once Login Azure portal](images/azure-rg-identity-01.png)

## 06 Uploading the RHCOS cluster image and bootstrap Ignition config file

**Create an Azure storage account to store the VHD cluster image**

```
[root@localhost aro06]# az storage account create -g $RESOURCE_GROUP --location $AZURE_REGION --name ${CLUSTER_NAME}sa --kind Storage --sku Standard_LRS
```

**Export the storage account key as an environment variable**

```
[root@localhost aro06]# export ACCOUNT_KEY=`az storage account keys list -g $RESOURCE_GROUP --account-name ${CLUSTER_NAME}sa --query "[0].value" -o tsv`

```

**Export the URL of the RHCOS VHD to an environment variable**

```
[root@localhost aro06]# export VHD_URL=$(openshift-install coreos print-stream-json | jq -r '.architectures.x86_64."rhel-coreos-extensions"."azure-disk".url')
[root@localhost aro06]# echo $VHD_URL
https://rhcos.blob.core.windows.net/imagebucket/rhcos-410.84.202201251210-0-azure.x86_64.vhd

```

**Create the storage container for the VHD**

```
[root@localhost aro06]# az storage container create --name vhd --account-name ${CLUSTER_NAME}sa --account-key $ACCOUNT_KEY

```

**Copy the local VHD to a blob**

```
[root@localhost aro06]# az storage blob copy start --account-name ${CLUSTER_NAME}sa --account-key $ACCOUNT_KEY --destination-blob "rhcos.vhd" --destination-container vhd --source-uri "$VHD_URL"

```

**Track copy progressgin..**
Take a few mins
```
status="unknown"
while [ "$status" != "success" ]
do
  status=`az storage blob show --container-name vhd --name "rhcos.vhd" --account-name ${CLUSTER_NAME}sa --account-key $ACCOUNT_KEY -o tsv --query properties.copy.status`
  echo $status
done


pending
pending
pending
success

```
**New Strorage Account on Azure portal**
![Once Login Azure portal](images/azure-storage-container-01.png)


**New vhd container on Azure portal**
![Once Login Azure portal](images/azure-storage-container-02.png)


**Create a blob storage container and upload the generated bootstrap.ign file**

```
[root@localhost aro06]# az storage container create --name files --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY}

[root@localhost aro06]# az storage blob upload --account-name ${CLUSTER_NAME}sa --account-key $ACCOUNT_KEY -c "files" -f "bootstrap.ign" -n "bootstrap.ign"
```


## 07 creating private DNS zones


**Create the private DNS zone**

```
[root@localhost aro06]# az network private-dns zone create -g $RESOURCE_GROUP -n ${CLUSTER_NAME}.${BASE_DOMAIN}

```


**New Privare DNS Zone on Azure portal**
![Once Login Azure portal](images/azure-private-dns-zone-01.png)


## 08 Prepare the ARM templates



```
[root@localhost aro06]# ls -l 0*.json
-rw-r--r--. 1 root root 2849 Apr  9 19:33 01_vnet.json
-rw-r--r--. 1 root root 1168 Apr  9 19:33 02_storage.json
-rw-r--r--. 1 root root 8332 Apr  9 19:33 03_infra.json
-rw-r--r--. 1 root root 6189 Apr  9 19:33 04_bootstrap.json
-rw-r--r--. 1 root root 5760 Apr  9 19:33 05_masters.json
-rw-r--r--. 1 root root 5589 Apr  9 19:33 06_workers.json

```


Please refer to the below link for all templates:
[OpenShift Pages](https://docs.openshift.com/container-platform/4.10/installing/installing_azure/installing-azure-user-infra.html)


## 09 Creating a VNet in Azure

**Create the deployment by using the az CLI**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file "01_vnet.json" --parameters baseName="${INFRA_ID}"

```

**Link the VNet template to the private DNS zone**

```
[root@localhost aro06]# az network private-dns link vnet create -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n ${INFRA_ID}-network-link -v "${INFRA_ID}-vnet" -e false
```



**New VNet & Networ Security Group on Azure portal**
![Once Login Azure portal](images/azure-vnet-nsg-01.png)

**New VNet has 2 subnets on Azure portal**
![Once Login Azure portal](images/azure-upi-network-01.png)
>One subnet is for master nodes and the other is for work nodes. Both are part of machine network.
>Picture was taken in different deployment. The infra ID is differet here.

**New VSecurity Group on Azure portal**
![Once Login Azure portal](images/azure-upi-network-02.png)
>Picture was taken in different deployment. The infra ID is differet here.

## 10 Deploying the RHCOS cluster image for the Azure infrastructure

**Export the RHCOS VHD blob URL as a variable**

```
[root@localhost aro06]# export VHD_BLOB_URL=`az storage blob url --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} -c vhd -n "rhcos.vhd" -o tsv`

```


**Deploy the cluster image**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file 02_storage.json --parameters vhdBlobURL="${VHD_BLOB_URL}" --parameters baseName="${INFRA_ID}"

```
>The blob URL of the RHCOS VHD to be used to create master and worker machines.

**New Image Azure portal**
![Once Login Azure portal](images/azure-image-01.png)


## 11 Creating networking and load balancing components in Azure

**Create the deployment by using the az CLI**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file "03_infra.json" --parameters privateDNSZoneName="${CLUSTER_NAME}.${BASE_DOMAIN}" --parameters baseName="${INFRA_ID}"

```

**Create an api DNS record in the public zone for the API public load balancer. The ${BASE_DOMAIN_RESOURCE_GROUP} variable must point to the resource group where the public DNS zone exists**


Export the following variable

```
[root@localhost aro06]# export PUBLIC_IP=`az network public-ip list -g ${RESOURCE_GROUP} --query "[?name=='${INFRA_ID}-master-pip'] | [0].ipAddress" -o tsv`
[root@localhost aro06]# echo $PUBLIC_IP
52.191.100.197

```

Create the api DNS record in a existing public zone

```
[root@localhost aro06]# az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${BASE_DOMAIN} -n api.${CLUSTER_NAME} -a ${PUBLIC_IP} --ttl 60

```


**New LBs & Public IP Azure portal**
![Once Login Azure portal](images/azure-LBs-PubIP-01.png)


## 12 Creating the bootstrap machine in Azure

**Export the bootstrap URL variable**

```
[root@localhost aro06]# bootstrap_url_expiry=`date -u -d "24 hours" '+%Y-%m-%dT%H:%MZ'`

[root@localhost aro06]# export BOOTSTRAP_URL=`az storage blob generate-sas -c 'files' -n 'bootstrap.ign' --https-only --full-uri --permissions r --expiry $bootstrap_url_expiry --account-name ${CLUSTER_NAME}sa --account-key ${ACCOUNT_KEY} -o tsv`

[root@localhost aro06]# echo $BOOTSTRAP_URL
https://arosa.blob.core.windows.net/files/bootstrap.ign?se=2022-04-10T12%3A04Z&sp=r&spr=https&sv=2018-11-09&sr=b&sig=N68Vy10Y6F3bggvFV3iZml6Sp11m3gnmB2N6OKFSSpg%3D

```

**Export the bootstrap ignition variable**

```
[root@localhost aro06]# export BOOTSTRAP_IGNITION=`jq -rcnM --arg v "3.2.0" --arg url ${BOOTSTRAP_URL} '{ignition:{version:$v,config:{replace:{source:$url}}}}' | base64 | tr -d '\n'`

```

**Create the deployment by using the az CLI**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file "04_bootstrap.json" --parameters bootstrapIgnition="${BOOTSTRAP_IGNITION}" --parameters baseName="${INFRA_ID}" 

```

**Bootstrap VM's resources on Azure portal**
![Once Login Azure portal](images/azure-bootstrap-vm-01.png)


**Click `Connection` for Bootstrap VM's resources on Azure portal**

Click `Connect` in the left panel

![Once Login Azure portal](images/azure-bootstrap-connection-01.png)

**Connect via SSH with client**
![Once Login Azure portal](images/azure-bootstrap-vm-03.png)


```
[root@localhost aro06]# ssh -i ~/.ssh/id_ed25519 core@aro-clmlm-bootstrap-ssh-pip.eastus.cloudapp.azure.com
The authenticity of host 'aro-clmlm-bootstrap-ssh-pip.eastus.cloudapp.azure.com (20.121.83.118)' can't be established.
ECDSA key fingerprint is SHA256:KKzO7WvXCLG+vlDM+DaDDBs+KMt4MmFegkJJ5aLjiN0.
...
---
This is the bootstrap node; it will be destroyed when the master is fully up.

The primary services are release-image.service followed by bootkube.service. To watch their status, run e.g.

  journalctl -b -f -u release-image.service -u bootkube.service

```

## 13 Creating the control plane machines in Azure

**Export the following variable needed by the control plane machine deployment**

```
export MASTER_IGNITION=`cat master.ign | base64 | tr -d '\n'`

```

**Create the deployment by using the az CLI**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file "05_masters.json" --parameters masterIgnition="${MASTER_IGNITION}" --parameters baseName="${INFRA_ID}"

```

```
[root@localhost aro06]# ssh -i ~/.ssh/id_ed25519 core@aro-clmlm-bootstrap-ssh-pip.eastus.cloudapp.azure.com

[core@aro-clmlm-bootstrap ~]$ journalctl -b -f -u release-image.service -u bootkube.service
...
Apr 09 12:43:42 aro-clmlm-bootstrap bootkube.sh[9150]: I0409 12:43:42.019928       1 waitforceo.go:58] cluster-etcd-operator bootstrap etcd
Apr 09 12:43:42 aro-clmlm-bootstrap bootkube.sh[9150]: bootkube.service complete
Apr 09 12:43:42 aro-clmlm-bootstrap systemd[1]: bootkube.service: Succeeded.

```

```
[root@localhost aro06]# openshift-install wait-for bootstrap-complete --dir /root/aro06 --log-level debug
DEBUG OpenShift Installer 4.10.6                   
DEBUG Built from commit 17c2fe7527e96e250e442a15727f7558b2fb8899 
INFO Waiting up to 20m0s (until 9:07PM) for the Kubernetes API at https://api.aro.example.opentlc.com:6443... 
INFO API v1.23.5+b0357ed up                       
INFO Waiting up to 30m0s (until 9:17PM) for bootstrapping to complete... 
DEBUG Bootstrap status: complete                   
INFO It is now safe to remove the bootstrap resources 
DEBUG Time elapsed per stage:                      
DEBUG Bootstrap Complete: 2s                       
DEBUG                API: 1s                       
INFO Time elapsed: 2s         
```

**Delete the bootstrap resources**

```
az network nsg rule delete -g ${RESOURCE_GROUP} --nsg-name ${INFRA_ID}-nsg --name bootstrap_ssh_in
az vm stop -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap
az vm deallocate -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap
az vm delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap --yes
az disk delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap_OSDisk --no-wait --yes
az network nic delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap-nic --no-wait
az storage blob delete --account-key ${ACCOUNT_KEY} --account-name ${CLUSTER_NAME}sa --container-name files --name bootstrap.ign
az network public-ip delete -g ${RESOURCE_GROUP} --name ${INFRA_ID}-bootstrap-ssh-pip
```

## 14 Creating additional worker machines in Azure

**Export the following variable needed by the worker machine deployment**

```
[root@localhost aro06]# export WORKER_IGNITION=`cat worker.ign | base64 | tr -d '\n'`

```

**Create the deployment by using the az CLI**

```
[root@localhost aro06]# az deployment group create -g ${RESOURCE_GROUP} --template-file "06_workers.json" --parameters workerIgnition="${WORKER_IGNITION}" --parameters baseName="${INFRA_ID}"

```

## 15 Approving the certificate signing requests for your machines

**Confirm that the cluster recognizes the machines**

```
[root@localhost aro06]# export KUBECONFIG=/root/aro06/auth/kubeconfig
[root@localhost aro06]# oc get node
NAME                 STATUS   ROLES    AGE   VERSION
aro-clmlm-master-0   Ready    master   27m   v1.23.5+b0357ed
aro-clmlm-master-1   Ready    master   29m   v1.23.5+b0357ed
aro-clmlm-master-2   Ready    master   28m   v1.23.5+b0357ed

```


**Review the pending CSRs**

```
[root@localhost aro06]# oc get csr | grep -i pending
csr-9jh6g                                        3m29s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper         <none>              Pending
csr-gnr47                                        4m25s   kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper         <none>              Pending
csr-sc9lm                                        4m2s    kubernetes.io/kube-apiserver-client-kubelet   system:serviceaccount:openshift-machine-config-operator:node-bootstrapper         <none>              Pending

```


**To approve all pending CSRs, run the following command**

```
[root@localhost aro06]# oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve
certificatesigningrequest.certificates.k8s.io/csr-9jh6g approved
certificatesigningrequest.certificates.k8s.io/csr-gnr47 approved
certificatesigningrequest.certificates.k8s.io/csr-sc9lm approved

```

Wait for a while, other 3 pending CSRs. Each work node has 2 CSR for approval

```
[root@localhost aro06]# oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs --no-run-if-empty oc adm certificate approve
certificatesigningrequest.certificates.k8s.io/csr-9jh6g approved
certificatesigningrequest.certificates.k8s.io/csr-gnr47 approved
certificatesigningrequest.certificates.k8s.io/csr-sc9lm approved

```

Node Status

```
[root@localhost aro06]# oc get node
NAME                        STATUS   ROLES    AGE    VERSION
aro-clmlm-master-0          Ready    master   32m    v1.23.5+b0357ed
aro-clmlm-master-1          Ready    master   33m    v1.23.5+b0357ed
aro-clmlm-master-2          Ready    master   33m    v1.23.5+b0357ed
aro-clmlm-worker-eastus-1   Ready    worker   106s   v1.23.5+b0357ed
aro-clmlm-worker-eastus-2   Ready    worker   111s   v1.23.5+b0357ed
aro-clmlm-worker-eastus-3   Ready    worker   113s   v1.23.5+b0357ed

```

## 16 Adding the Ingress DNS records

**Confirm the Ingress router has created a load balancer and populated the EXTERNAL-IP field**

```
[root@localhost aro06]# oc -n openshift-ingress get service router-default
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
router-default   LoadBalancer   172.30.251.189   20.81.3.162   80:32549/TCP,443:32144/TCP   37m

```


**Export the Ingress router IP as a variable**

```
[root@localhost aro06]# export PUBLIC_IP_ROUTER=`oc -n openshift-ingress get service router-default --no-headers | awk '{print $4}'`
[root@localhost aro06]# echo $PUBLIC_IP_ROUTER
20.81.3.162

```

**Add a *.apps record to the existing public DNS zone**

```
[root@localhost aro06]# az network dns record-set a add-record -g ${BASE_DOMAIN_RESOURCE_GROUP} -z ${BASE_DOMAIN} -n *.apps.${CLUSTER_NAME} -a ${PUBLIC_IP_ROUTER} --ttl 300

```

**Add a *.apps record to the private DNS zone**

Create a *.apps record by using the following command
```
[root@localhost aro06]# echo az network private-dns record-set a create -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps --ttl 300
az network private-dns record-set a create -g openenv-g96kt -z aro.example.opentlc.com -n *.apps --ttl 300
[root@localhost aro06]# echo az network private-dns record-set a add-record -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps -a ${PUBLIC_IP_ROUTER}
az network private-dns record-set a add-record -g openenv-g96kt -z aro.example.opentlc.com -n *.apps -a 20.81.3.162

```

Error

```
[root@localhost aro06]# az network private-dns record-set a create -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps --ttl 300
(PreconditionFailed) The Record set *.apps exists already and hence cannot be created again.
Code: PreconditionFailed
Message: The Record set *.apps exists already and hence cannot be created again.
[root@localhost aro06]# az network private-dns record-set a add-record -g ${RESOURCE_GROUP} -z ${CLUSTER_NAME}.${BASE_DOMAIN} -n *.apps -a ${PUBLIC_IP_ROUTER}
(BadRequest) The list of record sets of type 'A' may not contain multiple entries with the same 'ipv4Address'.
Code: BadRequest
Message: The list of record sets of type 'A' may not contain multiple entries with the same 'ipv4Address'.
```


### *apps is inside private DNS ZONE
Safely to ignore the from adding *.apps record to the private DNS zone
![Once Login Azure portal](images/azure-private-dns-record-01.png)



## 17 Fix image registry storage issue


**Storage Error**
 
It still look for resoure group `aro-clmlm-rg` instead of `openenv-g96kt`

```
[root@localhost aro06]# oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.10.6    True        False         False      13m     
baremetal                                  4.10.6    True        False         False      52m     
cloud-controller-manager                   4.10.6    True        False         False      56m     
cloud-credential                           4.10.6    True        False         False      71m     
cluster-autoscaler                         4.10.6    True        False         False      51m     
config-operator                            4.10.6    True        False         False      55m     
console                                    4.10.6    True        False         False      18m     
csi-snapshot-controller                    4.10.6    True        False         False      44m     
dns                                        4.10.6    True        False         False      44m     
etcd                                       4.10.6    True        False         False      50m     
image-registry                                       False       True          True       44m     Available: The deployment does not exist...
ingress                                    4.10.6    True        False         False      22m     
insights                                   4.10.6    True        False         False      48m     
kube-apiserver                             4.10.6    True        False         False      36m     
kube-controller-manager                    4.10.6    True        False         False      49m     
kube-scheduler                             4.10.6    True        False         False      49m     
kube-storage-version-migrator              4.10.6    True        False         False      44m     
machine-api                                4.10.6    True        False         False      48m     
machine-approver                           4.10.6    True        False         False      54m     
machine-config                             4.10.6    True        False         False      51m     
marketplace                                4.10.6    True        False         False      54m     
monitoring                                 4.10.6    True        False         False      7m26s   
network                                    4.10.6    True        False         False      55m     
node-tuning                                4.10.6    True        False         False      44m     
openshift-apiserver                        4.10.6    True        False         False      44m     
openshift-controller-manager               4.10.6    True        False         False      44m     
openshift-samples                          4.10.6    True        False         False      40m     
operator-lifecycle-manager                 4.10.6    True        False         False      52m     
operator-lifecycle-manager-catalog         4.10.6    True        False         False      52m     
operator-lifecycle-manager-packageserver   4.10.6    True        False         False      44m     
service-ca                                 4.10.6    True        False         False      55m     
storage                                    4.10.6    True        False         False      44m     

[root@localhost aro06]# oc describe co image-registry  
Name:         image-registry
Namespace:    
Labels:       <none>
    ....
    Message:               Progressing: Unable to apply resources: unable to sync storage configuration: failed to start creating storage account: storage.AccountsClient#Create: Failure sending request: StatusCode=0 -- Original Error: Code="AuthorizationFailed" Message="The client 'a2738a72-f94d-45e5-a3df-f6604659ce5a' with object id 'a2738a72-f94d-45e5-a3df-f6604659ce5a' does not have authorization to perform action 'Microsoft.Storage/storageAccounts/write' over scope '/subscriptions/ede7f891-835c-4128-af5b-0e53848e54e7/resourceGroups/aro-clmlm-rg/providers/Microsoft.Storage/storageAccounts/imageregistryaroclmz8sf9' or the scope is invalid. If access was recently granted, please refresh your credentials."
    Reason:                Error
    Status:                True
....
```

**Update the Resource Group for the storage of inernal image registry**

```
[root@localhost aro06]# oc project openshift-image-registry
Now using project "openshift-image-registry" on server "https://api.aro.example.opentlc.com:6443".

[root@localhost aro06]# oc get secret installer-cloud-credentials -o yaml
apiVersion: v1
data:
  azure_client_id: YzI2NDY5MjEtMmEyNy00ZDNlLTk5N2EtNGE3YzhjNDEyZTIy
  azure_client_secret: R1RhaVVhRUVJZGE2cGVtek8wUXljMENwU2ZLUHRzXzIzSA==
  azure_region: ZWFzdHVz
  azure_resource_prefix: YXJvLWNsbWxt
  azure_resourcegroup: YXJvLWNsbWxtLXJn
  azure_subscription_id: ZWRlN2Y4OTEtODM1Yy00MTI4LWFmNWItMGU1Mzg0OGU1NGU3
  azure_tenant_id: MWNlNzg1MmYtZGNmMy00MmJjLWFmZTYtM2JmODFhYjk4NGZi
kind: Secret
metadata:
  annotations:
    cloudcredential.openshift.io/credentials-request: openshift-cloud-credential-operator/openshift-image-registry-azure
  creationTimestamp: "2022-04-09T12:16:09Z"
  name: installer-cloud-credentials
  namespace: openshift-image-registry
  resourceVersion: "2573"
  uid: 6731c8cd-8f5b-4b8f-b178-76e8faefd1a5
type: Opaque
[root@localhost aro06]# echo YXJvLWNsbWxtLXJn | base64 -d
aro-clmlm-rg

[root@localhost aro06]# oc set data secret/installer-cloud-credentials azure_resourcegroup=openenv-g96kt
secret/installer-cloud-credentials data updated


```

**Wait for 10 min..check cluster status**

```
[root@localhost aro06]# oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.10.6    True        False         False      27m     
baremetal                                  4.10.6    True        False         False      65m     
cloud-controller-manager                   4.10.6    True        False         False      69m     
cloud-credential                           4.10.6    True        False         False      85m     
cluster-autoscaler                         4.10.6    True        False         False      65m     
config-operator                            4.10.6    True        False         False      68m     
console                                    4.10.6    True        False         False      32m     
csi-snapshot-controller                    4.10.6    True        False         False      57m     
dns                                        4.10.6    True        False         False      57m     
etcd                                       4.10.6    True        False         False      64m     
image-registry                             4.10.6    True        False         False      10m     
ingress                                    4.10.6    True        False         False      35m     
insights                                   4.10.6    True        False         False      62m     
kube-apiserver                             4.10.6    True        False         False      50m     
kube-controller-manager                    4.10.6    True        False         False      62m     
kube-scheduler                             4.10.6    True        False         False      63m     
kube-storage-version-migrator              4.10.6    True        False         False      57m     
machine-api                                4.10.6    True        False         False      62m     
machine-approver                           4.10.6    True        False         False      68m     
machine-config                             4.10.6    True        False         False      64m     
marketplace                                4.10.6    True        False         False      68m     
monitoring                                 4.10.6    True        False         False      21m     
network                                    4.10.6    True        False         False      69m     
node-tuning                                4.10.6    True        False         False      57m     
openshift-apiserver                        4.10.6    True        False         False      57m     
openshift-controller-manager               4.10.6    True        False         False      57m     
openshift-samples                          4.10.6    True        False         False      54m     
operator-lifecycle-manager                 4.10.6    True        False         False      65m     
operator-lifecycle-manager-catalog         4.10.6    True        False         False      65m     
operator-lifecycle-manager-packageserver   4.10.6    True        False         False      57m     
service-ca                                 4.10.6    True        False         False      68m     
storage                                    4.10.6    True        False         False      57m     

[root@localhost aro06]# oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.10.6    True        False         7m27s   Cluster version is 4.10.6


[root@localhost aro06]# openshift-install  wait-for install-complete 
-bash: ./openshift-install: No such file or directory
[root@localhost aro06]# 
[root@localhost aro06]# openshift-install  wait-for install-complete 
INFO Waiting up to 40m0s (until 10:23PM) for the cluster at https://api.aro.example.opentlc.com:6443 to initialize... 
INFO Waiting up to 10m0s (until 9:53PM) for the openshift-console route to be created... 
INFO Install complete!                            
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/aro06/auth/kubeconfig' 
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.aro.example.opentlc.com 
INFO Login to the console with user: "kubeadmin", and password: "XXXXXXXXXXXXX" 
INFO Time elapsed: 2s                  
```
