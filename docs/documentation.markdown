---
layout: page
title: Documentation
permalink: /documentation/
---


# Projet 3 Consultant réseau

## Sommaire

#### [1. Topologie](#Topo)
#### [2. Plan d’adressage Ipv4/Ipv6](#Plan)
#### [3. Ansible](#Ansible)
#### [4. Configuration des services d’infrastructures](#Infra)
 - #####  [4.1 DNS](#DNS)
 - #####  [4.2 NTP](#NTP)
#### [5. Connectivité Ipv4 et Ipv6](#Connect)
#### [6. Tests de fiabilité](#Test)
 - #####  [6.1 Spanning-Tree](#Span)
 - #####  [6.2 HSRP](#HSRP)
 - #####  [6.3 EIGRP](#EIGRP)
#### [7. Configuration parefeu FortiOS et Cisco](#Pare-feu)
 - #####  [7.1 Site distant et FortiOS](#Forti)
 - #####  [7.2 Pare-feu Cisco](#Parcisoc)
#### [8. Mise en place d'un VPN Ipsec Ipv4](#VPN4)
#### [9. Mise en place des services de surveillance](#Surveillance)
 - #####  [9.1 Syslog](#Syslog)
 - #####  [9.2 SNMP](#SNMP)
#### [10. Pour aller plus loin](#+loin)
 - ##### [10.1 Second Switchblock](#2switchblock)
 - ##### [10.2 VPN Ipsec Ipv6](#VPN6)
 - ##### [10.3 Focus Sécurité](#Secu)
#### [11. Annexes](#Annexes)
 - ##### [11.1 Fichiers de configuration](#config)



## Groupe Projet

**Groupe**: Tayana/Sandrine/Maxime/Cyril


[Gantt](https://github.com/reseau-2020/projet-three/blob/master/Gantt_projet_3.xlsx)

![Planning](https://github.com/reseau-2020/projet-three/blob/master/2020-05-27-planning.PNG?raw=true)

[Scrum](https://github.com/reseau-2020/projet-three/projects/1)

[Blog](https://reseau-2020.github.io/projet-three/)

<a id="Topo"></a>
## Topologie

[Topologie](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/2020-05-28-Topologie.png?raw=true)

<a id="Plan"></a>
## Plan d'adressage

[Adressage](https://github.com/reseau-2020/projet-three/blob/master/Plan%20d'adressage.md)

<a id="Config"></a>
## Configuration des périphérique (via Ansible)
[Config](https://github.com/reseau-2020/projet-three/tree/master/Configurations)

## Choix de technologie de routage dynamique

```
EIGRP
```
[Choix EIGRP](https://github.com/reseau-2020/projet-three/blob/master/Tableau%20comparatif%20IRP.md)





