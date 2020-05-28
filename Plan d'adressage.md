# PLAN D'ADRESSAGE
```
Masque Ipv6 par défaut : /64
Masque Ipv4 par défaut : /24
```

| Périphérique  |Interfaces  |Infos  | Adresse ipv4  | IPv6 Link-local |
|:---:|:-----:|:-----:|:----:|:----:|
R1 | Gi0/0 | connected to INTERNET | DHCP | fe80::cafe:3 |
R1 | Gi0/2 | connected to R2 on port Gi0/1 | 10.1.1.1 | fe80::1 |
R1 | Gi0/3 | connected to R3 on port Gi0/1 | 10.1.2.1 | fe80::1 |
R2 | Gi0/1 | connected to R1 on port Gi0/2 | 10.1.1.2 | fe80::2 |
R2 | Gi0/2 | connected to DS1 on port Gi2/0 | 10.2.1.2 | fe80::2 |
R2 | Gi0/3 | connected to R3 on port Gi0/2 | 10.1.3.2 | fe80::2 | 
R2 | Gi0/4 | connected to DS1 on port Gi3/0 | 10.2.2.2 | fe80::2 |
R2 | Gi0/5 | connected to DS2 on port Gi2/1 | 10.2.3.2 | fe80::2 |
R2 | Gi0/6 | connected to DS2 on port Gi3/1 | 10.2.4.2 | fe80::2 |
R2 | Gi0/8 | connected to DS3 on port Gi3/1 | 10.2.5.1 | fe80::2 |
R2 | Gi0/9 | connected to DS3 on port Gi3/1 | 10.2.6.1 | fe80::2 |
R2 | Gi0/10 | connected to DS4 on port Gi2/0 | 10.2.7.1 | fe80::2 |
R2 | Gi0/11 | connected to DS4 on port Gi3/0 | 10.2.8.1 | fe80::2 |
R3 | Gi0/1 | connected to R1 on port Gi2/1 | 10.1.2.3 | fe80::3 |
R3 | Gi0/2 | connected to R2 on port Gi3/1 | 10.1.3.3 | fe80::3 |
R3 | Gi0/3 | connected to DS2 on port Gi2/0 | 10.3.1.3 | fe80::3 |
R3 | Gi0/4 | connected to DS2 on port Gi3/0 | 10.3.2.3 | fe80::3 |
R3 | Gi0/5 | connected to DS1 on port Gi2/1 | 10.3.3.3 | fe80::3 |
R3 | Gi0/6 | connected to DS1 on port Gi3/1 | 10.3.4.3 | fe80::3 |
R3 | Gi0/8 | connected to DS3 on port Gi2/1 | 10.3.5.1 | fe80::3 |
R3 | Gi0/9 | connected to DS3 on port Gi3/1 | 10.3.6.1 | fe80::3 |
R3 | Gi0/10 | connected to DS4 on port Gi2/0 | 10.3.7.1 | fe80::3 |
R3 | Gi0/11 | connected to DS4 on port Gi3/0 | 10.3.8.1 | fe80::3 |
DS1 | Gi2/0 | connected to R2 on port Gi0/2 | 10.2.1.1 | fe80::d1 |
DS1 | Gi3/0 | connected to R2 on port Gi0/4 | 10.2.2.1 | fe80::d1 |
DS1 | Gi2/1 | connected to R3 on port Gi0/5 | 10.3.3.1 | fe80::d1 |
DS1 | Gi3/1 | connected to R3 on port Gi0/6 | 10.3.4.1 | fe80::d1 |
DS2 | Gi2/1 | connected to R2 on port Gi0/5 | 10.2.3.1 | fe80::d2 |
DS2 | Gi3/1 | connected to R2 on port Gi0/6 | 10.2.4.1 | fe80::d2 |
DS2 | Gi2/0 | connected to R3 on port Gi0/3 | 10.3.1.2 | fe80::d2 | 
DS2 | Gi3/0 | connected to R3 on port Gi0/4 | 10.3.2.2 | fe80::d2 |
DS3 | Gi2/0 | connected to R2 on port Gi0/8 | 10.2.5.2 | fe80::d3 |
DS3 | Gi3/0 | connected to R2 on port Gi0/9 | 10.2.6.2 | fe80::d3 |
DS3 | Gi2/1 | connected to R3 on port Gi0/8 | 10.3.5.2 | fe80::d3 |
DS3 | Gi3/1 | connected to R3 on port Gi0/9 | 10.3.6.2 | fe80::d3 |
DS4 | Gi2/1 | connected to R2 on port Gi0/10 | 10.2.7.2 | fe80::d4 |
DS4 | Gi3/1 | connected to R2 on port Gi0/11 | 10.2.8.2 | fe80::d4 |
DS4 | Gi2/0 | connected to R3 on port Gi0/10 | 10.3.7.2 | fe80::d4 | 
DS4 | Gi3/0 | connected to R3 on port Gi0/11 | 10.3.8.2 | fe80::d4 |
DS1 | VLAN10 | Port Access Gi2/0 | 10.192.10.254 | fe80::d0 |
DS1 | VLAN20 | Port Access Gi2/1 | 10.192.20.254 | fe80::d0 |
DS1 | VLAN30 | Port Access Gi2/2 | 10.192.30.254 | fe80::d0 |
DS1 | VLAN40 | Port Access Gi2/3 | 10.192.40.254 | fe80::d0 |
DS2 | VLAN10 | Port Access Gi2/0 | 10.192.10.254 | fe80::d0 |
DS2 | VLAN20 | Port Access Gi2/1 | 10.192.20.254 | fe80::d0 |
DS2 | VLAN30 | Port Access Gi2/2 | 10.192.30.254 | fe80::d0 |
DS2 | VLAN40 | Port Access Gi2/3 | 10.192.40.254 | fe80::d0 |
DS3 | VLAN10 | Port Access Gi2/0 | 10.193.10.254 | fe80::d0 |
DS3 | VLAN20 | Port Access Gi2/1 | 10.193.20.254 | fe80::d0 |
DS3 | VLAN30 | Port Access Gi2/2 | 10.193.30.254 | fe80::d0 |
DS3 | VLAN40 | Port Access Gi2/3 | 10.193.40.254 | fe80::d0 |
DS4 | VLAN10 | Port Access Gi2/0 | 10.193.10.254 | fe80::d0 |
DS4 | VLAN20 | Port Access Gi2/1 | 10.193.20.254 | fe80::d0 |
DS4 | VLAN30 | Port Access Gi2/2 | 10.193.30.254 | fe80::d0 |
DS4 | VLAN40 | Port Access Gi2/3 | 10.193.40.254 | fe80::d0 |
```
Passerelle IPv6 vers l'internet : fe80::e53:21ff:fe38:5800
```

