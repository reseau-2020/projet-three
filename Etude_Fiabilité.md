
# Etude Fiabilité

PC2 ip : 10.192.20.102/24 

PC6 ip : 10.192.20.2/24

DS2 est la route principale pour le VLAN20 (que on regarde avec la commande "show spanning-tree VLAN20" sur DS2)

## HSRP

#### Coupure de DS2

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

reprise du traffic sur po2 [Capture à mettre]

## EIGRP v4

#### Coupures entre CORE et DISTRIBUTION

![coupuresEIGRP](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/fiabilit%C3%A9-eigrp4.png?raw=true)

#### Coupure couche CORE

##### Test à partir de centos-1
On aurait pu penser au résultat suivant :
![COREcentos1](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_fromcentos1.png?raw=true)
Toutefois, un `traceroute` (sans aucune coupure) a permis de s'apercevoir que la route par défaut de centos vers internet était tout autre :
``` 
[root@localhost ~]# traceroute 192.168.122.221
traceroute to 192.168.122.221 (192.168.122.221), 30 hops max, 60 byte packets
 1  10.192.10.253 (10.192.10.253)  7.535 ms  9.567 ms  12.422 ms
 2  10.3.1.3 (10.3.1.3)  7.691 ms  12.740 ms  12.737 ms
 3  10.1.2.1 (10.1.2.1)  13.665 ms  18.387 ms  19.927 ms
```

##### Test à partir de centos-8

![COREcentos8](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_traceroute_centos8.png?raw=true)
