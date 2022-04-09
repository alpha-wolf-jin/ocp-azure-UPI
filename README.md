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
  `wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz`
- pull secet is stored in the `/root/pull-secet.txt`
- Create defautl dir for cluster installatin `mkdir aro06 ; cd aro06`

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
```
