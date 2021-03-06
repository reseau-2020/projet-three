---
layout: page
title: Documentation
permalink: /documentation/
---


---

## Sommaire

- #### [1. Topologie](#1)
- #### [2. Plan d’adressage Ipv4/Ipv6](#2)
- #### [3. Ansible](#3)
  - ##### [3.1 Activation des interfaces du tripod](#31)
  - ##### [3.2 Sauvegarder une configuration](#32)
- #### [4. Configuration des services d’infrastructures](#4)
  - #####  [4.1 DNS](#41)
  - #####  [4.2 NTP](#42)
- #### [5. Connectivité Ipv4 et Ipv6](#5)
  - #####  [5.1 Connectivité IPv4](#51)
  - #####  [5.2 Connectivité IPv6 vers Internet](#52)
- #### [6. Tests de fiabilité](#6)
  - #####  [6.1 Spanning-Tree](#61)
  - #####  [6.2 HSRP](#62)
  - #####  [6.3 EIGRP](#63)
- #### [7. Pare-feu Cisco](#7)
  - ##### [7.1 Configuration globale](#71)
  - ##### [7.2 Configuration spécifiques](#72)
- #### [8. Mise en place d'un VPN Ipsec Ipv4](#8)
  - #####  [8.1 Configuration de FortiOS](#81)
  - #####  [8.2 Configuration sur CiscoIOS](#82)
- #### [9. Mise en place des services de surveillance](#9)
  - #####  [9.1 Syslog](#91)
  - #####  [9.2 SNMP](#92)
- #### [10. Pour aller plus loin](#10)
  - ##### [10.1 Second Switchblock](#101)
  - ##### [10.2 VPN IPsec Ipv6](#102)
  - ##### [10.3 Focus Sécurité](#103)
- #### [11. Annexes](#11)


---

<a id="1"></a>
## 1. Topologie

![Topologie](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/Topo_autre.png?raw=true)

Notre topologie est constitué de : 

<table>
  <thead>
        <tr>
            <th>Ensemble</th>
            <th>Ressources</th>
        </tr>
  </thead>
  <tbody>
        <tr>
            <td>Site Controlleur</td>
            <td><p>Un CentOS Linux 7.5</p><p>Un switch Ethernet</p></td>
        </tr>
        <tr>
            <td>Couche Core</td>
            <td><p> Trois routeurs R1, R2 et R3 Cisco IOS Software, IOSv Software (VIOS-ADVENTERPRISEK9-M),  Version 15.7(3)M3 </p></td>
        </tr>
        <tr>
            <td>Couche Distribution</td>
            <td><p> Deux switchs de distribution DS1 et DS2 Cisco IOS Software, vios_l2 Software (vios_l2-ADVENTERPRISEK9-M), Experimental Version 15.2(20170321:233949) </p></td>
        </tr>
        <tr>
            <td>Couche Access</td>
            <td><p> Deux switchs AS1 et AS2 Cisco IOS Software, vios_l2 Software (vios_l2-ADVENTERPRISEK9-M) Experimental Version 15.2(20170321:233949)</p><p> 8 postes de travail VPCS </p><p> 2 postes de travail CentOS Linux 7.5 </p></td>
        </tr>
        <tr>
            <td>Site Internet</td>
            <td><p> Un switch Ethernet </p><p> Un nuage NAT utiliser comme router par GNS3 pour atteindre l'Internet </p><p> Une station pirate CentOS Linux 7.5 </p></td>
        </tr>
        <tr>
            <td>Site Forti3</td>
            <td><p> Pare-feu fortinet : Model name: FortiGate-VM64-KVM </p><p> Un PC-distant CentOS Linux 7.5 </p></td>
        </tr>
        <tr>
            <td>Site R4</td>
            <td><p> Un routeur Cisco IOS Software, IOSv Software (VIOS-ADVENTERPRISEK9-M), Version 15.6(2)T </p><p> Un switch Ethernet </p><p> Un PC-distant CentOS Linux 7.5 </p></td>
        </tr>
  </tbody>
</table>



*Une seconde topologie avec un deuxième switchblock est disponible*
![Topologie2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/Topologie_2_switchblocks.png?raw=true)


