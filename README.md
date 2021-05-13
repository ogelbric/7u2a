# vCenter update 7.0.2a (7u2a) delivers the VMware Operator
This is a wordpress example

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
[root@orfdns 7u2a]# k apply -f ./cluster.yaml 
tanzukubernetescluster.run.tanzu.vmware.com/tkg-berlin created
[root@orfdns 7u2a]# 
```
Here is the cluster YAMl file
```
# cluster.yaml
---
apiVersion: run.tanzu.vmware.com/v1alpha1
kind: TanzuKubernetesCluster
metadata:
  name: tkg-berlin
  namespace: namespace1000
spec:
  distribution:
    version: v1.18.15
  topology:
    controlPlane:
      count: 1
      class: best-effort-medium
      storageClass: pacific-gold-storage-policy
      volumes:
        - name: etcd
          mountPath: /var/lib/etcd
          capacity:
            storage: 4Gi
    workers:
      count: 2
      class: best-effort-medium
      storageClass: pacific-gold-storage-policy
      volumes:
        - name: containerd
          mountPath: /var/lib/containerd
          capacity:
            storage: 30Gi
  settings:
    network:
      services:
        cidrBlocks: ["198.51.100.0/24"]
      pods:
        cidrBlocks: ["192.0.2.0/22"]
    storage:
      classes: ["pacific-gold-storage-policy"]
      defaultClass: pacific-gold-storage-policy      
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
## Update the vm.yaml (user-data) with the above base 64 conversion
```
# vm.yml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: namespace1000
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: pacific-gold-storage-policy
  volumeMode: Filesystem
---
apiVersion: vmoperator.vmware.com/v1alpha1
kind: VirtualMachine
metadata:
  labels:
    vm-selector: mysql-centosvm
  name: mysql-centosvm
  namespace: namespace1000
spec:
  imageName: centos-stream-8-vmservice-v1alpha1-1619529007339
  className: best-effort-small
  powerState: poweredOn
  storageClass: pacific-gold-storage-policy
  networkInterfaces:
  - networkType: nsx-t
    networkName: ""
  volumes:
  - name: my-centos-vol
    persistentVolumeClaim:
      claimName: mysql-pvc
  readinessProbe:
    tcpSocket:
      port: 22
  vmMetadata:
    configMapName: centos-cloudinit
    transport: OvfEnv
---
apiVersion: v1
kind: ConfigMap
metadata:
    name: centos-cloudinit
    namespace: namespace1000
data:
  user-data: I2Nsb3VkLWNvbmZpZwpjaHBhc3N3ZDoKICAgIGxpc3Q6IHwKICAgICAgY2VudG9zOlZNd2FyZTEhCiAgICBleHBpcmU6IGZhbHNlCnNzaF9wd2F1dGg6IFRydWUKZ3JvdXBzOgogIC0gZG9ja2VyCnVzZXJzOgogIC0gZGVmYXVsdAogIC0gbmFtZTogY2VudG9zCiAgICBzc2gtYXV0aG9yaXplZC1rZXlzOgogICAgICAtIHNzaC1yc2EgQUFBQUIzTnphQzF5YzJFQUFBQURBUUFCQUFBQ0FRQ2pyNUxZVlRhNjNjNFhNc2g4TzY4MFFaYWo5WlBRN1J5U2JuSDgxTWJxSktmMmdmbXdWSHFvUEwwZ0VCM3lwS2oxb3REeTVTVnJOWkJzdzFLdm5OTTlvVTVmQi8vYVdTckdMUXZDQ1RXc3lxTWhHbE1DUUJQd3RuRnlBT1pTQ2xmK3dWenBFUGhmNVc4VjZnOE9ZMG0vcmg0VWtyMDNHcVg3d0N1aEZrNC9YK1VqeDdOQlVudmJaVjZNbkRDQmowRFJBZmVCYVpjdEpYVHJJR2VSY1VKNFJzbCs0VE4rTjZub1E3WjJld25CWlk4SXl6SlROYUlIQ2RkV0plSkduMFdhaUZvYWxFUmlqUlN6bGd0S2JpTHRDL0pkMi8rU3JKSVJVUHE4Q2hyS2xGMjRidDFSeU9JcWVHUlhCL0o4YVUzQUpEcjlCUGU1R2pmY01ZWS9jYjFtMW00UzB5NXVVaTNvYjRwUllsL3E2ZW84TTFZSWpFSEsxSUJBN2lHUGtCcVY3MXNPVlAzVC82b0swWnBweHFDd0FETWJmd0FyOVNDTVZ1bjVQcG5JOU1PVmpFd2o3cEhNZEpKd21Ga2MxUUszbEUyeUJUNW8yaVB1cDhEb3pSTEVOekRlWG5VaHZNSlc2UE9KSWtvZXhMTjZUVStMRFkvQU1kNDBrM0d3SitsVlFkZWhBZWRzbllOLzY0K2xBSUlDall1cGdLMHRHMjVDK1lTdGQxNHlBTS9hVEZjWjlNWHlEemJzNDMzZi9PU2xnaC8raFV5Vk41WjI2b2JyZzZVWHlYTjZGQ2FlZDRxdi9oQ2pNN3RpRGRIY21iUWxoSDNXaHBCUGxKSnF1SXlpcTh3S213a0tKdU5XeFNxUTl0cG9McGdFbkE1NXVjUElmWHpld3VLYjZRPT0gcm9vdEBvcmZkbnMKICAgIHN1ZG86IEFMTD0oQUxMKSBOT1BBU1NXRDpBTEwKICAgIGdyb3Vwczogc3VkbywgZG9ja2VyCiAgICBzaGVsbDogL2Jpbi9iYXNoCm5ldHdvcms6CiAgdmVyc2lvbjogMgogIGV0aGVybmV0czoKICAgICAgZW5zMTkyOgogICAgICAgICAgZGhjcDQ6IHRydWUKcGFja2FnZV91cGRhdGU6IHRydWUKcGFja2FnZXM6CiAgLSBteXNxbC1zZXJ2ZXIKICAtIG5ldC10b29scwpydW5jbWQ6CiAgLSBzeXN0ZW1jdGwgZW5hYmxlIG15c3FsZAogIC0gc3lzdGVtY3RsIHN0YXJ0IG15c3FsZAogIC0gc3VkbyBteXNxbCAtZSAiQ1JFQVRFIERBVEFCQVNFIHdvcmRwcmVzczsiCiAgLSBzdWRvIG15c3FsIC1lICJDUkVBVEUgVVNFUiAnd29yZHByZXNzX3VzZXInQCclJyBJREVOVElGSUVEIEJZICdwYXNzd29yZCc7IgogIC0gc3VkbyBteXNxbCAtZSAiR1JBTlQgQUxMIE9OIHdvcmRwcmVzcy4qIFRPICd3b3JkcHJlc3NfdXNlcidAJyUnIgogIC0gc3VkbyBteXNxbCAtZSAiRkxVU0ggUFJJVklMRUdFUzsiCiAgLSBzZWQgLWkgJyRhYmluZC1hZGRyZXNzPTAuMC4wLjAnIC9ldGMvbXkuY25mLmQvbXlzcWwtc2VydmVyLmNuZgogIC0gc3lzdGVtY3RsIHJlc3RhcnQgbXlzcWxkCiAgLSBmaXJld2FsbC1vZmZsaW5lLWNtZCAtLWFkZC1wb3J0PTMzMDYvdGNwCiAgLSBmaXJld2FsbC1jbWQgLS1yZWxvYWQKICAtIHN5c3RlbWN0bCByZXN0YXJ0IHNzaGQK
  hostname: centos-mysql
---
apiVersion: vmoperator.vmware.com/v1alpha1
kind: VirtualMachineService
metadata:
  name: mysql-vmservices
spec:
  ports:
  - name: ssh
    port: 22
    protocol: TCP
    targetPort: 22
  - name: mwsql
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    vm-selector: mysql-centosvm
  type: LoadBalancer
  ```
  ## Some commannds to make sure everything is good
  ```
[root@orfdns 7u2a]# k get nodes
NAME                               STATUS   ROLES    AGE   VERSION
420c9309ec6670a9a96e93ec914a445d   Ready    master   20h   v1.19.1+wcp.3
420c9eba86c0cc85380c88e784bddcd7   Ready    master   20h   v1.19.1+wcp.3
420ca783dbc48c2746ce30f6f42698b1   Ready    master   20h   v1.19.1+wcp.3

[root@orfdns 7u2a]# kubectl get virtualmachineclassbindings
NAME                  VIRTUALMACHINECLASS   AGE
best-effort-2xlarge   best-effort-2xlarge   175m
best-effort-4xlarge   best-effort-4xlarge   175m
best-effort-8xlarge   best-effort-8xlarge   175m
best-effort-large     best-effort-large     175m
best-effort-medium    best-effort-medium    175m
best-effort-small     best-effort-small     175m
best-effort-xlarge    best-effort-xlarge    175m
best-effort-xsmall    best-effort-xsmall    175m

[root@orfdns 7u2a]# kubectl get virtualmachineimages
NAME                                                         VERSION                           OSTYPE                FORMAT   AGE
centos-stream-8-vmservice-v1alpha1-1619529007339                                               centos8_64Guest       ovf      173m
ob-15957779-photon-3-k8s-v1.16.8---vmware.1-tkg.3.60d2ffd    v1.16.8+vmware.1-tkg.3.60d2ffd    vmwarePhoton64Guest   ovf      20h
ob-16466772-photon-3-k8s-v1.17.7---vmware.1-tkg.1.154236c    v1.17.7+vmware.1-tkg.1.154236c    vmwarePhoton64Guest   ovf      20h
ob-16545581-photon-3-k8s-v1.16.12---vmware.1-tkg.1.da7afe7   v1.16.12+vmware.1-tkg.1.da7afe7   vmwarePhoton64Guest   ovf      20h
ob-16551547-photon-3-k8s-v1.17.8---vmware.1-tkg.1.5417466    v1.17.8+vmware.1-tkg.1.5417466    vmwarePhoton64Guest   ovf      20h
ob-16897056-photon-3-k8s-v1.16.14---vmware.1-tkg.1.ada4837   v1.16.14+vmware.1-tkg.1.ada4837   vmwarePhoton64Guest   ovf      20h
ob-16924026-photon-3-k8s-v1.18.5---vmware.1-tkg.1.c40d30d    v1.18.5+vmware.1-tkg.1.c40d30d    vmwarePhoton64Guest   ovf      20h
ob-16924027-photon-3-k8s-v1.17.11---vmware.1-tkg.1.15f1e18   v1.17.11+vmware.1-tkg.1.15f1e18   vmwarePhoton64Guest   ovf      20h
ob-17010758-photon-3-k8s-v1.17.11---vmware.1-tkg.2.ad3d374   v1.17.11+vmware.1-tkg.2.ad3d374   vmwarePhoton64Guest   ovf      20h
ob-17332787-photon-3-k8s-v1.17.13---vmware.1-tkg.2.2c133ed   v1.17.13+vmware.1-tkg.2.2c133ed   vmwarePhoton64Guest   ovf      20h
ob-17419070-photon-3-k8s-v1.18.10---vmware.1-tkg.1.3a6cd48   v1.18.10+vmware.1-tkg.1.3a6cd48   vmwarePhoton64Guest   ovf      20h
ob-17654937-photon-3-k8s-v1.18.15---vmware.1-tkg.1.600e412   v1.18.15+vmware.1-tkg.1.600e412   vmwarePhoton64Guest   ovf      20h
ob-17658793-photon-3-k8s-v1.17.17---vmware.1-tkg.1.d44d45a   v1.17.17+vmware.1-tkg.1.d44d45a   vmwarePhoton64Guest   ovf      20h
ob-17660956-photon-3-k8s-v1.19.7---vmware.1-tkg.1.fc82c41    v1.19.7+vmware.1-tkg.1.fc82c41    vmwarePhoton64Guest   ovf      20h
ob-17861429-photon-3-k8s-v1.20.2---vmware.1-tkg.1.1d4f79a    v1.20.2+vmware.1-tkg.1.1d4f79a    vmwarePhoton64Guest   ovf      20h

[root@orfdns 7u2a]# kubectl get resourcequotas
NAME                         AGE    REQUEST                                                                                           LIMIT
namespace1000                3h5m   requests.storage: 0/200Gi                                                                         
namespace1000-storagequota   3h5m   pacific-gold-storage-policy.storageclass.storage.k8s.io/requests.storage: 0/9223372036854775807   

[root@orfdns 7u2a]# k get sc
NAME                          PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
pacific-gold-storage-policy   csi.vsphere.vmware.com   Delete          Immediate           true                   3h15m
```
## Get the network name (update the vm.yaml file with this)
```
[root@orfdns 7u2a]# kubectl get network
NAME        AGE
network-1   4h2m
[root@orfdns 7u2a]# 
```
## Deploy the VM via YAML file
```
[root@orfdns 7u2a]# k apply -f ./vm.yaml 
persistentvolumeclaim/mysql-pvc created
virtualmachine.vmoperator.vmware.com/mysql-centosvm created
configmap/centos-cloudinit created
virtualmachineservice.vmoperator.vmware.com/mysql-vmservices created
[root@orfdns 7u2a]# 
```
## Here is the YAML (in this case for NSX ALB (AVI) commented out is the NSX-T network configuration

