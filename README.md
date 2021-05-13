# vCenter update 7.0.2a (7u2a) delivers the VMware Operator. 

## Upgrade your vCenter to 7.0.2a via ISO mount onto vCenter VM.

Download this patch from VMware Customer Connect, you must navigate to Products and Accounts > Product Patches. 
From the Select a Product drop-down menu, select VC and from the Select a Version drop-down menu, select 7.0.2.

Attach the VMware-vCenter-Server-Appliance-7.0.2.00100-17920168-patch-FP.iso file to the vCenter Server CD or DVD drive.
Log in(ssh) to the vCenter appliance as a user with super administrative privileges (for example, root) and run the following commands (don not type shell!):

```
software-packages list
software-packages stage --iso  
software-packages list --staged
software-packages install --staged
```
![Version](https://github.com/ogelbric/7u2a/blob/main/vCenterVersion.png)

## Create content lib for k8 cluster 

Use this link and probably set download when needed (there are 14 images in the lib (5-13-2021)
```
https://wp-content.vmware.com/v2/latest/lib.json
```
![Content lib for k8 guestclusters](https://github.com/ogelbric/7u2a/blob/main/contentlibfork8.png)

## Create content lib for VMware operator

The image is located at the VMware cloud marketplace (login and download)
```
https://marketplace.cloud.vmware.com/services/details/vm-service-image-for-centos1111?slug=true
```
![Content lib for VMware Operator1](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator1.png)

![Content lib for VMware Operator2](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator2.png)

![Content lib for VMware Operator3](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator3.png)

![Content lib for VMware Operator4](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator4.png)

![Content lib for VMware Operator5](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator5.png)

![Content lib for VMware Operator6](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator6.png)

![Content lib for VMware Operator7](https://github.com/ogelbric/7u2a/blob/main/contentlibforvmwareoperator7.png)

## Enable the namespace service (enables yaml ns creation for developers) 

vCenter WCP enabled Cluster -> Configure -> General -> Namespace Service

namespaceservice1

![Enable namespaceservice1](https://github.com/ogelbric/7u2a/blob/main/namespaceservice1.png)

![Enable namespaceservice2](https://github.com/ogelbric/7u2a/blob/main/namespaceservice2.png)

![Enable namespaceservice3](https://github.com/ogelbric/7u2a/blob/main/namespaceservice3.png)

![Enable namespaceservice4](https://github.com/ogelbric/7u2a/blob/main/namespaceservice4.png)

## Create the fist namespace via YAML file 

Log onto supervisor cluster (I am showing the alias I have setup to make the login easier)
```
[root@orfdns 7u2a]# alias l1540
alias l1540='/usr/local/bin/kubectl-vsphere login --vsphere-username administrator@vsphere.local --server=https://192.168.5.40 --insecure-skip-tls-verify'
[root@orfdns 7u2a]# l1540

Password: 
Logged in successfully.

You have access to the following contexts:
   192.168.5.40

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# k apply -f ./namespace1000.yaml 
namespace/namespace1000 created
[root@orfdns 7u2a]# 
```
Here is the namespace YAML file

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: namespace1000  
 ```
 
![namespace result](https://github.com/ogelbric/7u2a/blob/main/namespaceresult.png)

![namespace result from template](https://github.com/ogelbric/7u2a/blob/main/namespaceresultfromteplate.png)

 ## Manual step 
 As of this writting 5-13-2021 the VMservice is not part of the template and hence not part of the YAML and has to be set up by hand
 
 
 