---

<a id="2"></a>
## 2. Plan d'adressage

Le plan d'adressage IP de la topologie est disponible en cliquant sur le lien : [Adressage](https://github.com/reseau-2020/projet-three/blob/master/Plan%20d'adressage.md)

---

<a id="3"></a>
## 3. Ansible

On a choisi le protocole EIGRP car, contrairement à OSPF, c'est un système autonome qui possède des routes secondaire et de multiples protocoles autre qu'IP. 
Et il est plus facile à configurer et plus rapide qu'OSPF. 


Les livres de jeux Ansible sont lancé depuis un poste "Controller". 
```
[root@controller]# ansible-playbook /ansible-ccna-lab/playbooks/ccna.yml
```
Il faut adapter à notre configuration les fichiers du répertoire **/playbooks/inventories/projet3_main/host_var/**

Pour mettre à jour sur le Controller les modifications on effectue un `git pull` dans le répertoire souhaité. 

Les parties suivantes démonstrent le déroulement de l'éxecution d’un code Ansible de bout en bout pour deux fonctionnalités.

<a id="31"></a>
### 3.1 Activation des interfaces du tripod

#### [projet3_main.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/projet3_main.yml)

Ce playbook configure une topologie à partir de deux autres livres de jeux en les important :
  - [switchblock.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/switchblock.yml)
  - [tripod.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/tripod.yml)


```
---
# projet3_main.yml
# Playbook to configure CCNA "switchblock" and "tripod" topologies
- import_playbook: switchblock.yml
- import_playbook: tripod.yml
```

La fonction "import_playbook" permet d’executer les jeux d’un autre livre en l’invoquant par son nom.

Ici : «import_playbook: tripod.yml »

```
# tripod.yml
- hosts: core
  gather_facts: False
  roles:
    - role: ios_common
    - role: ios_interface
    - role: ios_ipv4
    - role: ios_ipv6
    - role: ios_ipv4-routing
    - role: ios_ipv6-routing
    - role: ios_fhrp
    - role: ios_rip
      when: '"rip" in ipv4.routing'
    - role: ios_eigrp4
      when: '"eigrp" in ipv4.routing'
    - role: ios_ospfv2
      when: '"ospf" in ipv4.routing'
    - role: ios_eigrp6
      when:
        - '"eigrp" in ipv6.routing'
    - role: ios_ospfv3
      when:
        - '"ospf" in ipv6.routing'
    - role: ios_recursive-dns-server
    - role: ios_dhcp-server
- hosts: R1
  gather_facts: False
  roles:
    - role: ios_nat
- hosts: core
  gather_facts: False
  roles:
    - role: ios_write
```

Ce livre de jeux invoque une série de rôles qui serviront à configurer la topologie au fur et à mesure de leurs éxecutions. 
Le premier élément « hosts » permet de définir une variable ou un groupe de variables contenues dans un autre fichier. Les instances de ce fichier seront utilisées dans l’éxecution des rôles.

Ici : « hosts: core » appelle le groupe de variables "[core]" qui définissent le tripod. En voici la structure :

#### [hosts:](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/inventories/projet3_main/hosts)

```
[core]
R1
R2
R3
```

Voici un exemple de fichier routeur utilisé dans ce rôle : 

#### [R2](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/inventories/projet3_main/host_vars/R2)

L’élément « gather_facts » accepte une valeur booléenne. Elle permet de récupérer des informations concernant l’execution des rôles.

"Roles" définit une liste de role à executer dans un ordre décidé. Un rôle appelle un autre fichier playbook et en exécutera les tâches. L’élément rôle peut prendre des paramètres supplémentaires (ex : « where: ‘« ospf » in ipv4.routing’).

Ici, nous allons analyser : « role: ios_interface ». Le fichier « main » est appelé en premier lors de l’execution du code. 


#### [main.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/roles/ios_interface/tasks/main.yml)

```
---
- import_tasks: enable_interfaces.yml
  when: ansible_network_os == 'ios'
  tags:
    - interface
    - test
```

Il importe le livre de jeux « enable_interfaces.yml » à condition que la variable « ansible_network_os » corresponde à la valeur « ios ». Cette variable se situe dans le fichier « hosts » indiqué plus haut.

#### [enable_interfaces.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/roles/ios_interface/tasks/enable_interfaces.yml)

Dans ce livre de jeux, deux rôles figurent. Nous allons étudier celui qui permet d'activer les interfaces qui ne sont pas utilisées comme interface 'stub'.

```
- name: enable interface
  ios_interfaces:
    config:
      - name: "{{ item.id }}"
        enabled: True
        description: "{{ item.description }}"
  loop: "{{ interfaces }}"
  when:
    - item.stub is not defined
    - item.id is defined
  tags:
    - interface
 ```
 
«ios_interfaces» est une fonctionnalité d’Ansible qui permet, dans notre cas, d’agir sur les interfaces des routeurs du tripod. 

L’élément « config » permet de définir des options pour les interfaces. Le terme ‘item’ désigne l’instance de routeur en cours de traitement du code.

Ici il s’agit notamment d’activer les interface via l’option « enabled: True » à condition que ce ne soit pas une interface stub et que l’interface puisse être identifiée (ex : « GigabitEthernet0/0 »). 

Afin de couvrir toutes les interfaces d’un routeur, on utilise la fonction « loop » d’Ansible sur la variable "interfaces".


---
<a id="32"></a>
### 3.2 Sauvegarder une configuration

Ce livre de jeux permet, lors de son exécution, de sauvegarder les configurations de matériel sur une topologie.

#### [save.yml](https://github.com/reseau-2020/projet-three/blob/master/Ansible/playbooks/save.yml)

```
---
- name: BACKUP CONFIGURATIONS
  hosts: cisco
  connection: network_cli
  gather_facts: no
  
  tasks:
    - name: BACKUP THE CONFIG
      ios_config:
        backup: yes
      register: config_output
```

Il utilise le paramètre « backup » du module  'Ansible ios_config' afin de copier les fichiers d’une configuration courante et d’en faire la copie dans un dossier de sauvegarde local « backup/ ».

Le fichier de configuration enregistré de cette manière est encapsulé dans la variable « register: config_output ». L’intérêt de retenir cette variable est de pouvoir modifier le fichier traité.

La modification du nom du fichier est nécessaire afin qu’il corresponde au nom de la machine enregistrée dans la topologie. Le but étant de pouvoir écrasé ce fichier lors d’un rétablissement d’une sauvegarde de la configuration.

```
 - name: RENAME BACKUP
      copy:
        src: "{{config_output.backup_path}}"
        dest: « ./backup/{{inventory_hostname}}.cfg"
```

Remarque : on utilise la variable config_output citée plus haut pour en copier le contenu en modifiant le nom et ajouter l’extension « .cfg ».

Les modifications suivantes servent à supprimer les lignes de code parasites qui ne permettraient pas de restaurer un fichier de configuration. Ces lignes sont les suivantes :

```
Building configuration...
Current configuration with default configurations exposed : 393416 bytes
```

On utilisera le module « lineinlife » qui permet d’éditer le contenu des lignes de texte.
Le paramètre « state: absent » permet d’enlever la ligne.

```
    - name: REMOVE NON CONFIG LINES
      lineinfile:
        path: "./backup/{{inventory_hostname}}.cfg"
        line: "Building configuration..."
        state: absent

```

Le paramètre regex désigne ici : tous les contenus commençant par « current configuration » et ce qui suit.

```
    - name: REMOVE NON CONFIG LINES - REGEXP
      lineinfile:
        path: "./backup/{{inventory_hostname}}.cfg"
        regexp: 'Current configuration.*'
        state: absent
```

Maintenant que le fichier de configuration est sauvegardé dans un dossier à part, il pourra être déployé à nouveau en cas de nécessité de revenir à une version antérieur de la topologie.

Cependant, le fichier est sauvegardé en local sur la machine. Si nous voulons mettre à jour notre git, il suffit d'éxecuter ensuite les commandes suivantes :

```
git config --global user.email « example@email. com»
git config --global user.name "usernameGitHub"
git add backup
git commit -a -m "update"
git push
```

<a id="4"></a>
## 4. Configuration des services d’infrastructures

  
  
<a id="41"></a>
###  4.1 DNS

On remarquera que désormais le routage et la communication en IPv6 sera préféré par défaut lors de l'utilisation de `ping`.

Sur R1, R2 et R3
```
R1#conf t
R1(config)#ip name-server 1.1.1.1
R1(config)#ip domain lookup
R1(config)#ip dns server
R1(config)#end
R1#wr
```

`ping ip www.google.com`fonctionne à 100% sur R1, R2, R3, DS1, DS2, AS1 et AS2


DNS fonctionne correctement sur la partie tripod. 

Aussi, nous avons remarqué que `dns-server` était mal paramétré sur les VLANs (DS1 et DS2), sûrement dû à une mauvaise manipulation sur les fichiers Ansible. 

Nous avons donc rajouté manuellement sur DS1 et DS2 :

```
ip dhcp pool VLAN10
 dns-server 1.1.1.1
ip dhcp pool VLAN20
 dns-server 1.1.1.1
ip dhcp pool VLAN30
 dns-server 1.1.1.1
ip dhcp pool VLAN40
 dns-server 1.1.1.1
 ```
 
 Un ping en IPv4 et IPv6 des PCs vers `www.google.com` nous a rassuré sur son fonctionnement.

<a id="42"></a>
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

`#show ntp status`

`%NTP is not enabled`

`(config)#ntp server 3.fr.pool.ntp.org`

`(config)#ntp update-calendar`

Ensuite, nous avons configuré une interface R1 (10.1.1.1) en tant que serveur NTP pour les autres périphériques Cisco de la topologie, sur R2, R3, AS1, AS2, DS1 et DS2 :

`(config)#ntp server 10.1.1.1`

`(config)#ntp update-calendar`

Nous avons constaté que les commutateurs AS1 et AS2 n'étaient pas synchronisés. Le problème a été résolu en désactivant le routage et en ajoutant une route par défaut sur ces deux périphériques, l'adresse utilisée a été la passerelle du sous-réseau virtuel de gestion (VLAN99).

`(config)#no ip routing`

`(config)#ip default-gateway 10.192.1.252`


---
  
<a id="5"></a>
## 5. Connectivité Ipv4 et Ipv6

<a id="51"></a>
### 5.1 Connectivité IPv4

Des `ping` entre les différentes VLANs, ainsi que vers `www.test.tf`, `1.1.1.1` et `www.google.com` ont permis de vérifier le bon fonctionnement de notre topologie en terme de connectivité.

<a id="52"></a>
### 5.2 Connectivité IPv6 vers Internet

Dans un premier temps, afin de déployer la connectivité vers l’internet, nous avions configuré une adresse statique en IPV6, `ipv6 route ::/0 g0/0 FE80::E53:21FF:FE38:5800` avec :

    FE80::E53:21FF:FE38:5800 , l’adresse link-local de notre passerelle vers l’internet
    g0/0 notre interface de sortie vers l’extérieur

Avec un `show ipv6 route` sur R2 et R3, nous nous sommes aperçu que la route ne se distribuait pas automatiquement. Nous avons donc déployé `ipv6 route ::/0 g0/1 FE80::1` sur ces derniers.

Cela n’était pas nécessaire. Nous aurions pu forcer R1 à distribuer cette route statique : `ipv6 router eigrp 1 redistribute static` Par ailleurs, il est plus correct de limiter la route par défaut aux adresses publiques : `ipv6 route 2000::/3 g0/0 FE80::E53:21FF:FE38:5800`.

Nous avons fait le choix de ne pas déployer de LAN directement connectée sur R1. Par conséquent, aucune interface de R1 ne dispose d’adresse publique et donc ne peut pas joindre directement internet. Il est nécessaire de ping à partir des PCs des VLANs.

Toutefois, aucune connectivité vers l’internet ou entre les différents PC n'est observée.

`show ipv6 route` sur DS1 et DS2 nous apprend qu’il n’y a pas de route apprise par EIGRP vers l’internet. `show ipv6 eigrp neighbors` sur DS1 nous apprend qu’il ne voit pas R2 correctement. `show run | b ipv6 eigrp` sur R2 nous apprend que l’interface G0/4 n’est pas montée en EIGRP.

Sur R2 : `int g0/4 ipv6 eigrp 1` On remarque un log de EIGRP nous indiquant qu’une nouvelle route a été apprise. Les ping ipv6 de PC1 vers PC8 et de PC1 vers l’internet fonctionnent correctement.

---
  
<a id="6"></a>
## 6. Tests de fiabilité


Plusieurs essais ont été effectués en tombant des liaisons (couches ACCESS, DISTRIBUTION et CORE) ou des périphériques entiers. 
La connectivité entre les PCs et la connectivité vers internet sont restaurés systématiquement après quelques secondes d’attente (3 paquets perdus).

<a id="61"></a>
###  6.1 Spanning-Tree

Dans l'éventualité, lors d'un envoie de paquet, qu'une interface du chemin principale tombe, les switch trouveraient un chemin alternatif. 

C'est le cas dans cet exemple, lors d'un ping (IPV4 et IPV6) de PC2 a PC6, les interfaces G0/0 et G1/0 de DS2 tombe et le trafic trouve un autre chemin en ne perdant que 3 paquets. 
![Test spanning-tree sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20Spanning%20DS2.png?raw=true)
![Capture trafic de DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/Capture_po2-reprise_traffic_test_span_DS2.PNG?raw=true)


<a id="62"></a>
###  6.2 HSRP

- #### IPv4
Un ping depuis le PC centos-1 vers l'Internet passe par AS1 puis DS1. En testant un crash de DS1, les paquets transmis utilisent une autre passerelle et passe par DS2 pour atteindre l'Internet.  

![Test HSRP vers l'Internet](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS1%20routage%20internet.png?raw=true)

C'est le même principe qui est appliqué lors de communications entre deux PC. 
[Test HSRP sur DS2](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/test%20HSRP%20DS2.png?raw=true)

- #### IPv6
Notre topologie souffre lors de la mise à l’épreuve de HSRP en IPv6.

Il semblerait que l’adresse MAC de la passerelle virtuelle (fe80:d0) ne se mette pas à jour toute seule. Et même après redémarrage des périphériques, les liaisons ne sont pas rétablit. 
Nous n’avons pas trouvé l’origine du problème. Notre topologie semble conforme au modèle suivit.


<a id="63"></a>
###  6.3 EIGRP

On trouve la route suivit pas le trafic depuis le périphérique avec la commande `trace route xxx.xxx.xxx.xxx` avec les XX l'adresse IP de destination.

Dans le cas d'un ping (IPV4 et IPV6) du PC centos-1 vers l'internet, on bloque la route principale, puis la route secondaire et la route tertiaire entre les couches Core et Distribution. Le routage s'adapte aux différentes routes.

![Test EIGRP 3 coupes](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/fiabilit%C3%A9-eigrp4.png?raw=true)

Ping (IPV4 et IPV6) de Centos-1 vers l'internet
[Test EIGRP de centos-1](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_fromcentos1.png?raw=true)

Ping (IPV4 et IPV6) de Centos-8 vers l'internet 
[Test EIGRP de centos-8](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_fiabilite/testeigrp_traceroute_centos8.png?raw=true)


---

<a id="7"></a>
##  7. Pare-feu Cisco

<a id="71"></a>
### 7.1 Configuration globale

R1 fera office de pare-feu. C'est un routeur Cisco. 
Cette première configuration permet de mettre en place le filtrage sortant de notre réseau vers l'Internet. 

- #### CLASS MAPS

Ici, on vient définir la `class-map` pour le trafic internet. Quels sont les protocoles ou les ACLs que l'on souhaite examiner ici ?
Le trafic sortant concernera les protocoles HTTP, HTTPS, DNS, ICMP, SSH.

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

<a id="72"></a>
### 7.2 Configuration spécifiques

Il a été nécessaire de rajouter des ACLs pour les protocoles spécifiques suivant : SSH, DNS, DHCP, NTP, SYSLOG.

En effet, afin de pouvoir utiliser certains services, il est nécessaire d'autoriser le trafic de ces protocoles sur certains ports.

L'ensemble de ce code n'a pas été mis en place d'un seul bloc. Nous sommes aperçus au fur et à mesure de problèmes avec le filtrage du pare-feu. Par exemple avec NTP, le log suivant est apparu sur R1 lors de la mise en place de ce dernier sur notre topologie :
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

L'ensemble de la configuration pourra être inspecté avec les commandes suivantes.

```
show zone security
show zone-pair security
show policy-map type insp zone-pair
```

Aussi, les commandes suivantes nous ont permis de vérifier le bon fonctionnement du pare-feu R1 :

Sur centos-1 (VLAN10/16) :
```
ping www.google.com
ping -6 www.google.com
curl www.test.tf
```

Aussi, nous avons ajouté un PC-pirate afin de nous assurer des ports accessibles depuis Internet.
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


---

<a id="8"></a>
## 8. Mise en place d'un VPN IPsec Ipv4

On appelle site distant un site séparé du siège de la société par une distance assez importante pour que les échanges nécessitent de passer par Internet. Pour simuler un site distant nous avons ajouté un ordinateur  connecté à un FortiOS, qui représente un pare-feu Fortigate, à notre topologie.

Afin d’établir une connexion sécurisée, un tunnel VPN a été implémenté entre R1 et le Forti3 (FortiOS). Un tunnel VPN est une connexion VPN sécurisée et cryptée entre un appareil et l’internet public. Avec une connexion cryptée à l’aide d’algorithmes robustes, toute communication reste privée et confidentielle.

<a id="81"></a>
### 8.1 Configuration VPN IPSEC IPv4 sur FortiOS

Dans notre topologie le FortiOS a les fonctions de router et filtrer le trafic entre l’Internet et le site distant, bien comme établir la connexion VPN avec le réseau principal. 

La configuration a été faite à partir de l’interface d’administration du pare-feu accédée avec un navigateur Web.

Nous avons configuré 3 interfaces dans Forti3 :
| Interface  | Identification  | Adresse ipv4 (DHCP)  |
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
2 | Réseaux sociaux | lan(port1) | Internet(port2) |Rejecter|
3 | Internet | lan(port1) | Internet(port2) |Acepter|
4 | vpn_forti3-to-cisco_local | lan(port1) | forti3-to-cisco | Acepter|
5 | vpn_forti3-to-cisco_remote| forti3-to-cisco | lan(port1)|Acepter|
0 | Refus implicite | any | any |Rejecter|

Nous avons constaté que l’interface d’administration nous permet d’attribuer seulement une adresse IPv6 par interface. Devant l’impossibilité d’attribution d’une adresse privée et une publique, nous avons décidé de faire l’implémentation d’un pare-feu sur un périphérique Cisco pour le filtrage des trafics en IPv6


<a id="82"></a>
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

Configuration du trafic qui doit passer dans le tunnel (source/destination) :
```
ip access-list extended crypto-acl
permit ip 10.192.0.0 0.0.255.255 192.168.100.0 0.0.0.255
```

Configuration du trafic qui n'est pas traduit en NAT :
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

Des modifications sont nécessaires sur R1 afin de laisser passer les paquets isakmp sur le port 500 notamment. On crée une ACL spécifique au VPN, que l'on associe à une class-map `vpn-class`et que l'on ajoute à la règle de filtrage déjà existante `to-self-policy`.

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


---

<a id="9"></a>
## 9. Mise en place des services de surveillance

<a id="91"></a>
###  9.1 Syslog

 On considère que le client, depuis son PC, veut surveiller son installation avec le service **Syslog**. 
On installe le serveur sur le PC-distant `yum -y install rsyslog` qui a pour adresse IP `192.168.100.2` 

Il faut modifier le fichier de configuration `vi /etc/rsyslog.conf` 
et dé-commenter la partie UDP et TCP syslog reception avec port UDP : 514 et port TCP : 1514.

Il faut redémarrer le système Syslog pour que les modifications soit prises en compte : `systemctl restart rsyslog`

Le trafic TCP 1514 et UDP 514 passe donc par le VPN.

#### Nous avons configuré centos-1 en client Syslog :

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

#### Nous avons également configuré R1 en client Syslog :

En configuration terminal, on active le service Syslog avec `logging trap debugging` et on indique le server `logging 192.168.100.2`

Pour vérifier la configuration de R1 : `show logging`
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


<a id="92"></a>
###  9.2 SNMP

SNMP, tout comme Syslog, permet de collecter des informations de surveillance sur un serveur central provenant d'appareils distants. 
Mais SNMP permet de faire un peu de gestion et est plus sécurisé que Syslog. 

Toutefois, nous ne l'avons pas implémenté. Pour ce faire, il aura fallu suivre les instructions suivantes :


#### Pour configurer les périphériques Cisco

```
snmp-server community Projet3SNMP RO
snmp-server enable traps
snmp-server host 10.192.10.102 private   # L'adresse du server
``` 

#### Sécurisation envisageables

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


---

<a id="10"></a>
## 10. Pour aller plus loin


<a id="101"></a>
### 10.1 Second Switchblock
Afin d'enrichir le réseau principal de notre topologie, un deuxième switchblock a été ajouté, analogue au premier, représentant un deuxième sous-réseau du site principal. 

![Topologie2s](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/Deuxième%20_switchblock.png)

Sa configuration a été effectuée à l'aide de l'outil Ansible. Pour cela, quelques modifications ont été apportées à l'inventaire des livres de jeu :

-	Dans le fichier « hosts » les nouveaux commutateurs Cisco ont été ajoutés (AS3, AS4, DS3 et DS4) ;
-	Les fichiers de configuration « AS3 », « AS4 », « DS3 » et « DS4 » ont été créés dans le dossier « host_vars » ;
-	Les nouvelles interfaces ont été paramétrés dans les fichiers « R2 » et « R3 » ; 
-	La plage des adresses du nouveau sous-réseau ("10.193.0.0 0.0.255.255") a été ajouté dans le paramétrage du protocole NAT dans le fichier « R1 ».

Des `ping` entre les différentes VLANs et les deux switchblocks, ainsi que vers `1.1.1.1` et `www.google.com` ont permis de vérifier le bon fonctionnement de notre topologie en terme de connectivité IPv4. Malheureusement, on ne peut pas en dire autant de la connectivité IPv6, le diagnostic doit être fait en terme d'amélioration future du projet.

<a id="102"></a>
### 10.2 VPN IPsec Ipv6

- ### MISE EN PLACE

Nous avons voulu créer Un VPN IPSEC en IPv6. 

> IPSEC est un standard ouvert de l’IETF pour sécuriser les réseaux IP. Il protège et authentifie les paquets IP d’une origine à une destination grâce à des services de sécurité cryptographiques et à un ensemble de protocoles de transport. 
(*www.cisco.goffinet.org*)

Il assure les fonctions suivantes :

- confidentialité des données
- intégrité des données
- authentification de l'origine
- gestion des clés secrètes

Nous utiliserons ici :
- 3DES comme algorithme de chiffrement du trafic (encryption)
- SHA comme HMAC (intégrité des données)
- Diffie-Hellman en algorithme de chiffrement asymétrique (clés sécrètes)
- ESP (Encapsulating Security Payload) qui est le protocole de transport de la pile IPSEC qui est utilisé pour la confidentialité, l’authentification et l’anti-rejeu des échanges entre deux noeuds IP.

- ### Premier essai Cisco-Fortigate

Nous avions essayé entre 1 Fortigate et 1 Cisco mais cela semble très compliqué voir peut être impossible avec le fortiOS utilisé ici. L’usage de GUI pour ce faire sur FortiOs n’est pas possible, il aurait fallu utiliser CLI, sans être certain du résultat. Il semblerait que même en 2020, les fabricants tels que Fortinet ne se préoccupent pas ou très peu du développement en IPv6. C'est un constat que nous avions déjà effectué sur les PCs virtualisés en VPCS pour lesquels IPv6 n'est pas complètement implantés. 

- ### Second essai Cisco-Cisco

Mise en place d’un routeur R4 avec un LAN avec adressage IPv4 et Ipv6, NAT, DNS, connectivité vers internet, EIGRPv4 et v6 (Id 6.6.6.6).
R4 ne dispose pas de pare-feu.

![image](https://github.com/reseau-2020/projet-three/blob/master/_annexes/_topologies/VPN_IPV6.png?raw=true)

Les fichiers de configurations : 
- [R1](https://github.com/reseau-2020/projet-three/blob/master/Configurations/VPN_IPSEC_R1.txt)
- [R4](https://github.com/reseau-2020/projet-three/blob/master/Configurations/VPN_IPSEC_R4.txt)

Une adresse publique IPV6 a été rajoutée à l'interface g0/0 de R1.

Pour R4 :
> 2001:470:C814:7006::/64

> FD00:470:C814:7006::/64

> FE80::cafe:7 sur interface externe de R4

Mise en place du VPN IPSEC entre R1 et R4.

Nous avons créé une nouvelle interface Tunnel6 de chaque côté avec une adresse IPv6 virtuelle.

Un premier problème est apparu :
```
Dropping udp session [2001:470:C814:3001::1]:500 [2001:470:C814:7006::1]:500 on zone-pair self-internet class class-default due to  DROP action found in policy-map with ip ident 11032
```

Il semblerait que nous ayons un problème de règles de pare-feu.

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
En effet, nous n’avions pas paramétré les ACLs en IPv6.
```

L'ajout de cette ACL a permis d'éviter le log d'erreur, toutefois il nous est toujours impossible de ping d’un côté ou de l’autre. Pourtant le tunnel semble être mis en place et fonctionnel :
```
R1#show crypto isakmp sa

IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
192.168.122.221 192.168.122.55  QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA

 dst: 2001:470:C814:7006::1
 src: 2001:470:C814:3001::1
 state: QM_IDLE         conn-id:   1002 status: ACTIVE
```

Aussi, un `traceroute6`de chaque côté du tunnel (centos-1 et PC-distantR4) nous a permis d'observer l'arrêt des paquets aux interfaces du tunnel. 

L'arrêt total du pare-feu sur R1 nous a permis d'observer le bon fonctionnement du tunnel en IPV6. 

La suite à donner serait de trouver comment règler le pare-feu afin de laisser passer ce trafic sans détériorer la sécurité du réseau. 

Aussi, une bonne pratique serait d'utiliser l'adresse link-local de chacun des routeurs R1 et R4 comme sources et destinations des routes statiques permettant d'accéder au tunnel.

Aussi, il serait préférable d'utiliser des adresses privées comme Ipv6 virtuelles pour les interfaces Tunnel6.

<a id="103"></a>
### 10.3 Focus Sécurité

Par manque de temps, nous n'avons pas pu nous attarder sur l'aspect sécurité de notre topologie. Mais voici quelques pistes d'améliorations qui auraient pu être envisagées.

- Réaliser des scans du réseau à partir d'un PC pirate. Et potentiellement essayer d'accéder aux fichiers contenus sur un périphérique grâce à SSH par exemple.

- Améliorer les ACLs et policy-map des pare-feu en réduisant les blocs d'adresses;

- Mettre en place des groupes d'utilisateurs et des accès restreints en fonction des groupes et des VLANs (sur R1 et FortiOS);
 Notamment avec des mots de passe à plus de 8 caractères, combinaison de lettres, chiffres et caractères spéciaux, ainsi qu'une expiration.

- Interdire l'accès à certaines URLs et cookies sur FortiOS;

![Policy Forti3](https://github.com/reseau-2020/projet-three/blob/master/Configurations/Policy%20Forti3.png)

- Mettre en place un serveur RADIUS afin de sécuriser les accès administratifs aux différents périphériques de la topologie;

- Sécurisation envisageables pour SNMP :
  - En choisissant judicieusement un nom de Communauté 
  - En configurant des SNMP View 
  - En activant des ACLs sur les Communautés et sur les interfaces 
  - En isolant ce trafic dans un VLAN contrôlé par des ACLs 
  - En activant SNMPv3

- Mettre en place des sauvegardes automatiques des configurations et des données

- Installation d'Antivirus


---

<a id="11"></a>
## 11. Annexes

- [Fichiers de configuration](https://github.com/reseau-2020/projet-three/tree/master/Ansible/playbooks/backup)