```
# vm.yml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: namespace1000
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
  storageClassName: pacific-gold-storage-policy
  volumeMode: Filesystem
---
apiVersion: vmoperator.vmware.com/v1alpha1
kind: VirtualMachine
metadata:
  labels:
    vm-selector: mysql-centosvm
  name: mysql-centosvm
  namespace: namespace1000
spec:
  imageName: centos-stream-8-vmservice-v1alpha1-1619529007339
  className: best-effort-small
  powerState: poweredOn
  storageClass: pacific-gold-storage-policy
#NSX networking
#  networkInterfaces:
#  - networkType: nsx-t
#    networkName: ""
#distributed switch in this case NSX ALD (AVI)
#kubectl get network
  networkInterfaces:
  - networkName: "network-1"
    networkType: vsphere-distributed
  volumes:
  - name: my-centos-vol
    persistentVolumeClaim:
      claimName: mysql-pvc
  readinessProbe:
    tcpSocket:
      port: 22
  vmMetadata:
    configMapName: centos-cloudinit
    transport: OvfEnv
---
apiVersion: v1
kind: ConfigMap
metadata:
    name: centos-cloudinit
    namespace: namespace1000
data:
  user-data: I2Nsb3VkLWNvbmZpZwpjaHBhc3N3ZDoKICAgIGxpc3Q6IHwKICAgICAgY2VudG9zOlZNd2FyZTEhCiAgICBleHBpcmU6IGZhbHNlCnNzaF9wd2F1dGg6IFRydWUKZ3JvdXBzOgogIC0gZG9ja2VyCnVzZXJzOgogIC0gZGVmYXVsdAogIC0gbmFtZTogY2VudG9zCiAgICBzc2gtYXV0aG9yaXplZC1rZXlzOgogICAgICAtIHNzaC1yc2EgQUFBQUIzTnphQzF5YzJFQUFBQURBUUFCQUFBQ0FRQ2pyNUxZVlRhNjNjNFhNc2g4TzY4MFFaYWo5WlBRN1J5U2JuSDgxTWJxSktmMmdmbXdWSHFvUEwwZ0VCM3lwS2oxb3REeTVTVnJOWkJzdzFLdm5OTTlvVTVmQi8vYVdTckdMUXZDQ1RXc3lxTWhHbE1DUUJQd3RuRnlBT1pTQ2xmK3dWenBFUGhmNVc4VjZnOE9ZMG0vcmg0VWtyMDNHcVg3d0N1aEZrNC9YK1VqeDdOQlVudmJaVjZNbkRDQmowRFJBZmVCYVpjdEpYVHJJR2VSY1VKNFJzbCs0VE4rTjZub1E3WjJld25CWlk4SXl6SlROYUlIQ2RkV0plSkduMFdhaUZvYWxFUmlqUlN6bGd0S2JpTHRDL0pkMi8rU3JKSVJVUHE4Q2hyS2xGMjRidDFSeU9JcWVHUlhCL0o4YVUzQUpEcjlCUGU1R2pmY01ZWS9jYjFtMW00UzB5NXVVaTNvYjRwUllsL3E2ZW84TTFZSWpFSEsxSUJBN2lHUGtCcVY3MXNPVlAzVC82b0swWnBweHFDd0FETWJmd0FyOVNDTVZ1bjVQcG5JOU1PVmpFd2o3cEhNZEpKd21Ga2MxUUszbEUyeUJUNW8yaVB1cDhEb3pSTEVOekRlWG5VaHZNSlc2UE9KSWtvZXhMTjZUVStMRFkvQU1kNDBrM0d3SitsVlFkZWhBZWRzbllOLzY0K2xBSUlDall1cGdLMHRHMjVDK1lTdGQxNHlBTS9hVEZjWjlNWHlEemJzNDMzZi9PU2xnaC8raFV5Vk41WjI2b2JyZzZVWHlYTjZGQ2FlZDRxdi9oQ2pNN3RpRGRIY21iUWxoSDNXaHBCUGxKSnF1SXlpcTh3S213a0tKdU5XeFNxUTl0cG9McGdFbkE1NXVjUElmWHpld3VLYjZRPT0gcm9vdEBvcmZkbnMKICAgIHN1ZG86IEFMTD0oQUxMKSBOT1BBU1NXRDpBTEwKICAgIGdyb3Vwczogc3VkbywgZG9ja2VyCiAgICBzaGVsbDogL2Jpbi9iYXNoCm5ldHdvcms6CiAgdmVyc2lvbjogMgogIGV0aGVybmV0czoKICAgICAgZW5zMTkyOgogICAgICAgICAgZGhjcDQ6IHRydWUKcGFja2FnZV91cGRhdGU6IHRydWUKcGFja2FnZXM6CiAgLSBteXNxbC1zZXJ2ZXIKICAtIG5ldC10b29scwpydW5jbWQ6CiAgLSBzeXN0ZW1jdGwgZW5hYmxlIG15c3FsZAogIC0gc3lzdGVtY3RsIHN0YXJ0IG15c3FsZAogIC0gc3VkbyBteXNxbCAtZSAiQ1JFQVRFIERBVEFCQVNFIHdvcmRwcmVzczsiCiAgLSBzdWRvIG15c3FsIC1lICJDUkVBVEUgVVNFUiAnd29yZHByZXNzX3VzZXInQCclJyBJREVOVElGSUVEIEJZICdwYXNzd29yZCc7IgogIC0gc3VkbyBteXNxbCAtZSAiR1JBTlQgQUxMIE9OIHdvcmRwcmVzcy4qIFRPICd3b3JkcHJlc3NfdXNlcidAJyUnIgogIC0gc3VkbyBteXNxbCAtZSAiRkxVU0ggUFJJVklMRUdFUzsiCiAgLSBzZWQgLWkgJyRhYmluZC1hZGRyZXNzPTAuMC4wLjAnIC9ldGMvbXkuY25mLmQvbXlzcWwtc2VydmVyLmNuZgogIC0gc3lzdGVtY3RsIHJlc3RhcnQgbXlzcWxkCiAgLSBmaXJld2FsbC1vZmZsaW5lLWNtZCAtLWFkZC1wb3J0PTMzMDYvdGNwCiAgLSBmaXJld2FsbC1jbWQgLS1yZWxvYWQKICAtIHN5c3RlbWN0bCByZXN0YXJ0IHNzaGQK
  hostname: centos-mysql
---
apiVersion: vmoperator.vmware.com/v1alpha1
kind: VirtualMachineService
metadata:
  name: mysql-vmservices
spec:
  ports:
  - name: ssh
    port: 22
    protocol: TCP
    targetPort: 22
  - name: mwsql
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    vm-selector: mysql-centosvm
  type: LoadBalancer
```

