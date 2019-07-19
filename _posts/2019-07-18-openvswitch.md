---
layout: post
title: OpenVSwitch
---

## Présentation

## Lexique

### Bridge


### Port

## Draft


Pour demander un ip au dhcp
```
sudo dhclient NIC_NAME
```

Pour activer une interface réseau (nic)
```
sudo ip link set NIC_NAME up
```

Pour désactiver une interface réseau (nic)
```
sudo ip link set NIC_NAME down
```

Pour supprimer l'ip d'une interface (nic)
```
sudo ip addr del IP/MASK dev NIC_NAME
```