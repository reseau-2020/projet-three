
# Etude Fiabilité

PC2 ip : 10.192.20.102/24 

PC6 ip : 10.192.20.2/24

DS2 est la route principale pour le VLAN20 (que on regarde avec la commande "show spanning-tree VLAN20" sur DS2)

## HSRP

#### Coupure de DS2

![HSRPDS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS2.png?raw=true)

On fait un ping de PC6 vers PC2. 
DS2 a été éteint a la requete 10. 
Le trafic reprend a partir de la requete donc hsrp fonctionnne bien, le trafic a changer de routeur. 

```
PC6> ping 10.192.20.102 -t
84 bytes from 10.192.20.102 icmp_seq=1 ttl=64 time=5.658 ms
84 bytes from 10.192.20.102 icmp_seq=2 ttl=64 time=5.439 ms
84 bytes from 10.192.20.102 icmp_seq=3 ttl=64 time=5.450 ms
84 bytes from 10.192.20.102 icmp_seq=4 ttl=64 time=6.179 ms
84 bytes from 10.192.20.102 icmp_seq=5 ttl=64 time=5.661 ms
84 bytes from 10.192.20.102 icmp_seq=6 ttl=64 time=5.501 ms
84 bytes from 10.192.20.102 icmp_seq=7 ttl=64 time=4.871 ms
84 bytes from 10.192.20.102 icmp_seq=8 ttl=64 time=6.107 ms
84 bytes from 10.192.20.102 icmp_seq=9 ttl=64 time=5.499 ms
10.192.20.102 icmp_seq=10 timeout
10.192.20.102 icmp_seq=11 timeout
10.192.20.102 icmp_seq=12 timeout
84 bytes from 10.192.20.102 icmp_seq=13 ttl=64 time=5.492 ms
84 bytes from 10.192.20.102 icmp_seq=14 ttl=64 time=6.667 ms
84 bytes from 10.192.20.102 icmp_seq=15 ttl=64 time=6.822 ms
84 bytes from 10.192.20.102 icmp_seq=16 ttl=64 time=5.601 ms
```

#### Coupure de DS1

`ping www.google.com` continu de centos-1. Petite latence de quelques secondes avant reprise.

## Spanning Tree et Etherchannel

#### Coupure d'un brin de po2 puis des 2 brins
![coupurepo2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20Spanning%20DS2.png?raw=true)

On fait un ping de PC6 vers PC2. 
Le premier timeout c'est quand on a fait tombé une interface entre AS2 et DS2. 
Les 3 timeout après c'est quand la seconde interface est tombé. 
On voit bien que le trafic reprend. 

```
PC6> ping 10.192.20.102 -t
84 bytes from 10.192.20.102 icmp_seq=1 ttl=64 time=4.329 ms
84 bytes from 10.192.20.102 icmp_seq=2 ttl=64 time=5.738 ms
84 bytes from 10.192.20.102 icmp_seq=3 ttl=64 time=5.315 ms
84 bytes from 10.192.20.102 icmp_seq=4 ttl=64 time=4.737 ms
10.192.20.102 icmp_seq=5 timeout
84 bytes from 10.192.20.102 icmp_seq=6 ttl=64 time=5.193 ms
10.192.20.102 icmp_seq=7 timeout
10.192.20.102 icmp_seq=8 timeout
10.192.20.102 icmp_seq=9 timeout
84 bytes from 10.192.20.102 icmp_seq=10 ttl=64 time=8.279 ms
84 bytes from 10.192.20.102 icmp_seq=11 ttl=64 time=6.798 ms
84 bytes from 10.192.20.102 icmp_seq=12 ttl=64 time=6.715 ms
```


#### Rebranchement de po2

Quand on refait un ping après avoir remis la liaison PO2,
la capture de trafic montre qu'il repasse bien par Po2. 

reprise du traffic sur po2 

![Capture à mettre](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/Capture_po2-reprise_traffic_test_span_DS2.PNG?raw=true)

## EIGRP v4

#### Coupures entre CORE et DISTRIBUTION

![coupuresEIGRP](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/fiabilit%C3%A9-eigrp4.png?raw=true)

#### Coupure couche CORE

##### Test à partir de centos-8

![COREcentos8](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_traceroute_centos8.png?raw=true)

```
[root@localhost ~]# traceroute 192.168.122.221
traceroute to 192.168.122.221 (192.168.122.221), 30 hops max, 60 byte packets
 1  10.192.40.253 (10.192.40.253)  15.048 ms  31.439 ms  32.344 ms
 2  10.3.2.3 (10.3.2.3)  9.529 ms  11.169 ms  11.322 ms
 3  10.1.2.1 (10.1.2.1)  16.853 ms  18.339 ms  18.635 ms
```

On coupe R1 et R3

```
[root@localhost ~]# traceroute 192.168.122.221
traceroute to 192.168.122.221 (192.168.122.221), 30 hops max, 60 byte packets
 1  10.192.40.253 (10.192.40.253)  13.530 ms  22.151 ms  31.372 ms
 2  10.2.4.2 (10.2.4.2)  9.818 ms  11.352 ms  11.618 ms
 3  10.1.1.1 (10.1.1.1)  14.868 ms  16.440 ms  17.884 ms
```

##### Test à partir de centos-1

On remarquera que après rebranchement, la route prise par les paquets ICMPs restent inchangés : Passage par DS2
``` 
[root@localhost ~]# traceroute 192.168.122.221
traceroute to 192.168.122.221 (192.168.122.221), 30 hops max, 60 byte packets
 1  10.192.10.253 (10.192.10.253)  7.535 ms  9.567 ms  12.422 ms
 2  10.3.1.3 (10.3.1.3)  7.691 ms  12.740 ms  12.737 ms
 3  10.1.2.1 (10.1.2.1)  13.665 ms  18.387 ms  19.927 ms
```

Il a fallu un reboot complet de la topologie pour que les paquets passent par DS1 :
```
[root@localhost ~]# traceroute 192.168.122.221
traceroute to 192.168.122.221 (192.168.122.221), 30 hops max, 60 byte packets
 1  10.192.10.252 (10.192.10.252)  3.813 ms  15.899 ms  19.478 ms
 2  10.3.3.3 (10.3.3.3)  8.486 ms  9.971 ms  10.189 ms
 3  10.1.2.1 (10.1.2.1)  11.595 ms  12.783 ms  13.644 ms
```

## HSRP Ipv6

Nous avons réaliser plusieurs essais de `ping -6` entre centos-1 (VLAN 16) et centos-8 (VLAN 46) en coupant po1 (shut) ou en arrétant DS1. Systématiquement, la liaison s'arrêtait et ne repartait pas.

Etherchannel et spanning tree étant de couche 2, le problème venait nécessairement de HSRP.

`show standy` nous dit que VLAN16 est toujours actif, même après le `shut`de po1 : ce n'est pas normal. 

Aussi, `ip -6 neighbor` (sur centos-1) nous indique que la recupération de l'adresse mac de la passerelle fe80::d0 est FAILED.

On réalise un reboot, puis `ip -6 neighbor` de nouveau. On obtient bien une nouvelle adresse mac.

Pourtant, le `ping -6` vers centos-8 ne fonctionne toujours pas.

La configuration est bonne, les paramètres, les passerelles et les adresses IPv6 des différentes VLAN ont été vérifié. Aucune incohérence n'a été trouvé sur DS1 et DS2.

Il faudrait mettre à l'épreuve le modèle via une topologie vierge afin de confirmer le problème HSRP en Ipv6.