## Check on the VM
```
[root@orfdns 7u2a]# k get vm
NAME                                        POWERSTATE   AGE
mysql-centosvm                              poweredOn    2m44s
tkg-berlin-control-plane-v65v8              poweredOn    3h53m
tkg-berlin-workers-jtz4k-75dddb8999-2xmld   poweredOn    97m
tkg-berlin-workers-jtz4k-75dddb8999-9n67g   poweredOn    3h42m
tkg-berlin-workers-jtz4k-75dddb8999-djdt2   poweredOn    97m
[root@orfdns 7u2a]# 
```
 ![running vm](https://github.com/ogelbric/7u2a/blob/main/vmrunning.png)

## Testing the ingress connection on port 22 and 3306
```[root@orfdns 7u2a]# k get svc
NAME                               TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)                       AGE
mysql-vmservices                   LoadBalancer   10.96.0.58   192.168.5.44   22:32553/TCP,3306:31187/TCP   16m
tkg-berlin-control-plane-service   LoadBalancer   10.96.0.94   192.168.5.42   6443:31266/TCP                4h7m
[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# telnet 192.168.5.44 22
Trying 192.168.5.44...
Connected to 192.168.5.44.
Escape character is '^]'.
SSH-2.0-OpenSSH_8.0
^]
telnet> quit
Connection closed.
[root@orfdns 7u2a]# telnet 192.168.5.44 3306
Trying 192.168.5.44...
Connected to 192.168.5.44.
Escape character is '^]'.
      mysql_native_password^]
telnet> quit
Connection closed.
```

## ssh to the VM (centos/VMware1! from the cloud-init.yaml file)
```
[root@orfdns 7u2a]# ssh centos@192.168.5.44
The authenticity of host '192.168.5.44 (192.168.5.44)' can't be established.
ECDSA key fingerprint is SHA256:idAUzAEW8iVfaJcS161dqd0UCG5qfJM4d3a8pd0N5cQ.
ECDSA key fingerprint is MD5:a3:b7:65:34:10:33:50:19:05:a5:a1:0b:15:15:19:da.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.5.44' (ECDSA) to the list of known hosts.
[centos@centos-mysql ~]$ 
[centos@centos-mysql ~]$ ifconfig -a
ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.7.58  netmask 255.255.255.0  broadcast 192.168.7.255
        inet6 fe80::250:56ff:fe8c:cbec  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:8c:cb:ec  txqueuelen 1000  (Ethernet)
        RX packets 10986  bytes 52814074 (50.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7493  bytes 469297 (458.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[centos@centos-mysql ~]$ 
```
## Log onto guest cluster and swap context (login alias shown)
```
[root@orfdns 7u2a]# alias l2540
alias l2540='kubectl vsphere login --server 192.168.5.40 \
                --vsphere-username administrator@vsphere.local \
                --managed-cluster-namespace namespace1000 \
                --managed-cluster-name tkg-berlin \
                --insecure-skip-tls-verify'
[root@orfdns 7u2a]# l2540

Password: 
Logged in successfully.

You have access to the following contexts:
   192.168.5.40
   namespace1000
   tkg-berlin

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
[root@orfdns 7u2a]# kubectl config use-context tkg-berlin
Switched to context "tkg-berlin".
[root@orfdns 7u2a]# k get nodes
NAME                                        STATUS   ROLES    AGE    VERSION
tkg-berlin-control-plane-v65v8              Ready    master   4h7m   v1.18.5+vmware.1
tkg-berlin-workers-jtz4k-75dddb8999-2xmld   Ready    <none>   113m   v1.18.5+vmware.1
tkg-berlin-workers-jtz4k-75dddb8999-9n67g   Ready    <none>   4h1m   v1.18.5+vmware.1
tkg-berlin-workers-jtz4k-75dddb8999-djdt2   Ready    <none>   113m   v1.18.5+vmware.1
[root@orfdns 7u2a]# 
```
## Deploy permissions YAML
```
[root@orfdns ~]# k apply -f ./authorize-psp-for-gc-service-accounts.yaml
clusterrole.rbac.authorization.k8s.io/psp:privileged created
clusterrolebinding.rbac.authorization.k8s.io/all:psp:privileged created
```
## Here is the YAML
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: psp:privileged
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - vmware-system-privileged
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: all:psp:privileged
roleRef:
  kind: ClusterRole
  name: psp:privileged
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
[root@orfdns ~]# 
```
## Deploy Wordpress

```
[root@orfdns 7u2a]# k apply -f ./wordpress.yaml 
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created
service/mysqldb created
endpoints/mysqldb created
```
## Here is the YAML (Note the image is in my google image repot it can also be found here: at docker wordpress:5.7-apache )
                    (Note IP at end of YAML file has ot be updated with result from k get svc)
```
# wordpress.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: gcr.io/boreal-rain-XXXXXXXXXXX/wordpress:5.7-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: "$(MYSQLDB_SERVICE_HOST)"
        - name: WORDPRESS_DB_PASSWORD
          value: "password"
        - name: WORDPRESS_DB_USER
          value: "wordpress_user"
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
#      imagePullSecrets:
#      - name: regcred
---
apiVersion: v1
kind: Service
metadata:
  name: mysqldb
spec:
  ports:
  - name: mysql
    port: 3306
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: mysqldb
subsets:
- addresses:
  - ip: 192.168.5.54
  ports:
  - name: mysql
    port: 3306
    protocol: TCP
```
## Testing if the POD is up
```
[root@orfdns 7u2a]# k get pods
NAME                         READY   STATUS    RESTARTS   AGE
wordpress-699976b7f4-rm585   1/1     Running   0          7m17s
[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# k get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS                  AGE
wp-pv-claim   Bound    pvc-41626c06-fcb9-4c98-b999-890603a89788   20Gi       RWO            pacific-gold-storage-policy   7m23s
[root@orfdns 7u2a]# k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS                  REASON   AGE
pvc-41626c06-fcb9-4c98-b999-890603a89788   20Gi       RWO            Delete           Bound    default/wp-pv-claim   pacific-gold-storage-policy            7m26s
[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# k get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
kubernetes   ClusterIP      198.51.100.1     <none>         443/TCP        16m
mysqldb      ClusterIP      198.51.100.236   <none>         3306/TCP       7m31s
supervisor   ClusterIP      None             <none>         6443/TCP       15m
wordpress    LoadBalancer   198.51.100.204   192.168.5.46   80:30043/TCP   7m32s
[root@orfdns 7u2a]# 
[root@orfdns 7u2a]# 
```

 ![wordpress](https://github.com/ogelbric/7u2a/blob/main/wordpress.png)


## Inspiration for all of this came from this blog
```
https://navneet-verma.medium.com/the-new-modern-workload-on-vsphere-fed63d57ff8d
```
## The official VMware doc is here 
```
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-F81E3535-C275-4DDE-B35F-CE759EA3B4A0.html
https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-5D254A77-AB6B-40AB-AB27-1AE6A917DC52.html
```
## My Personal AVI deployment/enablement can be found here 
```
https://github.com/ogelbric/7u2a/blob/mainTanzuWithAVI.pptx
```

## Trouble shooting
```
for this write up 
kubectl get events -w
is always good 
```

## VM console access 
I have to test this again
```
As Administrator:
Menu -> Administration -> Users and Groups -> Users -> Domain vsphere.local -> ADD -> username wcpadmin -> password VMware1! -> ADD
Menu -> Host and Clusters -> left side select vcenter -> Permissions -> + (add permission) -> Domain vsphere.local User/Group wcpadmin Role Administrator Propagate to Children (select) -> OK 
Menu -> Administration -> Users and Groups -> Groups -> Pages 2 find ServiceProvidersUsers -> select round circle -> Edit -> Add Memebers Search  type wcpadmin => green entry -> Save
Log out 
Log in as wcpadmin@vsphere.local
Select VM and open web console
```







 
