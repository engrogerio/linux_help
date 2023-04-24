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

## install libapache2-mod-authnz-external pwauth
* enable module:
    a2enmod authnz_ldap

* add on config:    
AddExternalAuth pwauth /usr/sbin/pwauth 
SetExternalAuthMethod pwauth pipe 
<Directory "/srv/www/express"> 
        AuthType Basic 
        SSLRequireSSL 
        AuthName "Members Only" 
        AuthExternal pwauth 
        Require valid-user 
</Directory>

## LDAP auth
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

* to remove Indexes
remove Indexes word

* remove apache version on browser
vim /etc/apache2/conf-available/security.conf
ServerTokens - change from OS to Prod
ServerSignature - Change from On to Off 

_________________________________________________
# NGINX
* apt install nginx
* nginx.conf
/sites-availabe/conf
* root - base serving folder
* index - order nginx search for index files
* server_name - www.asf.com
* location / means base folder

* remove comments for pass PHP scripts to FastCGI server

* install apt install php7.4 php7.4-common php-pear php7.4-mbstring php7.4-fpm -y

* systemctl enable php7.4-fpm nginx
* nginx -t

### seting php app
* echo '<?php phpinfo(); ?>' >/usr/share/nginx/html/index.php


## 
cd /usr/share/nginx
wget https://github.com/rogerramossilva/linux/raw/master/asf.zip

* vim /etc/nginx/sites-available/default
line 41 change to nginx/asf;

* create dir /etc/pki/nginx

openssl genrsa -out asf.key 2048
openssl req -new -key asf.key -out asf.csr
openssl x509 -req -days 365 -in asf.csr -signkey asf.key -out asf.crt

* install the certificate
* add to /etc/nginx/sites-available/default
server {
        listen 443;
        listen [::]:443;

        server_name www.asf.com;

        root /usr/share/nginx/asf;
        index index.php index.html;
        ssl_certificate "/etc/pki/nginx/asf.crt";
        ssl_certificate_key "/etc/pki/nginx/asf.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;


        location / {
                try_files $uri $uri/ /index.php;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }
}

## redirect

* vim default - add line 52
rewrite ^(.*)$  https://www.asf.com/            permanent;


## proxy reverso
request to vespress.com -> arrive on storage.asf.com
### get real IP address
        proxy_set_header        X-Real-IP $remote_addr;
### keep original request
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        
        
        
* new file /available/express
server {
        listen 80;
        index index.php index.html index.htm;
        server_name vexpress.asf.com;
        proxy_redirect          off;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_sed_header        Host $http_host;

        location / {
                proxy_pass https://express.asf.com/;
        }
}



### dns change
ssh -p 52010 analista@192.168.1.10

cd /var/bind9/chroot/var/cache/bind/
vim db.asf.interna

ADD:vexpress IN A 192.168.1.30

* storage: vim /etc/resolv.conf
move 8.8.8.8 to the end

systemctl restart bind9


### firewall change
gateway:
vim /sbin/firewall.sh uncoment NAT SESSION section item #4 (line 170) and items 8 and 9 from FORWARD section 

run : firewall.sh

----------------------------
# PAM plugable authentication modules
** Not a Service **
* Manager type
_auth
_account
_password
_session

* Controls
_required - in case of fail , do not show message imediatly
_requisite - in case of fail, show message imediatly
_suficient - if not fail, its sufficient
_optional - do not interfere on the process no matter the result

* Module name

* Value

modulos
* cd /lib/x86_64-linux-gnu/security

config
* cd /etc/pam.d

* common-* config(affects the entire system)

Ex:auth required pam_faildelay.so delay=3000
a
# avoid othe users to login:
* vim /etc/nologin

## add time constraint to ssh and login
### enable modules
* add on sshd:
account  required   pam.time.so

### config
* vim /etc/security/time.conf
login;*;analista;Al0800-1800
sshd;*;analista;Al0800-2200

## Using module well to allow users to login with another user
* groupadd admins
* gpasswd -a analista admins

* vim /etc/pam.d/su
lin15 uncomment and add group=admins

## Limit simultaneous login or files opened
---------------------------
# Proxy service - Squid and ldap integration.
## DOCS /usr/share/doc/squid/squid.conf.documented
## Allow filter on what can be accessed
## Store internal cache.
## Firewal - services / Proxy http/ftp

* Gateway is the passage of the data flow from internal client to internet.

```
yum install squid -y
cd /etc/squid
vim squid.conf

#  on the beggining of the file, add: 

# calling basic_ldap_auth 
auth_param basic program /usr/lib64/squid/basic_ldap_auth -b dc=asf,dc=com -f uid=%s 192.168.1.20
# start 5 processes for authentication
auth_param basic children 5
auth_param basic realm Proxy Squid
# how long to store creds for inactive session?
auth_param basic credentialsttl 2 hours

# user must authenticate
acl password proxy_auth REQUIRED
# block non authenticated users
http_access deny !password


visible_hostname proxy.asf.com

# 1- created acls
# acl src - source ip
# acl dst -
# port, dstdomain,srcdomain, auth, arp, time,
url_regex,

# 2- create rules

```
* UNCOMMENT line 64 (cache_dir)
cache dir ufs /var/spool/squid 100 (mb-dir size) 16 (initial dirs) 256 (dirs inside first 16)
errorpage.css - error message for user when access is blocked

