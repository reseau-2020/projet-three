# NTP
## Configuration de NTP

```
conf t
clock timezone GMT +1         #Ajustement de l'heure des périphériques
clock summer-time FR recurring last SUN MAR 02:00 last SUN OCT 02:00
ntp server 3.fr.pool.ntp.org     #Server NTP publique sur R1 
ntp update-calendar
``` 

Interface R1 (10.1.1.1) en tant que serveur NTP pour les autres périphériques Cisco :
```
ntp server 10.1.1.1
ntp update-calendar
```
Désactiver le routage sur AS1 et AS2 et ajouter une route par défaut (l’adresse utilisée a été la passerelle du sous-réseau virtuel de gestion (VLAN99))
```
no ip routing
ip default-gateway 10.192.1.252
```

## Vérification de NTP
```
show ntp status
show calendar
show clock
show ntp config
```

# Syslog
## Configuration Syslog

On considaire que l'on veut vérifier notre topologie depuis chez nous. 

```
conf t
service timestamp log datetime
service timestamp debug datetime
```

### Notre serveur est le PC-distant : 
```
conf t 
yum -y install rsyslog
vi /etc/rsyslog.conf
```
Il faut enlever les commentaire contenu dans cette partie pour activer le transport TCP et UDP : 
```
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 1514
```
Puis `systemctl restart rsyslog`

### Sur les postes de travail 
```
conf t
yum -y install rsyslog
vi /etc/rsyslog.conf
```
Ajouter cette partie dans le fichier 

```
# rsyslog configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

# The imjournal module bellow is now used as a message source instead of imuxsock.
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability
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
*.* @@192.168.100.2:1514
```
UDP est le plus rapide, mais l’inconvénient c’est qu’en cas de surcharge réseau, vous risquerez des pertes de données. 
Ce qui ne serait pas le cas par exemple avec le protocole TCP.

@ pour le protocole UDP

@@ pour le protocole TCP

Puis `systemctl restart rsyslog`

## Sur les routeurs
```
conf t
logging trap debugging
logging 192.168.100.2
```
Pour vérifier : 
```
R1#show logging
    Trap logging: level debugging, 58 message lines logged
        Logging to 192.168.100.2 
```

* On retrouve les logs dans le PC-distant (server) dans le fichier `/var/log`

# SNMP

## Config périphériques Cisco

snmp-server community Projet3SNMP RO
snmp-server enable traps
snmp-server host 10.192.10.102 private

## Securisation envisageables

SNMPv2c se sécurise :

En choisissant judicieusement un nom de Communauté
En configurant des SNMP View
En activant des ACLs sur les Communautés et sur les interfaces
En isolant ce trafic dans un VLAN contrôlé par des ACLs
En activant SNMPv3

## REMOTE

Installation des outils Net-SNMP3 sous Rehat Enterprise Linux ou Centos (RHEL7) :

yum install net-snmp-utils

Usage :

snmpwalk -v2c -c <nom de la communauté> <périphérique à gérer>