# VLANS
*Switchblock 1*
VLAN | IPv4 | IPv6 privée | IPv6 publique |
|:---:|:-----:|:----:|:----:|
| vlan 10 | 10.192.10.0/24 | fd00:470:c814:3010::/64 | 2001:470:c814:3010::/64 |
| vlan 20 | 10.192.20.0/24 | fd00:470:c814:3020::/64 | 2001:470:c814:3020::/64 |
| vlan 30 | 10.192.30.0/24 | fd00:470:c814:3030::/64 | 2001:470:c814:3030::/64 |
| vlan 40 | 10.192.40.0/24 | fd00:470:c814:3040::/64 | 2001:470:c814:3040::/64 |
| vlan 99 | 10.192.1.0/24 | vlan natif & gestion

*Switchblock 2*
VLAN | IPv4 | IPv6 privée | IPv6 publique |
|:---:|:-----:|:----:|:----:|
| vlan 10 | 10.193.10.0/24 | fd00:470:c814:3010::/64 | 2001:470:c814:3010::/64 |
| vlan 20 | 10.193.20.0/24 | fd00:470:c814:3020::/64 | 2001:470:c814:3020::/64 |
| vlan 30 | 10.193.30.0/24 | fd00:470:c814:3030::/64 | 2001:470:c814:3030::/64 |
| vlan 40 | 10.193.40.0/24 | fd00:470:c814:3040::/64 | 2001:470:c814:3040::/64 |
| vlan 99 | 10.193.1.0/24 | vlan natif & gestion
```
Le Vlan de gestion et Vlan natif pourraient être séparés. 
Le Vlan de gestion n'est pas utile dans notre cas (déploiement par Ansible).
Toutefois, nous l'avons conservé pour des raisons de sécurité (si l'accès distant à partir de notre console de controle tombe par exemple).
```
## Portchannel
| Périphérique  | PortChannel | Port Physique |  Infos
|:---:|:-----:|:----:|:----:|
DS1/DS3 | po1 | Gi0/0, Gi1/0 | Link to AS1 |
DS1/DS3 | po11 | Gi0/1, Gi1/1 | Link to AS2 |
DS1/DS3 | po3 | Gi0/2, Gi1/2 | Link to DS2 | 
DS2/DS4 | po2 | Gi0/0, Gi1/0 | Link to AS2 |
DS2/DS4 | po22 | Gi0/1, Gi1/1 | Link to AS1 |
DS2/DS4 | po3 | Gi0/2, Gi1/2  | Link to DS1 |

