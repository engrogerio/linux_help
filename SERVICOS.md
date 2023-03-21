6/2/2023 - Serviços Linux
bind C-! setw synchronize-panes
# DNS
## root servers
https://root-servers.org/

UDP porta 53

INTRANET: ssh -p 52010 analista@192.168.1.10
DATACENTER: ssh -p 52020 analista@192.168.1.20

## Install dns server:
* apt install bind9 bind9utils dnsutils -y
## chroot (isolar serviço de dns) wiki.debian.org/bind9
* mkdir -pv /var/bind9/chroot
* mkdir -pv /var/bind9/chroot/{etc,dev,var/cache/bind,var/run/named}
### point dns service to get info from the folder
* vim /etc/default/named
* add target to OPTIONS: -t /var/bind9/chroot
* ln -s /var/bind9/chroot/etc/bind/ /etc/bind

### create null and random devices
mknod <device name> <device id> <ver minor> <ver <major> (ls -l /dev/null shows the data.
mknod null c 1 3
mknod random c 1 8

### change some permissions
* chmod 660 /var/bind9/chroot/dev/*

dns uses user bind:

* chown bind:bind /var/bind9/chroot/etc/bind/rndc.key /var/bind9/chroot/var/ -R
 
* chmod 775 /var/bind9/chroot/var/ -R 

* cd /etc/init (service scripts)

* edit named file
line 28 change from run/named/named.pid to /var/bind9/chroot/var/run/named/named.pid

### bind9 or named service can not start (systemctl)

### mandatory system - internal SO protection
debian apparmor
centos selinux

### add rules for apparmor

* logs on /var/log/messages shows bind9 is blocked

* edit /etc/apparmor.d/usr.sbin.named
from line 24 add:
  /var/bind9/chroot/etc/bind/** r,
  /var/bind9/chroot/var/** rw,
  /var/bind9/chroot/dev/** rw,

### restart armor service
systemctl restart apparmor

### trust relationship dns config on INTRANET to trust on DATACENTER
### on intranet
* on /etc/bind, create the tsig.key file:
tsig-keygen TRANSFER > tsig.key

* vim tsig.keygen
add line:

server 192.168.1.20 {
    keys { TRANSFER; };
};
* copy to Datacenter /etc/bind and change ip to .10

### copy to DATCENTER

vim /etc/bind/named.config

Add the lines:

include "/etc/bind/tsig.key";
include "/etc/bind/bind.keys";


### INTRANET: copy from git repo rogerramossilva/linuxforce
cp ./intranet/bind_chroot/etc/bind/named.conf.* /etc/bind/ -v
cp ./intranet/bind_chroot/bind/*  /var/bind9/chroot/var/cache/bind/ -v
### for both INTRANET and DATACENTER
*chattr -i /etc/resolv.conf
*add 8.8.8.8 to last line

### DATACENTER: copy from git repo rogerramossilva/linuxforce
cp ./datacenter/bind_chroot/etc/bind/named.conf.* /etc/bind/ -v

### dns global options
* file /etc/bind/named.conf.option
(allow-query - ips that can query dns server)
(allow-recursion)
(allow-transfer -key used for master-slave)
(also-notify - slave ip (only for master dns)
(forwarders - ips to forward when dns has no answer)

* file .default-zones
root servers relations
specify a file for each zone
. = root.hints (/usr/share/dns/root.hints)

### fix missing root.hints
* cp -p /usr/share/dns/root.hints /var/bind9/chroot/var/cache/bind
* restart service named
* named-checkconf - show errors if exists

### check named.conf.local - customized zones (owned domains/zones) asf.com to internal and external nw
* any change, has to change serial number to synchronize with the other dns server

### check zone db for local - /var/bind9/chroot/var/cache/bind/db.asf.internal - table for all services address dns uses.
* dns servers are using NS (name server)
ns1.asf.com. (dot to make it not autocomplete with asf.com)
* MX = mail exchange
* @ = current address
* Serial is used by slave to check if master has changed


### rev.asf.interna (from inside network)
*reverso ip -> nome 
 
### db.asf.externa (from outside network)
@ local domain (asf.com)



### DIG
mx = mail servers
ns = dns servers
soa = start of authority
ptr


dig +short @8.8.8.8 ns guardian.com mx

dig +short gateway.asf.com

dig +short -x 192.168.1.20 = return dns name

________________________________________
# OPENLDAP
* existent users are keeped
* central database dc=asf, dc=com (dc=domain component)
* ou=users, ou=groups (ou=organization unit)
* uid=rogerio (uid=user identification)
* cn=rogerio (cn=common name)

## Install
* apt install slapd ldap-utils -y

* systemctl is-enabled slapd

* cd /etc/ldap
* ldap.conf (client config)

* cd /usr/share/doc/slapd/example/slapd.conf.gz
* gzip -k -d /usr/share/doc/slapd/examples/slapd.conf.gz -c > /etc/ldap/slapd.conf
* vim slapd.conf:
** loglevel - man 5 slapd.conf
* loglevel trace packets
* logfile /var/log/slapd.log

* moduleload change to back_bdb (berkley database)
* do the same to lines 39 and 51
* fix the suffix and rootdn(also uncomment) to asf
* run slappaswd (create hash)
* add to rootpw 

* mv /etc/ldap/slap.d /var/backup
* mkdir /etc/ldap/slap.d

chown openldap:openldap -R /var/lib/ldap/ /etc/ldap/slapd.d/

* test config:
slaptest -f slap.conf -F slapd.d/
restart slapd service and apply the chown again till it works
## 1- cadastrar users e groups locais
* groupadd -g 5000 g_ti
* groupadd -g 5001 g_diretoria
* groupadd -g 5002 g_suporte

* useradd -m -k /etc/skel/ -s /bin/bash -u 5000 -g g_ti -G g_diretoria,g_suporte kamila
* change password:
    passwd kamila


## 2- gerar arquivo de importação ldif para o LDAP apartir dos usuarios locais
### pode se usar LDAP account manager.
### install migrationtools
* scripts on /usr/share/migrationtools/

* vim migrate_common.ph
    * line 71 and 74 change to asf
    * line 96 and 97 to 5000 and uncomment
    * line 100 and 101 to 6000 and uncomment
* vim migrate_base.pl
    line 39 add fullpath to require
    
* run ./migrate_base.pl > /etc/ldap/base.ldif

* vim migrate_group.pl
    line 39 add fullpath to require

* run ./migrate_group.pl /etc/group /etc/ldap/groups.ldif
* vim migrate_common.ph
    line 39 add fullpath to require

* run ./migrate_passwd.pl /etc/passwd /etc/ldap/users.ldif

    
## 3- importar ldif

cd /etc/ldap

* ldapadd -x -D cn=admin,dc=asf,dc=com -f base.ldif -W
-x = simple authentication
-W = ask passwor

* ldapadd -x -D cn=admin,dc=asf,dc=com -f groups.ldif -W
* ldapadd -x -D cn=admin,dc=asf,dc=com -f users.ldif -W

* slapcat - Show import result
### search ldap
* ldapsearch -x -b dc=asf,dc=com uid=sofia -LLL (-LLL do not show header)

## 4- change 
* vim shell.ldif

dn: uid=sophia,ou=People,dc=asf,dc=com
changetype: modify
replace: loginShell
loginShell: /bin/bash

* ldapmodify -x cn=admin dc=asd,dc=com -f shell.ldif -W


## backup base de dados
* Stop ldap service
systemctl stop slapd

* slapcat -l /var/backups/bkpldap.ldif

## deletar toda a base de dados
(-r = remove)
* ldapdelete -x -D cn=admin,dc=asf,dc=com -r dc=asf,dc=com -W
* slapcat (shows empty)

* ldapdelete may be used with dn for specific users

## restore the ldap db
* stop slapd
(-c = ignore errors)
* slapadd -cl  /var/backups/bkpldap.ldif

## install interface ldap
sudo apt -y install apache2 php php-cgi libapache2-mod-php php-mbstring php-common php-pear

* sudo apt install ldap-account-manager -y

* vim /etc/apache2/conf-available/ldap-account-manager.conf

* 192.168.1.100/lam/templates/login.php
* lan configuration
* edit server profile
    password: lam
    server address: ldap://192.168.1.20:389
* Tool Settings
* TreeSufix: dc=asf,dc=com
* Security Settings
* List of valid users: cn=admin,dc=asf,dc=com

* Account type -> active account types
    Users and Groups LDAP sufix change to asf.com

## add computer to LDAP
* apt install libnss-ldapd  libpam-ldapd  ldap-utils -y

URI do servidor dap
    ldap://192.168.1.20

services: passwd group shadow

## plugable authentication modules
* pam-auth-update
    * mark "Create home directory on login"
Now on login a user existing on ldap, it will create user home


## Mail server
* MTA - Mail Transfer Agent

* MUA (mail user agent) -> MTA1 <- protocol SMTP (port 25) SMTPS (port 465 or 587)-> MTA2
* MDA Mail delivery agent (filter /anti virus on messages)
* MAA Mail access agent (IMAIM (143  and 993)  or POP3 (port 110 and 995)

* MTA - Postfix
* MAA - Dovecot
* MUA - Thunderbird

apt install postfix procmail bsd-mailx -y

systemctl status postfix

*ss -ntpl
(see port 25 smtp)

* vim /etc/services - ports mapping

* Install dovecot
apt install dovecot-core dovecot-pop3d dovecot-imapd -y

* vim /etc/dovecot/dovecot.conf
line 30 - uncomment listen (all ips listening for requests)

* vim /conf.d/10-auth.conf (enable auth)
line 10 - uncomment and change to no
line 100 - add: login

* vim /conf.d/10-mail.conf (tell dovcot the mailbox path)
comment line 30
add line mail_location = maildir:~/Maildir (mailbox on user home)

* vim /conf/10-master.conf (usr/grps needed for spooler)
line 107 and 108 - uncomment
add lines:
    user = postfix
    group = postfix
 }
 
* systemctl restart dovecot
* ss -ntpl |grep dovecot

## config Postfix
### enable ports
/etc/postfix
* vim master.cf 
uncomment lines 17 to 21
right after line 21, add line -o smtpd_sasl_type=dovecot

uncomment line smtps and next 3 lines after that

* restart service
systemctl restart postfix

* check if ss -ntpl |grep master shows ports 25, 587 and 465

## add Datacenter vm to LDAP
* apt install libnss-ldap libpam-ldapd ldap-utils -y
ldap://192.168.1.20
mark passwd, group and shadow

* pam-auth-update
Select "create home directory on login"




SAMBA

LVM

RAID

VPN - OPEN VPN

DATE
/etc/localtime
timedatactl

cp
