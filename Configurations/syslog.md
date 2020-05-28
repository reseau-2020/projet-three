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

MODIFICATION DU PARE-FEU

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

