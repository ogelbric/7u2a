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