## Spanning-Tree & HSRP
| Périphérique  | Interface | Spanning-Tree |  Adresse ipv4 | IPv6 Link-local | IPv6 privée | IPv6 publique |Group |	Priorité |
|:---:|:-----:|----|:----:|:----:|:----:|:----:|:----:|:----:|
DS1 | VLAN10 | Root Primary | 10.192.10.252/24 | fe80::d1 | fd00:470:c814:3010::252 | 2001:470:c814:3010::252 | 10/16 |	150, prempt
DS1 | VLAN20 | Root Secondary | 10.192.20.252/24 | fe80::d1 | fd00:470:c814:3020::252 | 2001:470:c814:3020::252 |20/26 |	default
DS1 | VLAN30 | Root Primary | 10.192.30.252/24 | fe80::d1 | fd00:470:c814:3030::252 | 2001:470:c814:3030::252 | 30/36 |	150, prempt
DS1 | VLAN40 | Root Secondary | 10.192.40.252/24 | fe80::d1 | fd00:470:c814:3040::252 | 2001:470:c814:3040::252 |40/46 |	default
DS2 | VLAN10 | Root Secondary | 10.192.10.253/24 | fe80::d2 | fd00:470:c814:3010::253 | 2001:470:c814:3010::253 | 10/16 |	default
DS2 | VLAN20 | Root Primary | 10.192.20.253/24 | fe80::d2 | fd00:470:c814:3020::253 | 2001:470:c814:3020::253 | 20/26 |	150, prempt
DS2 | VLAN30 | Root Secondary | 10.192.30.253/24 | fe80::d2 | fd00:470:c814:3030::253 | 2001:470:c814:3030::253 | 30/36 |	default
DS2 | VLAN40 | Root Primary | 10.192.40.253/24 | fe80::d2 | fd00:470:c814:3040::253 | 2001:470:c814:3040::253 | 40/46 |	150, prempt
DS3 | VLAN10 | Root Primary | 10.193.10.250/24 | fe80::d3 | fd00:470:c814:3010::250 | 2001:470:c814:3010::250 | 10/16 |	150, prempt
DS3 | VLAN20 | Root Secondary | 10.193.20.250/24 | fe80::d3 | fd00:470:c814:3020::250 | 2001:470:c814:3020::250 |20/26 |	default
DS3 | VLAN30 | Root Primary | 10.193.30.250/24 | fe80::d3 | fd00:470:c814:3030::250 | 2001:470:c814:3030::250 | 30/36 |	150, prempt
DS3 | VLAN40 | Root Secondary | 10.193.40.250/24 | fe80::d3 | fd00:470:c814:3040::250 | 2001:470:c814:3040::250 |40/46 |	default
DS4 | VLAN10 | Root Secondary | 10.193.10.251/24 | fe80::d4 | fd00:470:c814:3010::251 | 2001:470:c814:3010::251 | 10/16 |	default
DS4 | VLAN20 | Root Primary | 10.193.20.251/24 | fe80::d4 | fd00:470:c814:3020::251 | 2001:470:c814:3020::251 | 20/26 |	150, prempt
DS4 | VLAN30 | Root Secondary | 10.193.30.251/24 | fe80::d4 | fd00:470:c814:3030::251 | 2001:470:c814:3030::251 | 30/36 |	default
DS4 | VLAN40 | Root Primary | 10.193.40.251/24 | fe80::d4 | fd00:470:c814:3040::251 | 2001:470:c814:3040::251 | 40/46 |	150, prempt
