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
- openshift-install command `wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz`
- pull secet is stored in the `/root/pull-secet.txt`
- Create defautl dir for cluster installatin `mkdir aro06 ; cd aro06`

Please refer to the below link for those tasks:
[OpenShift Pages](https://docs.openshift.com/container-platform/4.10/installing/installing_azure/installing-azure-user-infra.html)

## Test azure login

### Add the evironment variable to `~/.bashrc`

[root@localhost aro05]# az login --service-principal -u $CLIENT_ID -p $PASSWORD --tenant $TENANT

