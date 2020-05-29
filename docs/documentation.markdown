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
#### [7. Configuration parefeu Cisco](#Parcisoc)
#### [8. Mise en place d'un VPN Ipsec Ipv4](#VPN4)
 - #####  [8.1 Configuration de FortiOS](#FortiOS)
 - #####  [8.2 Configuration sur CiscoIOS](#VPN4Cisco)
#### [9. Mise en place des services de surveillance](#Surveillance)
 - #####  [9.1 Syslog](#Syslog)
 - #####  [9.2 SNMP](#SNMP)
#### [10. Pour aller plus loin](#+loin)
 - ##### [10.1 Second Switchblock](#2switchblock)
 - ##### [10.2 VPN Ipsec Ipv6](#VPN6)
 - ##### [10.3 Focus Sécurité](#Secu)
#### [11. Annexes](#Annexes)
 - ##### [Fichiers de configuration](#config)


<a id="Topo"></a>
## Topologie

Une couche Core maillé de 3 Routeurs Cisco iOS (R1, R2, R3) 
Un switch block (4 VLANs utiles) composé de 2 périphérique de couche Distribution (DS1, DS2) et 2 périphérique de couche Acces (AS1, AS2)
Un maillage entre la couche Core et le switchblock
Un PC-distant avec un Fortigate (FortiGate VM64-KVM). 
Un PC pirate pour faire des tests. 
Un controleur afin d'implémenter la configuration Ansible. 
![Topologie](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/2020-05-28-Topologie.png?raw=true)

*Avec un deuxième switchblock*
![Topologie2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/Topologie_2_switchblocks.png)

<a id="Plan"></a>
## Plan d'adressage

