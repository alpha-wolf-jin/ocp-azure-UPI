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


## Test azure portal login


[Azure Portal](https://portal.azure.com)

![Once Login Azure portal](images/azure-portal-01.png?raw=true "Optional Title")

