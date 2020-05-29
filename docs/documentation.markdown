---
layout: page
title: Documentation
permalink: /documentation/
---


# Documentation Technique

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


<a id="Topo"></a>
## Topologie

![Topologie](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/2020-05-28-Topologie.png?raw=true)

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
###  4.2 NTP


<a id="Connect"></a>
## 5. Connectivité Ipv4 et Ipv6


<a id="Test"></a>
## 6. Tests de fiabilité


<a id="Span"></a>
###  6.1 Spanning-Tree

Dans l'éventualité, lors d'un envoie de paquet, qu'une interface du chemin principale tombe, les switch trouveraient un chemin alternatif. 

C'est le cas dans cette exemple, lors d'un ping (IPV4 et IPV6) de PC2 a PC6, les interfaces G0/0 et G1/0 de DS2 tombe et le trafic trouve un autre chemin en ne perdant que 3 paquets. 
![Test spanning-tree sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20Spanning%20DS2.png?raw=true)
![Capture traffic de DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/Capture_po2-reprise_traffic_test_span_DS2.PNG?raw=true)


<a id="HSRP"></a>
###  6.2 HSRP

Un ping depuis le PC centos-1 vers l'Internet passe par AS1 puis DS1. En testant un crash de DS1, les paquets transmis utilise une autre passerelle et passe par DS2 pour atteindre l'Internet.  
![Test HSRP vers l'Internet](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS1%20routage%20internet.png?raw=true)

C'est le même principe qui est appliquer lors de communications entre deux PC. 
![Test HSRP sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS2.png?raw=true)

Il semblerait que l’adresse MAC de la passerelle virtuelle (fe80:d0) ne se mette pas à jour toute seule. Et meme après redémarrage des périphériques, les liaisons ne sont pas rétablit. 
Nous n’avons pas trouvé l’origine du problème. Notre topologie semble conforme au modèle suivit.


<a id="EIGRP"></a>
###  6.3 EIGRP

On trouve la route suivit pas le trafic depuis le périphérique avec la commande `trace route xxx.xxx.xxx.xxx` avec les XX l'adresse IP de destination.

Dans le cas d'un ping (IPV4 et IPV6) du PC centos-1 vers l'internet, on bloque la route principale, puis la route secondaire et la route tertiaire entre les couches Core et Distribution. Le routage s'adapte aux différentes routes. 
![Test EIGRP 3 coupes](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/fiabilit%C3%A9-eigrp4.png?raw=true)


Ping (IPV4 et IPV6) de Centos-1 vers l'internet
![Test EIGRP de centos-1](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_fromcentos1.png?raw=true)


Ping (IPV4 et IPV6) de Centos-8 vers l'internet 
![Test EIGRP de centos-8](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_traceroute_centos8.png?raw=true)


<a id="Pare-feu"></a>
## 7. Configuration parefeu FortiOS et Cisco


<a id="Forti"></a>
###  7.1 Site distant et FortiOS


<a id="Parcisoc"></a>
###  7.2 Pare-feu Cisco

### 7.2.1 Configuration globale

R1 fera office de pare-feu. C'est un routeur cisco. 
Cette première configuration permet de mettre en place le filtrage sortant de notre réseau vers l'Internet. 

- #### CLASS MAPS

Ici, on vient définir la `class-map` pour le traffic internet. Quels sont les protocoles ou les ACLs que l'on souhaite examiner ici ?
Le traffic sortant concernera les protocoles HTTP, HTTPS, DNS, ICMP, SSH.

```
class-map type inspect match-any internet-trafic-class
  match protocol http
  match protocol https
  match protocol dns
  match protocol icmp
  match protocol ssh
```

- #### POLICY MAPS

La configuration de la policy-map `internet-trafic-policy` permet de définir la règle de filtrage. On inspectera tout le trafic de la `class-map` précédemment définie. 

```
policy-map type inspect internet-trafic-policy
  class type inspect internet-trafic-class
   inspect
```

- #### ZONES ET INTERFACES

On crée les zones de sécurités que l'on vient associer aux interfaces de R1.

```
zone security lan
zone security internet
```
```
interface G0/0
 description Interface WAN
 zone-member security internet
interface G0/2
 description Interface vers R2
 zone-member security lan
interface G0/3
 description Interface vers R3
 zone-member security lan
 ```

- #### ZONE PAIR

Enfin, on vient affecter une zone à notre policy-map. Ici la zone allant de la LAN à INTERNET.

```
zone-pair security lan-internet source lan destination internet
  service-policy type inspect internet-trafic-policy
```


### 7.2.2 Configuration spécifiques

Il a été nécessaire de rajouter des ACLs pour les protocoles spécifiques suivant : SSH, DNS, DHCP, NTP, SYSLOG.

En effet, afin de pouvoir utiliser certains services, il est nécessaire d'autoriser le traffic de ces protocoles sur certains ports.

L'ensemble de ce code n'a pas été mis en place d'un seul bloc. Nous sommes aperçu au fur et à mesure de problèmes avec le filtrage du pare-feu. Par exemple avec NTP, le log suivant est apparu sur R1 lors de la mise en place de ce dernier sur notre topologie :
```
Dropping udp session 188.165.250.19:123 192.168.122.221:123 on zone-pair internet-self class class-default due to  DROP action found in policy-map with ip ident 23648
```

De la même façon que précédemment, nous avons mis en place les ACLs, les class-maps, les policy-maps et les zones-pair.

```
ip access-list extended SSH
 permit tcp any any eq 22
 deny tcp any any
ip access-list extended DHCP
 permit udp any eq 67 any eq 68
 permit udp any eq 68 any eq 67
 deny udp any any
ip access-list extended DNS
 permit udp any any eq 53
 permit udp any eq 53 any
 deny udp any any
ip access-list extended NTP
 permit udp any any eq 123
 permit udp any eq 123 any
 deny udp any any
ip access-list extended SYSLOG
 permit udp any any eq 514
 permit udp any eq 514 any
 permit tcp any any eq 1514
 permit tcp any eq 1514 any
 deny udp any any
```

```
class-map type inspect match-any remote-access-class
 match access-group name SSH
class-map type inspect match-any icmp-class
 match protocol icmp
class-map type inspect match-any dhcp-class
 match access-group name DHCP
class-map type inspect match-any dns-class
 match access-group name DNS
class-map type inspect match-any ntp-class
 match access-group name NTP
class-map type inspect match-any syslog-class
 match access-group name SYSLOG
```

```
policy-map type inspect to-self-policy
 class type inspect remote-access-class
  pass
 class type inspect icmp-class
  inspect
 class class-default
  drop log
 class type inspect dhcp-class
  pass
 class type inspect dns-class
  pass
 class type inspect ntp-class
  pass
 class type inspect syslog-class
  pass
```

```
zone-pair security internet-self source internet destination self
 service-policy type inspect to-self-policy
zone-pair security self-internet source self destination internet
 service-policy type inspect to-self-policy
zone-pair security internet-lan source internet destination lan
 service-policy type inspect to-self-policy
```

- #### VERIFICATIONS

L'ensemble de la configuration pourra être inspecter avec les commandes suivantes.

```
show zone security
show zone-pair security
show policy-map type insp zone-pair
```

Aussi, les commandes suivantes nous a permis de vérifier le bon fonctionnement du pare-feu R1 :

Sur centos-1 (VLAN10/16) :
```
ping www.google.com
ping -6 www.google.com
curl www.test.tf
```

Aussi, nous avons ajouter un PC-pirate afin de nous assurer des ports accessibles depuis Internet.
192.168.122.221 étant l'adresse externe de R1 sur G0/0.

![Image PC Pirate](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_divers/pirate.PNG?raw=true)

```
nmap 192.168.122.221
Starting Nmap 6.40 ( http://nmap.org ) at 2020-05-26 11:51 CEST
Nmap scan report for 192.168.122.221
Host is up (0.97s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 0C:D0:27:38:D0:00 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 90.78 seconds
```

On remarquera que seul SSH est disponible.



<a id="VPN4"></a>
## 8. Mise en place d'un VPN Ipsec Ipv4


<a id="Surveillance"></a>
## 9. Mise en place des services de surveillance


<a id="Syslog"></a>
###  9.1 Syslog

 On considère que le client, depuis son PC, veut surveiller son installation avec le service **Syslog**. 
On installe le serveur sur le PC-distant `yum -y install rsyslog` qui a pour adresse IP `192.168.100.2` 

Il faut modifier le fichier de configuration `vi /etc/rsyslog.conf` 
et décommenter la partie UDP et TCP syslog reception avec port UDP : 514 et port TCP : 1514.

Il faut redémarrer le système Syslog pour que les modifications soit prises en compte : `systemctl restart rsyslog`

Le traffic TCP 1514 et UDP 514 passe donc par le VPN.

#### Nous avons configuré centos-1 en client syslog :

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

#### Nous avons également configuré R1 en client syslog :

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


<a id="SNMP"></a>
###  9.2 SNMP

SNMP, tout comme Syslog, permet de collecter des informations de surveillance sur un server central provenant d'appareils distants. 
Mais SNMP permet de faire une peu de gestion et est plus sécurisé que Syslog. 

Toutefois, nous ne l'avons pas implémenter. Pour ce faire, il aurai fallut suivre les instructions suivantes :


#### Pour configurer les périphériques Cisco

```
snmp-server community Projet3SNMP RO
snmp-server enable traps
snmp-server host 10.192.10.102 private   # L'adresse du server
``` 

#### Securisation envisageables

SNMPv2c se sécurise :

 - En choisissant judicieusement un nom de Communauté
 - En configurant des SNMP View
 - En activant des ACLs sur les Communautés et sur les interfaces
 - En isolant ce trafic dans un VLAN contrôlé par des ACLs
 - En activant SNMPv3

#### REMOTE

Installation des outils Net-SNMP3 sous Rehat Enterprise Linux ou Centos (RHEL7) :

yum install net-snmp-utils

Usage :

snmpwalk -v2c -c <nom de la communauté> <périphérique à gérer>



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
