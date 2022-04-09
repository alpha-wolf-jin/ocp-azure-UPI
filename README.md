# ocp-azure-UPI

## Azure environment details

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


## Prepare the commands and pull secret

The below commands werer installed.

- oc command
- openshift-install command 
  - `wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz`
- pull secet is stored in the /root/pull-secet.txt
- Create defautl dir for cluster installatin
  - `mkdir aro06 ; cd aro06`
- Set Git Credentail cache
  - `git config --global credential.helper 'cache --timeout 7200'`


Please refer to the below link for those tasks:
[OpenShift Pages](https://docs.openshift.com/container-platform/4.10/installing/installing_azure/installing-azure-user-infra.html)

## Test azure cli login

### Add the evironment variable to `~/.bashrc`

[root@localhost aro06]# az login --service-principal -u $CLIENT_ID -p $PASSWORD --tenant $TENANT

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
[root@localhost aro06]# ../openshift-install create install-config
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

**Customize configation file**

You can customize the configuartion file. Below command can help to understand how to customize it.
`[root@localhost aro06]# ../openshift-install explain installconfig.platform`

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

[root@localhost aro06]# yq -r .platform.azure.baseDomainResourceGroupName install-config.yaml
openenv-g96kt

[root@localhost aro06]# export BASE_DOMAIN_RESOURCE_GROUP=openenv-g96kt
```
**backup configuration file**

The original configuration file will be consumed druing creation manifest files.

`[root@localhost aro06]# cp install-config.yaml ~/backup/install-config.yaml-9-April`



## 03 Create Configuration file

**Create OCP configation file**

```
[root@localhost aro06]# cp install-config.yaml ~/backup/install-config.yaml-9-April
[root@localhost aro06]# ../openshift-install create manifests
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
