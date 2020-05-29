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
 
 On considère que le client, depuis son PC, veut surveiller son installation avec le service **Syslog**. 
On installe le serveur sur le PC-distant `yum -y install rsyslog` qui a pour adresse IP `192.168.100.2` 

Il faut modifier le fichier de configuration `vi /etc/rsyslog.conf` 
et décommenter la partie UDP et TCP syslog reception avec port UDP : 514 et port TCP : 1514.

Il faut redémarrer le système Syslog pour que les modifications soit prises en compte : `systemctl restart rsyslog`

Le traffic TCP 1514 et UDP 514 passe donc par le VPN.



Nous avons configuré centos-1 en client syslog :

Il faut installer Syslog sur le PC : `yum -y install rsyslog` et modifier le fichier de configuration : `vi /etc/rsyslog.conf` 
``` 
$template TmplAuthpriv, "/var/log/remote/auth/%HOSTNAME%/%PROGRAMNAME:::secpath--
replace%.log"
$template TmplMsg, "/var/log/remote/msg/%HOSTNAME%/%PROGRAMNAME:::secpath-replacc
e%.log"
# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514
# Provides TCP syslog reception
$ModLoad imtcp
# Adding this ruleset to process remote messages
$RuleSet remote1
authpriv.*   ?TmplAuthpriv
*.info;mail.none;authpriv.none;cron.none   ?TmplMsg
$RuleSet RSYSLOG_DefaultRuleset   #End the rule set by switching back to the deff
ault rule set
$InputTCPServerBindRuleset remote1  #Define a new input and bind it to the "remoo
te1" rule set
$InputTCPServerRun 1514
*.* @192.168.100.2:514
*.* @192.168.100.2:1514
```
Il faut redémarrer le système Syslog pour que les modifications soit prises en compte : `systemctl restart rsyslog`


&thinsp;
Nous avons également configuré R1 en client syslog :

En configuration terminal, on active le service Syslog avec `logging trap debugging` et on indique le server `logging 192.168.100.2`

Pour vérifier la configuratin de R1 : `show logging`
```
Trap logging: level debugging, 111 message lines logged
        Logging to 10.192.1.101
```

Il a fallu adapter le pare-feu de R1 pour laisser passer le trafic du pour 514 en UDP et 1514 en TCP : 
```
ip access-list extended SYSLOG
 permit udp any any eq 514
 permit udp any eq 514 any
 permit tcp any any eq 1514
 permit tcp any eq 1514 any
 deny udp any any
!
class-map type inspect match-any syslog-class
 match access-group name SYSLOG
!
policy-map type inspect to-self-policy
 class type inspect syslog-class
  pass
```

**L’ensemble des logs sont consultables directement sur le PC-distant. 
Syslog est donc fonctionnel.**


 - #####  [9.2 SNMP](#SNMP)
#### [10. Pour aller plus loin](#+loin)
 - ##### [10.1 Second Switchblock](#2switchblock)
 - ##### [10.2 VPN Ipsec Ipv6](#VPN6)
 - ##### [10.3 Focus Sécurité](#Secu)
#### [11. Annexes](#Annexes)
 - ##### [11.1 Fichiers de configuration](#config)



## Groupe Projet

**Groupe**: Tayana/Sandrine/Maxime/Cyril

## Planning

![Planning](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_planning/Planning.png)

[Gantt](https://github.com/reseau-2020/projet-three/blob/master/Gantt_projet_3.xlsx)

[Scrum](https://github.com/reseau-2020/projet-three/projects/1)

[Blog](https://reseau-2020.github.io/projet-three/)

<a id="Topo"></a>
## Topologie

[Topologie](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/2020-05-28-Topologie.png?raw=true)

<a id="Plan"></a>
## Plan d'adressage

[Adressage](https://github.com/reseau-2020/projet-three/blob/master/Plan%20d'adressage.md)

<a id="Ansible"></a>
## 3. Ansible


<a id="Infra"></a>
## 4. Configuration des services d’infrastructures


<a id="DNS"></a>
###  4.1 DNS


<a id="NTP"></a>
###  4.2 NTP]


<a id="Connect"></a>
## 5. Connectivité Ipv4 et Ipv6


<a id="Test"></a>
## 6. Tests de fiabilité


<a id="Span"></a>
###  6.1 Spanning-Tree


<a id="HSRP"></a>
###  6.2 HSRP


<a id="EIGRP"></a>
###  6.3 EIGRP


<a id="Pare-feu"></a>
## 7. Configuration parefeu FortiOS et Cisco


<a id="Forti"></a>
###  7.1 Site distant et FortiOS


<a id="Parcisoc"></a>
###  7.2 Pare-feu Cisco


<a id="VPN4"></a>
## 8. Mise en place d'un VPN Ipsec Ipv4


<a id="Surveillance"></a>
## 9. Mise en place des services de surveillance


<a id="Syslog"></a>
###  9.1 Syslog


<a id="SNMP"></a>
###  9.2 SNMP


<a id="+loin"></a>
## 10. Pour aller plus loin


<a id="2switchblock"></a>
### 10.1 Second Switchblock


<a id="VPN6"></a>
### 10.2 VPN Ipsec Ipv6


<a id="Secu"></a>
### 10.3 Focus Sécurité

<a id="Annexes"></a>
## 11. Annexes


<a id="config"></a>
### 11.1 Fichiers de configuration
