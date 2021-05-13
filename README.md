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
 
 ![vmservice 1](https://github.com/ogelbric/7u2a/blob/main/vmservice1.png)
 
 ![vmservice 2](https://github.com/ogelbric/7u2a/blob/main/vmservice2.png)
 
 ![vmservice 3](https://github.com/ogelbric/7u2a/blob/main/vmservice3.png)
 
 ![vmservice 4](https://github.com/ogelbric/7u2a/blob/main/vmservice4.png)
 
 ![vmservice 5](https://github.com/ogelbric/7u2a/blob/main/vmservice5.png)
 
 ![vmservice 6](https://github.com/ogelbric/7u2a/blob/main/vmservice6.png)
 
## Create guest cluster
```
[root@orfdns 7u2a]# k apply -f ./tkg-cluster-berlin.yaml 
tanzukubernetescluster.run.tanzu.vmware.com/tkg-berlin created
[root@orfdns 7u2a]# 
```
Here is the YAMl file
```
apiVersion: run.tanzu.vmware.com/v1alpha1
kind: TanzuKubernetesCluster
metadata:
  name: tkg-berlin
  namespace: namespace1000
spec:
  distribution:
    version: v1.18.5
    fullVersion:
  topology:
    controlPlane:
      count: 1
      class: best-effort-small
      storageClass: pacific-gold-storage-policy
    workers:
      count: 3
      class: best-effort-small
      storageClass: pacific-gold-storage-policy
  settings:
    network:
      cni:
        name: calico
      services:
        cidrBlocks: ["198.51.100.0/12"]
      pods:
        cidrBlocks: ["192.0.2.0/16"]
```

Check on the cluster
```
[root@orfdns 7u2a]# l1540

Password: 
Logged in successfully.

You have access to the following contexts:
   192.168.5.40
   namespace1000

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
[root@orfdns 7u2a]# k1
Switched to context "namespace1000".
[root@orfdns 7u2a]# k get tkc
NAME         CONTROL PLANE   WORKER   DISTRIBUTION                     AGE     PHASE      TKR COMPATIBLE   UPDATES AVAILABLE
tkg-berlin   1               3        v1.18.5+vmware.1-tkg.1.c40d30d   4m18s   creating   True             [1.19.7+vmware.1-tkg.1.fc82c41 1.18.15+vmware.1-tkg.1.600e412]
[root@orfdns 7u2a]# 
```
 ![guest cluster](https://github.com/ogelbric/7u2a/blob/main/guestcluster.png)


## Generate a ssh key on your server (needed for cloud-init.yaml file for VM to log into)
```
[root@orfdns .ssh]# ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:HZq+oegejQHIcA3ck2t4GGUOGlvfoNH6N1AF0MGBtmc root@orfdns
The key's randomart image is:
+---[RSA 4096]----+
|o.**=*+=.        |
|oBoBOo+          |
|+.o*o=.   .      |
|  +.* E  + .     |
|   +.+  S .      |
|    .+o.         |
|    o...o        |
|     o . o       |
|   o+ . .        |
+----[SHA256]-----+
```
## Take the pub key and place into cloud-init.yaml file 
```
[root@orfdns 7u2a]# cat ../.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCjr5LYVTa63c4XMsh8O680QZaj9ZPQ7RySbnH81MbqJKf2gfmwVHqoPL0gEB3ypKj1otDy5SVrNZBsw1KvnNM9oU5fB//aWSrGLQvCCTWsyqMhGlMCQBPwtnFyAOZSClf+wVzpEPhf5W8V6g8OY0m/rh4Ukr03GqX7wCuhFk4/X+Ujx7NBUnvbZV6MnDCBj0DRAfeBaZctJXTrIGeRcUJ4Rsl+4TN+N6noQ7Z2ewnBZY8IyzJTNaIHCddWJeJGn0WaiFoalERijRSzlgtKbiLtC/Jd2/+SrJIRUPq8ChrKlF24bt1RyOIqeGRXB/J8aU3AJDr9BPe5GjfcMYY/cb1m1m4S0y5uUi3ob4pRYl/q6eo8M1YIjEHK1IBA7iGPkBqV71sOVP3T/6oK0ZppxqCwADMbfwAr9SCMVun5PpnI9MOVjEwj7pHMdJJwmFkc1QK3lE2yBT5o2iPup8DozRLENzDeXnUhvMJW6POJIkoexLN6TU+LDY/AMd40k3GwJ+lVQdehAedsnYN/64+lAIICjYupgK0tG25C+YStd14yAM/aTFcZ9MXyDzbs433f/OSlgh/+hUyVN5Z26obrg6UXyXN6FCaed4qv/hCjM7tiDdHcmbQlhH3WhpBPlJJquIyiq8wKmwkKJuNWxSqQ9tpoLpgEnA55ucPIfXzewuKb6Q== root@orfdns
[root@orfdns 7u2a]# 
```
Here is the cloud-init.yaml file 
```
#cloud-config
chpasswd:
    list: |
      centos:VMware1!
    expire: false
ssh_pwauth: True
groups:
  - docker
users:
  - default
  - name: centos
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCjr5LYVTa63c4XMsh8O680QZaj9ZPQ7RySbnH81MbqJKf2gfmwVHqoPL0gEB3ypKj1otDy5SVrNZBsw1KvnNM9oU5fB//aWSrGLQvCCTWsyqMhGlMCQBPwtnFyAOZSClf+wVzpEPhf5W8V6g8OY0m/rh4Ukr03GqX7wCuhFk4/X+Ujx7NBUnvbZV6MnDCBj0DRAfeBaZctJXTrIGeRcUJ4Rsl+4TN+N6noQ7Z2ewnBZY8IyzJTNaIHCddWJeJGn0WaiFoalERijRSzlgtKbiLtC/Jd2/+SrJIRUPq8ChrKlF24bt1RyOIqeGRXB/J8aU3AJDr9BPe5GjfcMYY/cb1m1m4S0y5uUi3ob4pRYl/q6eo8M1YIjEHK1IBA7iGPkBqV71sOVP3T/6oK0ZppxqCwADMbfwAr9SCMVun5PpnI9MOVjEwj7pHMdJJwmFkc1QK3lE2yBT5o2iPup8DozRLENzDeXnUhvMJW6POJIkoexLN6TU+LDY/AMd40k3GwJ+lVQdehAedsnYN/64+lAIICjYupgK0tG25C+YStd14yAM/aTFcZ9MXyDzbs433f/OSlgh/+hUyVN5Z26obrg6UXyXN6FCaed4qv/hCjM7tiDdHcmbQlhH3WhpBPlJJquIyiq8wKmwkKJuNWxSqQ9tpoLpgEnA55ucPIfXzewuKb6Q== root@orfdns
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo, docker
    shell: /bin/bash
network:
  version: 2
  ethernets:
      ens192:
          dhcp4: true
package_update: true
packages:
  - mysql-server
  - net-tools
runcmd:
  - systemctl enable mysqld
  - systemctl start mysqld
  - sudo mysql -e "CREATE DATABASE wordpress;"
  - sudo mysql -e "CREATE USER 'wordpress_user'@'%' IDENTIFIED BY 'password';"
  - sudo mysql -e "GRANT ALL ON wordpress.* TO 'wordpress_user'@'%'"
  - sudo mysql -e "FLUSH PRIVILEGES;"
  - sed -i '$abind-address=0.0.0.0' /etc/my.cnf.d/mysql-server.cnf
  - systemctl restart mysqld
  - firewall-offline-cmd --add-port=3306/tcp
  - firewall-cmd --reload
  - systemctl restart sshd
```
## Now convert the cloud-init.yaml file to base 64
```
cat cloud-init.yaml | base64 -w0
I2Nsb3VkLWNvbmZpZwpjaHBhc3N3ZDoKICAgIGxpc3Q6IHwKICAgICAgY2VudG9zOlZNd2FyZTEhCiAgICBleHBpcmU6IGZhbHNlCnNzaF9wd2F1dGg6IFRydWUKZ3JvdXBzOgogIC0gZG9ja2VyCnVzZXJzOgogIC0gZGVmYXVsdAogIC0gbmFtZTogY2VudG9zCiAgICBzc2gtYXV0aG9yaXplZC1rZXlzOgogICAgICAtIHNzaC1yc2EgQUFBQUIzTnphQzF5YzJFQUFBQURBUUFCQUFBQ0FRQ2pyNUxZVlRhNjNjNFhNc2g4TzY4MFFaYWo5WlBRN1J5U2JuSDgxTWJxSktmMmdmbXdWSHFvUEwwZ0VCM3lwS2oxb3REeTVTVnJOWkJzdzFLdm5OTTlvVTVmQi8vYVdTckdMUXZDQ1RXc3lxTWhHbE1DUUJQd3RuRnlBT1pTQ2xmK3dWenBFUGhmNVc4VjZnOE9ZMG0vcmg0VWtyMDNHcVg3d0N1aEZrNC9YK1VqeDdOQlVudmJaVjZNbkRDQmowRFJBZmVCYVpjdEpYVHJJR2VSY1VKNFJzbCs0VE4rTjZub1E3WjJld25CWlk4SXl6SlROYUlIQ2RkV0plSkduMFdhaUZvYWxFUmlqUlN6bGd0S2JpTHRDL0pkMi8rU3JKSVJVUHE4Q2hyS2xGMjRidDFSeU9JcWVHUlhCL0o4YVUzQUpEcjlCUGU1R2pmY01ZWS9jYjFtMW00UzB5NXVVaTNvYjRwUllsL3E2ZW84TTFZSWpFSEsxSUJBN2lHUGtCcVY3MXNPVlAzVC82b0swWnBweHFDd0FETWJmd0FyOVNDTVZ1bjVQcG5JOU1PVmpFd2o3cEhNZEpKd21Ga2MxUUszbEUyeUJUNW8yaVB1cDhEb3pSTEVOekRlWG5VaHZNSlc2UE9KSWtvZXhMTjZUVStMRFkvQU1kNDBrM0d3SitsVlFkZWhBZWRzbllOLzY0K2xBSUlDall1cGdLMHRHMjVDK1lTdGQxNHlBTS9hVEZjWjlNWHlEemJzNDMzZi9PU2xnaC8raFV5Vk41WjI2b2JyZzZVWHlYTjZGQ2FlZDRxdi9oQ2pNN3RpRGRIY21iUWxoSDNXaHBCUGxKSnF1SXlpcTh3S213a0tKdU5XeFNxUTl0cG9McGdFbkE1NXVjUElmWHpld3VLYjZRPT0gcm9vdEBvcmZkbnMKICAgIHN1ZG86IEFMTD0oQUxMKSBOT1BBU1NXRDpBTEwKICAgIGdyb3Vwczogc3VkbywgZG9ja2VyCiAgICBzaGVsbDogL2Jpbi9iYXNoCm5ldHdvcms6CiAgdmVyc2lvbjogMgogIGV0aGVybmV0czoKICAgICAgZW5zMTkyOgogICAgICAgICAgZGhjcDQ6IHRydWUKcGFja2FnZV91cGRhdGU6IHRydWUKcGFja2FnZXM6CiAgLSBteXNxbC1zZXJ2ZXIKICAtIG5ldC10b29scwpydW5jbWQ6CiAgLSBzeXN0ZW1jdGwgZW5hYmxlIG15c3FsZAogIC0gc3lzdGVtY3RsIHN0YXJ0IG15c3FsZAogIC0gc3VkbyBteXNxbCAtZSAiQ1JFQVRFIERBVEFCQVNFIHdvcmRwcmVzczsiCiAgLSBzdWRvIG15c3FsIC1lICJDUkVBVEUgVVNFUiAnd29yZHByZXNzX3VzZXInQCclJyBJREVOVElGSUVEIEJZICdwYXNzd29yZCc7IgogIC0gc3VkbyBteXNxbCAtZSAiR1JBTlQgQUxMIE9OIHdvcmRwcmVzcy4qIFRPICd3b3JkcHJlc3NfdXNlcidAJyUnIgogIC0gc3VkbyBteXNxbCAtZSAiRkxVU0ggUFJJVklMRUdFUzsiCiAgLSBzZWQgLWkgJyRhYmluZC1hZGRyZXNzPTAuMC4wLjAnIC9ldGMvbXkuY25mLmQvbXlzcWwtc2VydmVyLmNuZgogIC0gc3lzdGVtY3RsIHJlc3RhcnQgbXlzcWxkCiAgLSBmaXJld2FsbC1vZmZsaW5lLWNtZCAtLWFkZC1wb3J0PTMzMDYvdGNwCiAgLSBmaXJld2FsbC1jbWQgLS1yZWxvYWQKICAtIHN5c3RlbWN0bCByZXN0YXJ0IHNzaGQK[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# 
```






 
