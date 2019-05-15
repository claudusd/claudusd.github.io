---
layout: post
title: "Vagrant et Virtualbox : Routing"
---

## Problématique

J'ai un VagrantFile qui me permet de lancer plusieurs VM. Toute mes VM sont dans un réseau privée.

## Création du réseau

``` VagrantFile
config.vm.define "opstctrl01" do |server|
    
    server.vm.hostname = "opstctrl01"
    server.vm.box = "centos/7"
    server.vm.network "private_network", ip: "192.168.50.2"
    server.vm.provider "virtualbox" do |v|
      v.memory = 4096
      v.cpus = 3
    end
  end
```

Je fixe moi même les IPs de mes VMs dans le réseau 192.168.50.0/24. J'utilise un ``private_network``. 
Le mode ``private_network`` de Vagrant correspond au mode ``host_only`` de VirtualBox [cf. Vagrant code source](https://github.com/hashicorp/vagrant/blob/v2.2.4/plugins/providers/virtualbox/action/network.rb#L66-L68) et [cf. Virtual Box Manuel 6.7. Host-Only Networking](https://www.virtualbox.org/manual/ch06.html)

## Gateway

Sur mes VM par défaut la gateway est sur 10.0.2.2. Je devoir passer par 192.168.50.1 qui est l'ip de la machine host.

Je dois d'abord supprimer cette gateway :
```
ip route del default via 10.0.2.2 dev eth0
```

Et je dois créer ma gateway via
```
ip route add default via 192.168.50.1 dev eth1
```



## Routage internet

Quand 

```
iptables -A FORWARD -o eno1 -i vboxnet1 -s 192.168.50.0/24 -m conntrack --ctstate NEW -j ACCEPT
```

```
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

```
iptables -t nat -F POSTROUTING
iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
```

iptables -t nat -A PREROUTING -i vboxnet1 -s 192.168.50.0/24 -p tcp --dport 80 -j REDIRECT --to-port 3128


source : 
 * [VirtualBox: Internet Access With Host-Only Network](https://kyrofa.com/posts/virtualbox-internet-access-with-host-only-network)
