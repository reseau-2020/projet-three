---
layout: post
title:  "Daily Recap JOUR 07"
date:   2020-05-27
categories: welcome
---


## Réalisations de la journée :

 - Diagnostique du VPN IPSEC Ipv4
 - Mise en place d'un 2nd site distant et d'un VPn IPSEC Ipv6
 - Organiser les services de surveillances
   - [x] SNMP
 - Début de mise en place d'un 2nd switchblock
 - Mise en place de fichier de sauvegarde et restauration Ansible

## Débriefing

### VPN Ipv4

Le VPN Ipv4 ne fonctionnait pas : pas de `ping` de PC-distant vers VLAN10 ; pas de `ping` de R1 vers PCdistant.

Il manquait les règles de pare-feu `internet-lan` et `self-internet`.

```
zone-pair security self-internet source self destination internet
 service-policy type inspect to-self-policy

zone-pair security internet-lan source internet destination lan
 service-policy type inspect to-self-policy
```
### VPN Ipv6

Mise en place d'un routeur R4 avec un LAN avec adressage IPv4 et Ipv6,nat, dns, connectivité vers internet, eirgpv4 et v6 (Id 6.6.6.6). 
> 2001:470:C814:7006::/64
>
> FD00:470:C814:7006::/64
>
> FE80::cafe:7 sur interface externe de R4

Mise en place d'un VPN Ipv6 entre deux Cisco. Nous avions essayé entre 1 fortigate et 1 cisco mais cela semble très compliqué voir peut être impossible avec le fortiOS utilisé ici. L'usage de GUI pour ce faire sur FortiOs n'est pas possible, il aurait fallu utiliser CLI, sans être certain du résultat.

Nous avons utilisé 3des/sha pour le VPON ipv6 IPSEC et créé une nouvelle interface Tunnel6 de chaque côté avec une adresse IPv6 virtuelle.

Un premier problème est apparu :

```
Dropping udp session [2001:470:C814:3001::1]:500 [2001:470:C814:7006::1]:500 on zone-pair self-internet class class-default due to  DROP action found in policy-map with ip ident 11032
```
De nouveau un problème de pare-feu :
```
class-map type inspect match-any vpn-class
 match access-group name VPN6

ipv6 access-list VPN6
 permit icmp any any nd-na
 permit icmp any any nd-ns
 permit udp any any eq 500
 permit udp any eq 500 any
 permit udp any any eq isakmp
 permit ahp any any
 permit esp any any
 permit udp any any eq non500-isakmp
```
En effet, nous n'avions pas paramétré les ACLs en IPv6.

Le première a été résolu mais toutefois, il nous est toujours impossible de `ping` d'un côté ou de l'autre. Pourtant le tunnel semble être mis en place et fonctionnel :

``` 
show crypto isakmp sa
```
sur R1 :
```
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
192.168.122.221 192.168.122.55  QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA

 dst: 2001:470:C814:7006::1
 src: 2001:470:C814:3001::1
 state: QM_IDLE         conn-id:   1002 status: ACTIVE
```

**Suite à donner :**
Nous allons mettre de côté ce VPN pour nous consacrer sur la fiabilité et la sécurité de la partie fonctionnelle de notre topologie.
Il serait interessant pour les prochains diagnostique de faire `traceroute` et une étude plus appofondie des règles de pare-feu mis en place sur R1.
 

## Update docs

   - [x] Update Blog
    
## Programme du 28.05.2020
  
 - Mise à jour documentation
 - Diagnostiques et tests de fiabilité
 - Diagnostique et tests de sécurité
 - Mise en place d"un 2nd switchblock
  
