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

## Serveur syslog

Ici **PC-distant**
```
yum -y install rsyslog
```

### Modification du fichier de configuration 
```
vi /etc/rsyslog.conf
```
```
# rsyslog configuration file
# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html
#### MODULES ####
# The imjournal module bellow is now used as a message source instead of imuxsocck.
$ModLoad imuxsock # provides support for local system logging (e.g. via logger ccommand)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 1514
```

On décommenter la partie UDP et TCP syslog reception et port TCP : 1514.


### Enregistrer et redémarrer 
```
systemctl restart rsyslog
```
## Client

### installation du service rsyslog
```
yum -y install rsyslog
```
### Modification du fichier de configuration 
```
vi /etc/rsyslog.conf
```
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

### Enregistrer et redémarrer 
```
systemctl restart rsyslog
```

## Configuration sur R1
```
logging trap debugging
logging 192.168.100.2
```

On peut vérifier la configuration:
```
show logging
```
```
Trap logging: level debugging, 111 message lines logged
        Logging to 10.192.1.101
```

* On retrouve les logs dans le PC-distant (server) dans le fichier `/var/log`

# MODIFICATION DU PARE-FEU

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
