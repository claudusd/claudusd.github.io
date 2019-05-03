---
layout: post
title: Proxmox, Terraform et Cloud-Init
---

## Problématique

Nous créons des machines virtuelles sur un Cluster Proxmox avec Terraform. 
Lors du ``apply`` de Terraform nous utilisons des provisioners pour executer
des playbook Ansible. Mais nos playbook crachent car apt est lock. 

Le apt est lock car Cloud-Init execute une mise à jour des packages. 


## Template Proxmox

Création de notre template

```
#  configure a CDROM drive which will be used to pass the Cloud-Init data
#  to the VM    
qm set "${vmid}" --ide2 "local:cloudinit"
```

le binding avec ide2 est réalisé du côté de l'interface du PVE. 

Datacenter -> pve1 -> templates -> hardware

![](/images/posts/a/pve_cloud-init.png)

## Cloud-Init Datasources

```
cat /dev/sr0
```

```
instance-id: 0fc18e419e44e0acedd05630e70f3075c216da7d
version: 1
config:
    - type: physical
      name: eth0
      mac_address: 'fe:59:0f:26:fa:b2'
      subnets:
      - type: dhcp4
    - type: nameserver
      address:
      - '192.168.100.74'
      search:
      - 'cri.local.isima.fr'
#cloud-config
hostname: test-claude
manage_etc_hosts: true
user: claude
password: $5$a2/hOXIi$AEyT8ObnHeEAA6UejCoTLeqx2xZWChUWuRRBQv6iKz1
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDTcCU0rGjs2oXT/8l50/6EY9/kEdn+Inmor0KrhQPohZceB9fduhc6IlbpkoQVEWDhl/dCRWYAsLMRBvvwllnss2xC2AnHiPCUM2gnlxtMbkS/CYZoIaUhP581mtIL6RxhN3RbN11g1ybBXX9CchXnNPJn1L0a5nIvYkppbDUl/2fmnlXX/H9QZnZdzhIR/ixT8R9MVgFqzB0SD5Rm+gudDoEdE1kIESNE/3GhrCcSaafrOJVl/amq8c8bz9GwtbeRww8ivAGX/KQMFzgSK7jXo2ww9W/SPhxo1otLUoj5+GXvcCBHi8nbbCD1aNfIhJNtdFXTB6Rw9DPuScDrEzGW3POcPTgmLYgurJyBUxP8V4VsogF0gulj/LJmA/VOWdnh9ijHtGat06zWxR7ypHEJEfJqeguwJx8TNSo65lYMwgyKWYLDQJYMetn2aABioiae+h/hZ32H2eOvJtPj4wWqXDsf9vpa0BIredQ9n2hjxMSUTrQ7SzlosfUHWz/so3S2O4K9pc18T3aZWTyRsxpGwNm6Q92iDxxPI2tqnJF6BHcR9e8DD0nARFwqcOx1xEcSljb9gkImYseGpFTBarWesaAF4lOGpjY9DGtajG7VionQUCZ01Mp6g3baeihkfJCq6B8lremVo+EjJisEnURJk7IEBvxSoqklwx9ZAwp0Yw== claude.dioudonnat@isima.fr
chpasswd:
  expire: False
users:
  - default
package_upgrade: true
```

Cette information est produite ici : [PVE/QemuServer/Cloudinit.pm#143](https://git.proxmox.com/?p=qemu-server.git;a=blob;f=PVE/QemuServer/Cloudinit.pm;h=f46f7fdec65271fcc141dcb08c41545075223dc9;hb=HEAD#l143)