[Adressage](https://github.com/reseau-2020/projet-three/blob/master/Plan%20d'adressage.md)

<a id="Ansible"></a>
## 3. Ansible

On a choisi le protocol EIGRP car, contrairement a OSPF, c'est un système autonome qui possède des routes secondaire et de multiples protocoles autre que IP. 
Et il est plus facile a configurer que OSPF. 


Les livres de jeux Ansible sont lancé depuis un poste "controller". 
```
[root@controller]# ansible-playbook /ansible-ccna-lab/playbooks/ccna.yml
```
Il faut adapter à notre configuration les fichiers du répertoire **/playbooks/inventories/projet3_main/host_var/**

Pour mettre a jour sur le controller les modifications on effetue un `git pull` dans le répertoire souhaité. 


<a id="Infra"></a>
## 4. Configuration des services d’infrastructures


<a id="DNS"></a>
###  4.1 DNS

**Copier-coller Daily recap Jour 5 : **

On remarquera que désormais le routage et la communication en IPv6 sera préféré par défault lors de l'utilisation de `ping`.

Sur R1, R2 et R3
```
R1(config)#ip name-server 1.1.1.1
R1(config)#ip domain lookup
R1(config)#ip dns server
R1(config)#end
```
conf t
ip name-server 1.1.1.1
ip domain lookup
ip dns server
end
wr

`ping ip www.google.com`fonctionne à 100% sur R1, R2, R3, DS1, DS2, AS1 et AS2


DNS fonctionne correctement sur la partie tripod. Mais pas sur les Vlans.


<a id="NTP"></a>
###  4.2 NTP

Network Time Protocol (NTP) est un protocole TCP/IP qui permet de synchroniser à travers le réseau l'horloge locale des ordinateurs sur une date et une heure de référence. 
Dans le cadre du projet nous avons ajusté l'heure de tous les périphériques Cisco selon les paramètres de la zone horaire et l'heure d'été français.

``
(config)#clock timezone GMT +1
``

``
(config)#clock summer-time FR recurring last SUN MAR 02:00 last SUN OCT 02:00
``

Deuxièmement, nous avons vérifié le statut NTP (désactivé par défaut sur le matériel Cisco) et paramétré un serveur NTP publique sur R1 : 

``
#show ntp status
``

``
%NTP is not enabled.
``

``
(config)#ntp server 3.fr.pool.ntp.org
``

``
(config)#ntp update-calendar
``

Ensuite, nous avons configuré une interface R1 (10.1.1.1) en tant que serveur NTP pour les autres périphériques Cisco de la topologie, sur R2, R3, AS1, AS2, DS1 et DS2 :

``
(config)#ntp server 10.1.1.1
``

``
(config)#ntp update-calendar
``

Nous avons constaté que les commutateurs AS1 et AS2 n'étaient pas synchronisés. Le problème a été résolu en désactivant le routage et en ajoutant une route par défaut sur ces deux périphériques, l'adresse utilisée a été la passerelle du sous-réseau virtuel de gestion (VLAN99).

``
(config)#no ip routing
``

``
(config)#ip default-gateway 10.192.1.252
``


<a id="Connect"></a>
## 5. Connectivité Ipv4 et Ipv6

Copier coller du Daily-Recap-Jour-5:

Connectivité IPv6 vers Internet

Dans un premier temps, afin de déployer la connectivité vers l’internet, nous avions configuré une adresse statique en IPV6 ipv6 route ::/0 g0/0 FE80::E53:21FF:FE38:5800 avec :

    FE80::E53:21FF:FE38:5800 , l’adresse link-local de notre passerelle vers l’internet
    g0/0 notre interface de sortie vers l’extérieur

Avec un show ipv6 routesur R2 et R3, nous nous sommes aperçu que la route ne se distribuait pas automatiquement. Nous avons donc déployer ipv6 route ::/0 g0/1 FE80::1 sur ces derniers.

Cela n’était pas nécessaire. Nous aurions pu forcer R1 à distribuer cette route statique : ipv6 router eigrp 1 redistribute static Par ailleurs, il est plus correct de limiter la route par défaut aux adresses publiques : ipv6 route 2000::/3 g0/0 FE80::E53:21FF:FE38:5800.

Nous avons fait le choix de ne pas déployer de LAN directement connectée sur R1. Par conséquent, aucune interface de R1 ne dispose d’adresse publique et donc ne peut pas joindre directement internet. Il est nécessaire de pingà partir des PCs des VLANs.

Toutefois, aucune connectivité vers l’internet ou entre les différents PC est observée.

show ipv6 route sur DS1 et DS2 nous apprend qu’il n’y a pas de route apprise par eigrp vers l’internet. show ipv6 eigrp neighborssur DS1 nous apprend qu’il ne voit pas R2 correctement. show run | b ipv6 eigrpsur R2 nous apprend que l’interface G0/4 n’est pas montée en eigrp.

Sur R2 : int g0/4 ipv6 eigrp 1 On remarque un log de eigrp nous indiquant qu’une nouvelle route a été apprise. Les ping ipv6 de PC1 vers PC8 et de PC1 vers l’internet fonctionnent correctement.

<a id="Test"></a>
## 6. Tests de fiabilité
Plusieurs essais ont été effectuée en tombant des liaisons (couches ACCESS, DISTRIBUTION et CORE) ou des periphériques entiers. 
La connectivité entre les PCs et la connectivité vers internet sont restaurés systématiquement après quelques secondes d’attente (3 paquets perdus).

<a id="Span"></a>
###  6.1 Spanning-Tree

Dans l'éventualité, lors d'un envoie de paquet, qu'une interface du chemin principale tombe, les switch trouveraient un chemin alternatif. 

C'est le cas dans cette exemple, lors d'un ping (IPV4 et IPV6) de PC2 a PC6, les interfaces G0/0 et G1/0 de DS2 tombe et le trafic trouve un autre chemin en ne perdant que 3 paquets. 
![Test spanning-tree sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20Spanning%20DS2.png?raw=true)
![Capture traffic de DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/Capture_po2-reprise_traffic_test_span_DS2.PNG?raw=true)


<a id="HSRP"></a>
###  6.2 HSRP

- #### IPv4
Un ping depuis le PC centos-1 vers l'Internet passe par AS1 puis DS1. En testant un crash de DS1, les paquets transmis utilise une autre passerelle et passe par DS2 pour atteindre l'Internet.  

![Test HSRP vers l'Internet](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS1%20routage%20internet.png?raw=true)

C'est le même principe qui est appliquer lors de communications entre deux PC. 
[Test HSRP sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS2.png?raw=true)

- #### IPv6
Notre topologie souffre lors de la mise à l’épreuve de HSRP en IPv6.

Il semblerait que l’adresse MAC de la passerelle virtuelle (fe80:d0) ne se mette pas à jour toute seule. Et meme après redémarrage des périphériques, les liaisons ne sont pas rétablit. 
Nous n’avons pas trouvé l’origine du problème. Notre topologie semble conforme au modèle suivit.


<a id="EIGRP"></a>
###  6.3 EIGRP

On trouve la route suivit pas le trafic depuis le périphérique avec la commande `trace route xxx.xxx.xxx.xxx` avec les XX l'adresse IP de destination.

Dans le cas d'un ping (IPV4 et IPV6) du PC centos-1 vers l'internet, on bloque la route principale, puis la route secondaire et la route tertiaire entre les couches Core et Distribution. Le routage s'adapte aux différentes routes.

![Test EIGRP 3 coupes](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/fiabilit%C3%A9-eigrp4.png?raw=true)

Ping (IPV4 et IPV6) de Centos-1 vers l'internet
[Test EIGRP de centos-1](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_fromcentos1.png?raw=true)

Ping (IPV4 et IPV6) de Centos-8 vers l'internet 
[Test EIGRP de centos-8](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_traceroute_centos8.png?raw=true)



<a id="Parcisoc"></a>
###  7 Pare-feu Cisco

### 7.1 Configuration globale

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


### 7.2 Configuration spécifiques

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

On appelle site distant un site séparé du siège de la société par une distance assez importante pour que les échanges nécessitent de passer par Internet. Pour simuler un site distant nous avons ajouté un ordinateur  connecté à un FortiOS, qui représente un pare-feu Fortigate, à notre topologie.

Afin d’établir une connexion sécurisée, un tunnel VPN a été implémenté entre R1 et le Forti3 (FortiOS). Un tunnel VPN est une connexion vpn sécurisée et cryptée entre un appareil et l’internet public. Avec une connexion cryptée à l’aide d’algorithmes robustes, toute communication reste privée et confidentielle.

<a id="FortiOS"></a>
### 8.1 Configuration VPN IPSEC IPv4 sur FortiOS

Dans notre topologie le FortiOS a les fonctions de router et filtrer le trafic entre l’Internet et le site distant, bien comme établir la connexion vpn avec le réseau principal. 

La configuration a été faite à partir de l’interface d’administration du pare-feu accédé avec un navigateur Web.

Nous avons configuré 3 interfaces dans Forti3 :
| Interface  | Identification  | Adresse ipv4 (dhcp)  |
|:-----:|:-----:|:-----:|
Port1 | LAN | 192.168.100.1 |
Port2 | Internet | 192.168.122.55 |
Port3 | Gestion | 192.168.122.56 |

Paramètres du tunnel IPsec :
-	Template : site-to-site
-	Remote device : Cisco
-	Passerelle : 192.168.122.221 (IP de l’interface extérieure du routeur Cisco)
-	Interface : Internet (port2)
-	Encryption : DES
-	Authentification : MD5

Règles pare-feu pour les trafics IPv4 :
| ID  | Nom  | De | Vers | Action |
|:-----:|:-----:|:-----:|:-----:|:-----:|
2 | Réseaux sociaux | lan(port1) | Internet(port2 |Rejecter|
3 | Internet | lan(port1) | Internet(port2 |Acepter|
4 | vpn_forti3-to-cisco_local | lan(port1) | forti3-to-cisco | Acepter|
5 | vpn_forti3-to-cisco_remote| forti3-to-cisco | lan(port1)|Acepter|
- | Refus implicite | Internet(port2) |all|Rejecter|

Nous avons constaté que l’interface d’administration nous permet d’attribuer seulement une adresse IPv6 par interface. Devant l’impossibilité d’attribution d’une adresse privée et une publique, nous avons décidé de faire l’implémentation d’un pare-feu sur un périphérique Cisco pour le filtrage des trafics en IPv6


<a id="VPN4Cisco"></a>
### 8.2 Configuration VPN IPSEC IPv4 sur CiscoIOS

Nous avons monté un VPN IPSEC en Ipv4 entre R1 et Forti3. Nous avions essayé dans un premier temps avec une encryption/authentification en `esp-des`/`esp-sha-hmac`. Toutefois, le tunnel ne se montait pas et restait inactif. Après quelques recherches, et l'aide précieuse de nos collègues du groupe 4. Il s'est avéré qu'un tunnel ne pouvait être monté entre un CiscoIOS et un FortIOS qu'avec `esp-des`/`esp-md5-hmac`. Nous avons donc réalisé les modifications nécessaires et le tunnel s'est monté.

- #### CRYPTO ISAKMP - phase 1 `esp-des`/`esp-md5`

Encryption : des
Authentification : md5
192.168.122.55 : IPv4 extérieur de Forti3

```
crypto isakmp policy 1
encr des
hash md5
authentication pre-share
group 5
lifetime 86400

crypto isakmp key cisco123 address 192.168.122.55
```

- #### TRANSFORM SET - phase 2 `esp-des`/`esp-md5-hmac`

```
crypto ipsec transform-set cisco-to-forti3-set esp-des esp-md5-hmac
```

- #### CRYPTO MAP

```
crypto map cisco-to-forti3 1 ipsec-isakmp
set peer 192.168.122.55
set transform-set cisco-to-forti3-set
match address crypto-acl

interface G0/0
crypto map cisco-to-forti3
```

- #### CRYPTO ACL

Configuration du traffic qui doit passer dans le tunnel (source/destination) :
```
ip access-list extended crypto-acl
permit ip 10.192.0.0 0.0.255.255 192.168.100.0 0.0.0.255
```

Configuration du traffic qui n'est pas traduit en NAT :
```
no ip access-list standard LANS
ip access-list extended LANS
5 deny ip 10.0.0.0 0.255.255.255 192.168.100.0 0.0.0.255    
10 permit ip 10.0.0.0 0.255.255.255 any

no ip access-list extended crypto-acl
ip access-list extended crypto-acl
permit ip 192.168.100.0 0.0.0.255 10.0.0.0 0.255.255.255 
permit ip 10.0.0.0 0.255.255.255 192.168.100.0 0.0.0.255
```

- #### MODIFICATION PARE-FEU

Des modifications sont nécessaire sur R1 afin de laisser passer les paquets isakmp sur le port 500 notamment. On crée une ACL spécifique au VPN, que l'on associe à une class-map `vpn-class`et que l'on ajoute à la règle de filtrage déjà existante `to-self-policy`.

```
ip access-list extended VPN
 permit udp any any eq isakmp
 permit ahp any any
 permit esp any any
 permit udp any any eq non500-isakmp

class-map type inspect match-any vpn-class
 match access-group name VPN

policy-map type inspect to-self-policy
 class type inspect vpn-class
  inspect
```



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

- Réaliser des scans du réseau à partir d'un PC pirate. Et potentiellement essayer d'accéder aux fichiers contenu sur un périphérique grâce à SSH par exemple.




- Améliorer les ACLs et policy-map des pare-feu en réduisant les blocs;

- Mettre en place des groupes d'utilisateurs et des accès restreints en fonction des groupes et des VLANs (sur R1 et FortiOS);
 Notamment avec des mots de passe à plus de 8 caractères, combinaison de lettres, chiffres et caractères spéciaux, ainsi qu'une expiration.

- Interdire l'accès à certaines URLs et cookies sur FortiOS;

![Policy Forti3](https://github.com/reseau-2020/projet-three/blob/master/Configurations/Policy%20Forti3.png)

- Mettre en place un serveur RADIUS afin de sécuriser les accès administratifs aux différents périphériques de la topologie;


- Sécurisation envisagables pour SNMP :
En choisissant judicieusement un nom de Communauté 
En configurant des SNMP View 
En activant des ACLs sur les Communautés et sur les interfaces 
En isolant ce trafic dans un VLAN contrôlé par des ACLs En activant SNMPv3

- Avoir des fichiers de sauvegarde
- UTM
- Antivirus sur les PC

<a id="Annexes"></a>
## 11. Annexes


<a id="config"></a>
- [Fichiers de configuration](https://github.com/reseau-2020/projet-three/tree/master/Ansible/playbooks/backup)