### enable port 3128 on the firewall
vim /sbin/firewall.sh
* uncomment line 71 (related to port 3128)
firewall.sh
* uncomment line 103 (output session, item 7)


### logs on /var/log/squid/access.logfile

### reconfigure
Parse file for errors:
* squid -k parse -f /etc/squid/squid.conf
Run config
* squid -k reconfigure

=============================
# RAID
apt install mdadm

mdadm --create /dev/md0 --level=5 --raid-devices=6 /dev/sdb .../dev/sdg --spare-devices=1 /dev/sdh 

mdadm --detail  /dev/md0

cat /proc/mdstat

## mount
mount /dev/md0 /mnt

## format ext4
mkfs -t ext4 /dev/md0

=============================
#LVM - logical volume manager
apt install lvm2

1- pvcreate - create physical volume
- prepare disk for use (divide in small blocks called PE (physical extent)
* pvcreate /dev/md0

-list devices already existing
* pvscan

* pvdisplay

2- create VG - volume group 
- create group 
* vgcreate storage /dev/md0
- expand to other dev id needed
* vgextend storage /dev/md1

* pvdisplay now shows VG name and size ( PE size)

* vgdisplay storage

3- create LV - logic volume
* lvcreate -L 2G -n devel storage

* lvscan
4- Format LV

5- Mount LV

-------------------------------------------------
#SAMBA
- assistir aulas 23 a 25?

-------------------------------------------------
# VPN - OPEN VPN
easy-rsa = scripts to create certs

* yum install openvpn easy-rsa -y

* /usr/share/easy-rsa/3/easyrsa init-pki

* /usr/share/easy-rsa/3/easyrsa build-ca nopass

* cp ca.crt to both client and server directory

VPN trust relationship

1st request client -> server on internet (unsecure key exchange) using diffhelm key:
* /usr/share/easy-rsa/3/easyrsa gen-dh

* cp dh.pem to both client and server directory

_create server certificate (named vpn-server):
* /usr/share/easy-rsa/3/easyrsa build-server-full vpn-server nopass
* cp /pki/private/vpn-server.key and /pki/issued/vpn-server.crt to ./server

_create client certificate
* /usr/share/easy-rsa/3/easyrsa build-client-full vpn-client-01

-certificate revoking list (in case of fired employee for example)
* /usr/share/easy-rsa/3/easyrsa gen-crl
* copy to both client and server dirs

-to revoke:
* /usr/share/easy-rsa/3/easyrsa revoke <cert number>

-create simetric key for communication between client and server (tls alt)
* openvpn --genkey secret pki/ta.key

-criar arquivos de configuração da vpn server e client (first file to be read when service start)
* cp client.conf and server.conf (from repo)

- server.conf
dev tun - will create /dev/tun0
local 200.50.100.10 (server public ip)
ifconfig 10.0.0.100 - network addrs server
prot udp
port 1194 (service port that must have firewall rule for udp on 1194)
keepalive 10 120 (check if connection is active every 10 seconds. if not, restart after 120 seconds)
user/group nobody (no previleges user)
route 10.0.0.0 255.255.255.0 - can see all the 10... network
client-config-dir /etc/openvpn/ccd -dir for specific client config per file 

-start with @server for reading server.conf
systemctl restart openvpn-server@server
systemctl enable openvpn-server@server

- vim /sbin/firewall.sh
* line 51 (input session)
1-
iptables -A INPUT -i tun0 -j ACCEPT
* line 75 - uncomment
* line 87 (output session)
iptables -A OUTPUT -o tun0 -j ACCEPT
* line 111 - remove comment (ssh cliente externo) temporarily to send files.
* line 118 - add: (any connection forward to the server)
iptables -A FORWARD -i tun0 -j ACCEPT
iptables -A FORWARD -0 tun0 -j ACCEPT

## copy client folder to client(cliente externo)
install openvpn on cliente externo

* mv /client -> /etc/openvpn/client
* vim client.conf
remote -> ip for server (for initial connection)
* systemctl edit --full openvpn-server@client
change WorkingDirectory to client
* systemctl enable openvpn-server@client
* systemctl restart openvpn-server@client
* ip a -> show tun0 as down cause has no ip

* on vpn server, /etc/openvpn/ccd
* vim vpn-client-01 and add:
ifconfig-push 10.0.0.200 10.0.0.100
push "route 192.168.1.0 255.255.255.0 10.0.0.200"
push "dhcp-option DNS 192.168.1.10"


* ip a should show tun0 up

* ip route show routes through vpn


--------------------------------------------------
DATE
/etc/localtime
timedatactl

cp
