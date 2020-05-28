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

###VPN Ipv4

Le VPN Ipv4 ne fonctionnait pas : pas de `ping` de PC-distant vers VLAN10 ; pas de `ping` de R1 vers PCdistant.

Il manquait les règles de pare-feu `internet-lan` et `self-internet`.

```
zone-pair security self-internet source self destination internet
 service-policy type inspect to-self-policy

zone-pair security internet-lan source internet destination lan
 service-policy type inspect to-self-policy
```
### VPN Ipv6

Mise en place d'un VPN Ipv6 entre deux Cisco. Nous avions essayé entre 1 fortigate et 1 cisco mais cela semble très compliqué voir peut être impossible avec le fortiOS utilisé ici. L'usage de GUI pour ce faire sur FortiOs n'est pas possible, il aurait fallu utiliser CLI, sans être certain du résultat.

Nous avons utilisé 3des/sha pour le VPON ipv6 IPSEC.

Un premier problèmeest apparu :

 

## Update docs

   - [x] Update Blog
    
## Programme du 27.05.2020
  
 - Mise à jour documentation
 - Diagnostiques et tests de fiabilité
 - Diagnostique et tests de sécurité
 - Mise en place d"un 2nd switchblock
  
