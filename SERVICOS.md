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

## configure postfix
 
* man 5 postconf - configure postfix´

* postconf = change /etc/postfix/main.cf

* postconf |more - show all existing config

* postconf -e myorigin=asf.comment
or
* edit /etc/postfix/main.cf

* cp /root/linuxforce/datacenter/postfix/etc/postfix/main.cf /etc/postfix

* vim main.cf
    on line 6 add mydomain = asf.com
    
## Create Alias

* vim /etc/aliases
add rh: kamila@asf.com
add ti: rafaela@asf.com

## compile aliases file (creates /etc/aliases.db)

* postalias /etc/aliases 

* systemctl restart postfix

## Send email

* cat /etc/hosts |mail -s "TESTE 01" ti@asf.comment

* mailq or postqueue -p - check if there is a message to be send on the queue

* check /var/log/mail.log file


## Relay - connect to external servers

## vim main.cf
* line 46: add asf.com

## Install maildrop (command maildirmake)

* maildirmake Maildir
* maildirmake Maildir/.Enviados
* maildirmake Maildir/.Ocultos
* maildirmake Maildir/.Lixeira
* maildirmake Maildir/.Spam


## run mailuser 
/root/linuxforce/datacenter/postfix/mail_user.sh



# smtpd_sasl_type

apt install libsasl2-2 sasl2-bin libsasl2-modules

* vim /etc/default/saslauthd

OPTIONS change to /var/spool/posfix/var/run/saslauthd

* mkdir -pv /var/spool/postfix/var/run/saslauthd

* add user postfix sasl2

* systemctl restart saslauthd
* systemctl enable saslauthd

## Create certs mentioned on /etc/postfix/main.cf
* mkdir /etc/postfix/ssl

### create key
openssl genrsa -out webmail.key 1024

### cert requests
openssl req -new -key webmail.key -out webmail.csr


Country Name (2 letter code) [AU]:BR
State or Province Name (full name) [Some-State]:SAO PAULO
Locality Name (eg, city) []:SAO PAULO
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ASF
Organizational Unit Name (eg, section) []:TI
Common Name (e.g. server FQDN or YOUR name) []:mail.asf.com
Email Address []:analista@asf.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

### cert sign
openssl x509 -req -days 365 -in webmail.csr -signkey webmail.key -out webmail.crt

### create CA key (certificate authority)
openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 365

### restarting services
systemctl restart postfix dovecot saslauthd

_________________________________________________
# Webservers - 24/3/2023

apache.org
httpd.apache.org (apache2)
## serviços modulares
core + modules + configurations + site

mods /conf/sites
available - initial deployment
enabled - up

### Modules
a2enmod (enable)
a2dismod (disable)

### Config files
a2enconf
a2disconf

### Sites
a2ensite
a2dissite

### vim /etc/apache2/apache.conf
apache2ctl -t - check if there is an error on conf files


### env vars /etc/apache2/envvars

### to create file /etc/apache2/available-sites/ express.conf


<VirtualHost  express.asf.com:80>
        DocumentRoot /srv/www/express
        ServerName express.asf.com
        ServerAdmin analista@asf.com
        ErrorLog /var/log/apache2/express-error.log
        CustomLog /var/log/apache2/express-access.log common
</VirtualHost>

apache2ctl -t

systemctl restart apache2

en2sites express.conf

## Criar SSL cert para https
* create dir /etc/ssl/express
* enable 2 apache modules
    a2enmod ssl
    a2enmod rewrite - module need for doing redirect 
    
* Create cert and key
openssl genrsa -out express.key 2048
openssl req -new -key express.key -out express.csr

* signature on cert
openssl x509 -req -days 365 -in express.csr -signkey express.key -out express.crt

* change the conf file
<VirtualHost  express.asf.com:443>
        DocumentRoot /srv/www/express
        ServerName express.asf.com
        ServerAdmin analista@asf.com
        ErrorLog /var/log/apache2/express-error.log
        CustomLog /var/log/apache2/express-access.log common
        SSLEngine On
        SSLCertificateFile /etc/ssl/express/express.crt
        SSLCertificateKeyFile /etc/ssl/express/express.key
</VirtualHost>

* add redirect http -> https

<VirtualHost express.asf.com:80> 
        RewriteEngine On 
        Options FollowSymLinks 
        RewriteCond %{SERVER_PORT} 80 
        RewriteRule ^(.*)$ https://express.asf.com/  [R,L] 
</VirtualHost>

## Install db on Datacenter vm
* apt install mariadb-server -y

* run command:
    mariadb-secure-instalation
password for root <enter>
switch to unix_socket authentication y
change root passwd y
Linuxforce01

Remove anonimous user y
Disallow root login remotelly y
Remove test database and access y
Reload privilege tables y


## create database
* mysql -u root -p
* show databases
* create database backup;
* use backup;
* source /root/linuxforce/datacenter/backup.sql

## create user for database
grant all privileges on backup.* to express@'%' identified by 'AllSafe0!';

flush privileges;

show grants for express@'%';

* vim /etc/mysql/mariadb.conf.d/50-server.cnf
    comentar linha com bind-address ou mudar para 0.0.0.0

## add apache2 autentication
add on express.conf
<Directory "/srv/www/express/backup.php">
        AuthType Basic
        AuthName "Acesso restrito a equipe de Backup"
        AuthUserFile /etc/apache2/.express
        Require valid-user
</Directory>

## create auth file
-c create (only first time)
-m mda
htpasswd -c -m /etc/apache2/.express rogerio

## Pam authentication module (did not worked)
pam - authentication, authorization, session and password
* apt install libapache2-mod-authnz-pam
* create file /etc/pam.d/apache2
    auth required       pam_unix.so nullok
    account required    pam_unix.so 
* add on express.conf
<Directory "/srv/www/express" >
    AuthType Basic
    AuthName Express Security
    AuthPAM_Enabled On
    Require valid-user
</Directory>
 

## LDAP auth
* install libapache2-mod-authnz-external pwauth

* add:
<Directory "/srv/www/express/downloads">
        Options Indexes FollowSymLinks
        AllowOverRide none
        SSLRequireSSL
        AuthType Basic
        AuthName "Ldap Authentication"
        AuthBasicProvider ldap
        AuthLDAPURL "ldap://192.168.1.20:389/ou=People,dc=asf,dc=com?uid?sub?(objectClass=*)"
        Require valid-user
</Directory>

SAMBA

LVM

RAID

VPN - OPEN VPN

DATE
/etc/localtime
timedatactl

cp
