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



MTA

SAMBA

LVM

RAID

VPN - OPEN VPN

DATE
/etc/localtime
timedatactl

cp
