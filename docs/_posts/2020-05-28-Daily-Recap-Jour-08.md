---
layout: post
title:  "Daily Recap JOUR 08"
date:   2020-05-28
categories: welcome
---


## Réalisations de la journée :

 - Tests de Fiabilité HSRP (IPV4 et Ipv6), etherchannel et spanning-tree
 - Mise en place d'un 2nd switchlock sur une copie de la topologie
 - Revision des services de surveillances
   - [x] SNMP
   - [x] Syslog

## Débriefing

### Tests de fiabilité IPV4

Tout fonctionne correctement. Plusieurs essais ont été effectuée en tombant des liaisons (couches ACCESS, DISTRIBUTION et CORE) ou des periphériques entiers.
La connectivité entre les PCs et la connectivité vers internet sont restaurés systématiquement après quelques secondes d'attente (3 paquets perdus).

### Tests de fiabilité IPV6

Notre topologie souffre lors de la mise à l'épreuve de HSRP en IPv6.

Il semblerait que l'adresse MAC de la passerelle virtuelle (`fe80:d0`) ne se mette pas à jour toute seule. Et meme après redémarrage des périphériques, les liaisons ne sont pas rétablit.
Nous n'avons pas trouvé l'origine du problème. Notre topologie semble conforme au modèle suivit.

### Syslog

Reprise de Syslog, et installation du serveur sur le PC-distant `192.168.100.2`. Nous avons configuré R1 et centos-1 en client syslog. Le traffic TCP 1514 et UDP 514 passe donc par le VPN. Il a fallu adapter le pare-feu de R1.
L'ensemble des logs sont consultables directement sur le PC-distant. Syslog est donc fonctionnel.


## Update docs

   - [x] Update Blog
   - [X] Update Scrum
   
    
## Programme du 29.05.2020
  
 - Mise à jour documentation
 - Préparation présentation
 - Mise à jour github
 - Etude de sécurité
  
