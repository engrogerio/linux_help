Acesse o PORTAL DO ALUNO através do link: https://forceclass-00.linuxforce.com.br/

# 1-TURN ON / OFF

## Desligar
 
halt
 
powerof
 
shutdown -h now
 
init 0
 
telinit 0
 
## Reiniciar
 
reboot
 
shutdown -r
 
now init 6
 
telinit 6
 
-------------------------
# 2-Basic admin commands

estrutura arquivo /etc/passwd
 
1 - login
 
2 - senha
 
3 - uid
 
4 - gid
 
5 - comentários
 
6 - diretório home
 
7 - shell
 
## enviar msg ao usuario'
 
wall 
 
## HOSTNAMECTL
hostnamectl set-hostname invent.com.br

## VISUDO - configura o sudo

## WHO or w 

check what users are currently connected

## HISTORY 
histsize -> env var that set max items on history
 
histfile -> history filepath
 
history -c (apaga o historico)
 
history -w (comit)
 
history 10 (last 10 lines)
 
!20 -> runs the command on history line 20
 
fc abre editor com ultima linha do historico

## HELP
help - show builtin commands
 
info - show applications (commands) separated by functionality
 
info ls - show info about command ls
 
man -k network - show all manuals with the word network
(nome , (seção), descrição do manual)
 
apropos - same as man -k
 
man 5 systemd.network
 
whatis <command> show what command does brefly
 
man -f - same as whatis
 
whereis <command> - obvious
 
which <command> 
 
mandb - mantem base de dados dos manuais atualizadas
/usr/share/man - repositorio dos manuais
 
## USEFULL PATHS
/usr/share -all installed apps
 
/usr/share/doc - app docs
 
/etc/profile -> configuração global (qualquer usuario) + para env variables
 
~/.bashrc -> configuração do usuario (mais para funções e alias)
 
-------------------------
# Search text and manipulation 

## REDIRECT STDerr, in, out
ls /dev/std*

choose where to redirect : 0-stdin 1-stdout 2-stderr
Ex:
 
find / -size +30M 1> saida.txt 2> erros.txt 
 
find / -size +30M 1> saida.txt 2>&1
 
find / -size +30M 2> /dev/null |xargs ls -lh 
 
find / -iname help - find all files with the word "help"

## TEE
tee  -read from stdin and write to stdout

date |tee file1 file2
|sort |nl -sort and enumerate
head -n 5  = first 5 lines

## NL
nl < /etc/host
 
cat -n /etc/host - show line numbers

## CUT
good for csv like files.
 
cat passwd | cut -d ":" -f 4 = using ":" as separator, bring the 4th column
 
## SET
set -o noclobber -> torna impossivel sobrescrever um arquivo na sessão aberta
 
set +o ... volta 
 
help set

## WC
wc -l /etc/host - return line count
 
wc -w ->word count / wc -c -> characteres count

## TR
tr : a-z A-Z translate all letters to uppercase
 
tr : at AT translate all a to A and t to T
 
tr -d abc - delete chars abc

## GREP
grep -i --color -n -r(recursive) word /etc/ search for word on all files on /etc/
 
grep -v(inverte a busca) --color -n <palavra> /etc/config
 
## REGEX
egrep --color -r [dD]ebian /etc/
 
egrep --color -r [!dD] /etc/ -> todos exceto iniciando com d ou D

## SORT
tee  -read from stdin and write to stdout
 
date |tee file1 file2
|sort |nl -sort and enumerate

## UNIQ
uniq -cd = count repeated lines in sequence
 
uniq retorna não repetidas

## PASTE
Join 2 files line by line

##JOIN
Join 2 files line by line

## CUT
head /etc/passwd | cut -d : -f 1,7 (retorna o login e shell)
 
... | cut -d : -f 1-4 (retorna items 1 a 4)
 
ls -l /etc/*.conf | cut -c 1-10,43-70 (por caractereres)

## AWK
head /etc/passwd |awk -F : '{ print "Login:" $1,"\tDiretório:", $7 }'

## FIND
find /var /etc \
 
-name: (nome exato) \
 
-iname: (ignore case)
 
2> /dev/null -> não mostra erros
 
-user usuario - busca | xargs cat
 
-type l (only links. other options are f, d)
 
-size - por tamanho
 
em minutos
 
-amin  = acessados
 
-mmin = modificados
 
-cmin = alterados
 
em dias
 
-atime = 
 
-mtime = 
 
-ctime =
 
## STAT - show folder/ files statistics

## WATCH 'find...' -> Executa o comando a cada 2segs e mostra na tela

## LOCATE
locate filename.
 
não faz buscas em dirs temporários
 
have to run: updatedb before use locate

## TO DEFINE STD EDITOR
update-alternatives --config editor

## SED
-e = exhibit only
 
-i = change file Inplaceb
 
/g = global (all occurrences)
 
/d = delete line
 
/p = print
 
s = replace
 
1,10 = lines 1 to 10
 
sed -e '1,10s/nologin/login/g'
 
pipe may be used for special chars:
 
replace duble quotes to nothing:
 
blkid |sed -e 's|"||g'
 
-------------------------

# Advanced commands

## DD
dd -conversões e cópias de arquivos
 
criar usb bootavel
 
ex:
 
dd if=/var/log/messages of=/tmp/logs
 
clone partition
 
dd if=/dev/sda  of=/dev/sdb 
 
## CREATING SWAP FILE
adding file as swap file
 
dd if=/dev/zero of=/var/swapfile bs=1M count=500
 
lsblk = lista blocos do disco
 
mkswap /var/swapfile

swapon /var/swapfile

swapon -v -> lista a composição do swap

swapoff /var/swapfi

/proc/swaps = swap info file

## SPLIT
split -b 4M -d /tmp/arquivo /tmp/arq

split -l (line)

juntar: cat arq* >arquivonovo

cat > arquivo.txt -> espera entrada de texto e grava no arquivo

## SCRIPT <filename> 
Grava stdin e stdout em arquivo

-a = append on existing file

exit para sair
 
-------------------------
# Resources administration

## DISK ADMIN

### fdisk -l = lista 
/dev/sdax
 
x 1-4 = partições primarias
 
5-16 = partições lógicas (not mountable)
 
Partição extendida contem partições lógicas que só fazem sentido
 
se são necessárias > 4 partições

### lsblk -f

Show disk partitions

### file /proc/partitions

partitions size store file

### blkid

Show disk partitions IDs (for mounting) and other info. Do not show extended partitions info.

### df

### du -sh /etc /home /usr = show each folder size used
|sort -h = show ordered by size

## CPU ADMIN
lscpu = processor info
 
cat /proc/cpuinfo = file that has cpu info
 
top
 
id = idle
 
P = order by CPU%

## MEM ADMIN
free -h show mem status
 
cat /proc/meminfo (file where mem info is saved)
 
/proc/sys/vm/drop_caches = cache 
 
echo 3 > /proc/sys/vm/drop_caches = clean cache
 
top 
 
M - order by memory

## USB ADMIN
lsusb = show ports and devices connected
 
-v show port details 
 
## PCI ADMIN
lspci = show pci hardware
 
lspci -k = show drivers / kernel modules
 
lspci -v = show memory addresses and IO.

## RESOURCES / HARDWARE INVENTORY
lshw -html (or xml or json) > result.html
 
other options:
 
-short
 
ocs inventory
 
kacy
 
-------------------------
# Manage disks

## 1- Partitionning
fdisk /dev/sdb1

## 2- Formatting
lsblk -f
 
### Formats
            ext4*            xfs        btrfs
 
max files: 4bilhoes         2^64
 
max file    16Tb            8Eb
 
max volume  1Eb             8Eb
 
* mais utilizado
 
### Format to xfs:
mkfs -t xfs  /dev/sdb1
 
### add label to xfs partitions
xfs_admin -L express /dev/sdb1
 
xfs_admin -L log /dev/sdb2
 
xfs_admin -L backup /dev/sdc
 
### add label to ext4 partitions
tune2fs -L git /dev/sdb1
 
tune2fs -L backup /dev/sdb2
 
## 3-Mount

### temporary mount
mount /dev/sdb1 /srv/www/

### definitive mount
man fstab
 
vim /etc/fstab
 
adding
 
/dev/sdb1      /srv/www    xfs   defaults 0* 0**
 
or

LABEL=git      /home/analista/git ext4 defaults 0 0
 
defaults = wr
 
noexec = block execution
 
* dump = opção para backup com dump. 0 = No Dump
 
** pass = To check disk on initialization. 1 = Do check
 
mount -a should mount or show the errors if there is on fstab
 
## Convert ext3 to ext4 format

sha1sum - return hash for file
 
1- umount /backups
 
2- convert partition to ext4
 
tune2fs -O extents,uninit_bg,dir_index /dev/sdb2
 
-------------------------
# Manage file packages

## CPIO
<files list> | cpio -ov > file
 
cat file | cpio -iv

## TAR
tar --create -v --file /backups/conf.tar /home/analista/git /var/log
 
tar -x(extract)vzf 
 
tar -t show content
 
## compression
### gzip
gzip conf.tar
 
gunzip conf.tar.gz
 
### xz
xz conf.tar
 
unxz conf.tar.xz
 
### tar
-z = gzip
 
-J = xz
 
### lsof - show opened files
 
-------------------------
# Network
Virtualbox docs
## DEFINITIONS
NAT = network address translation
 
local -> internet


## COMMANDS 
nmcli con show = config and change interfaces ips
 
ip a = show newtwk interfaces

chattr +i /file = makes the file imutable (-i = remove, lsattr show if is immutable)

https://www.virtualbox.org/manual/ch06.html

repo git/rogerramossilva/linuxforce

## GATEWAY APPLIANCE 

### NETWORK CONFIG - CentOS distro
* copy interfaces from repo (enp0s8 e 0s9)
cp /root/linuxforce/gateway/network-scripts/ TO 
/etc/sysconfig/network-scripts/

iface file example:
TYPE=Ethernet
BOOTPROTO=static or dhcp
DEVICE= same name as the interface name
ONBOOT=yes -> interface active on os load
IPADDR =for 1 ip
IPADDR1 = ip addresses
IPADDR2 = ip addresses


### DNS SERVER
/etc/resolv.conf - resolução de nomes pela ordem de declaração dos servidores:

search asf.com
domain asf.com
nameserver 8.8.8.8
nameserver 192.168.1.10
nameserver 192.168.1.20
* chattr +i /etc/resolv.conf


### ROUTER APPLIANCE CONFIG
/etc/systl.conf
* add the line:
net.ipv4.ip_forward=1

* sysctl -p (to load the changes)

### NAT
To create a firewall rule. (block, manipulate(masquerade), release, etc)

Stop and disable firewalld service
* systemctl stop firwalld
* systemctl disable firewalld

Install and start iptables-services (firewall)
(iptables is deprecated. nftable?)
* yum install -y iptables-services
* systemctl enable iptables
* copy config files from repo /linuxforce/gateway/sbin/firewall.sh to /sbin
* rodar o firewall.sh
* ip route - show routes


### DHCP SERVER (na maquina gateway)
* yum install dhcp-server
copy from git repo: /root/linuxforce/dhcp/dhcpd.conf to
/etc/dhcp/dhcpd.conf - arquivo de configuração

systemctl restart dhcpd
systemctl enable dhcpd

## OTHER APPLIANCES

### NETWORK CONFIG Debian
* edit primary interface:

vim /etc/network/interfaces

auto enp0s3 (interface up when linux load)
iface enp0s3 inet static (or dhcp)
    address 192.168.1.30
    netmask 255.255.255.0
    gateway 192.168.1.1

* restart service 
systemctl restart networking

__________________________________
# Packages management

## Package management for Debian
### Important folders:
/etc/apt/sources.list - config apt install
/var/cache/apt/archives/ - apt keep installers here.
/usr/local/src or /usr/src - place to store source codes.


### APT options
apt search <packname> - suggests packages and similaries
apt show <packname> - show info about the append
apt policy <packname> - show if its installed and the candidates
apt list <packname> - list the exact pack for the app
apt source <packname> - download app source
apt remove <packname> - remove packages leaving changed files.
apt purge <packname> - remove packages including all changed files.
apt download <packname> - download .deb
apt autoremove - remove orfaos packages

apt clean - remove packages deb wich were used to install 

Other package mgmt tool: aptitude



### Compiling

apt build-deb <app> - install all dependencies needed for to compile the app

ap source <app> - download the source code for app

./configure
make 
make install

### Installing from .deb  using dpkg (low level)

* verify if packname is installed:
 
dpkg -l <packname>
 
* baixar o pacote .deb:
 
apt download <packname> - download .deb
 
dpkg -I <packname> show Info when not installed
 
dpkg -s <packname> show info when its installed
 
dpkg -c <packname> show pack structure before installs
 
dpkg -i <packname> install from .deb file
 
dpkg -L <packname> show file structure when installed
 
dpkg -S /usr/sbin/iftop - show to what package this file belongs to
 
dpkg -r <packname> removes the package files
 
## PACKAGES MANAGEMENT FOR CentOS (REDHAT)

### Convert from .deb to .rpm (RedHat)
alien -r packname.Debian

### Important folders:
/etc/yum.repos.d - repos used by the os
/var/log/dnf.log - log on package management
 
enabled = 1 - enabled
 
gpgcheck=1 - check key on the web address

### Installing .rpm packages

RedHat uses dnf or yum as package management


### YUM
yum repolist - show active repos
 
yum makecash - update internally the package list (like apt update)
 
yum check-update - check packages that need to be updatedb
 
yum update - update the packages (like apt upgrade)
 
yum list <packname> - check if pacakge existing
 
yum info <packname> - show info
 
yum search <packname> - search for a package by name
 
yum install <packname> - install

yum download <package> - download the .rmp package file

yum remove <packname> - uninstall package
 
yum purge <packname> - remove even changed files

yum grouplist <optional groupname> - list packs by environment grouplist

yum groupinfo <optional groupname> - list group info

yum groupinstall <groupname> - install group elements

yum groupremove <groupname> - uninstall group elements

yum clean all - remove package files from local folder

### RPM (install a .rpm file)

rpm -q <packname> - check if package is installed
  
rpm -qpi <packname> - query package info before installing
 
rpm -qpl <packname> - query package list before installing
 
rpm -ivh  <packname> --percent  - install verbose human 

rpm -ivh  <packname> --test - test installation
 
rpm -Uvh <.rpm pack> - Upgrade package

rpm -qf <filename> - query file - show to which package the file belongs to.

rpm -e <packname> - erase (remove) a package

rpm -Va - compare instalation and show diferences on files comparing to the original from repo (audit) 

rpm -qa - query all packages installed

__________________________________
# Libraries management
.so shared objects (dinamic libraries)

## important folders
/lib/x86-64-linux-gnu - libraries installed

/etc/ld.so.conf.d - conf files that shows libraries paths

## commands

ldd /bin/ls - list so libraries for the executable (/bin/ls in this case)

ldconfig -p - list all installed libraries cached

ldconfig  - update the cash

__________________________________
# Modules management

## important folders
/boot - kernel compiled image:
    vmlinux - kernel
    config - show what modules are loaded (y)
    aditional modules (m)

/boot/initrd.img.... - kernel image
    
/lib/modules/ - kernel modules available (not loaded) (.ko = modules) 

/lib/modules/5.10.0-20-amd64/ - kernel modules list files 

/etc/modules-load.d - conf files to declare other modules to initialize.

/etc/udev - ??

## commands
lsmod - list all loaded modules

modinfo <mod name> show all module info

rmmod <mod path> remove module (unload)

depmod - atualiza lista de modules 

modprobe <mod name> - read modules.dep e carrega o modulo solicitado

udev - 

__________________________________
# Processes
Activities started.
 
PID = process son
 
PPID = process parent


## important folders
/proc - processes (running on RAM)
 
 
## read commands
ps -aux - list all process: state(Sleep Running), MEM and CPU
 
ps -eF
 
ps -ely - show state and nice (ni) or priority
 
ps -axjf - show process tree
 
ps -ax -o pid,user,nice,stat,command = customized output
 
pstree - show process tree
 
pgrep -a appname - show process related to app
 
pidof app - return app pid if app is running
 
top / htop

## management commands

kill
killall process name - kill all process by name
pkill

man 7 signal - man for Standard signals
kill -l =show signals list

Most used:
15 SIGTERM - terminate signal 
 
19 SIGSTOP - stop signal
 
3 SIGQUIT -
 
9 SIGKILL - kill process
 
18 SIGCONT - continue a stoped process
 
ctr+z - send task to background
 
jobs -l = show backgrounds tasks and its number
 
command & = run in background
 
fg task number = bring task to foreground
 
nohup = run a command imune to hungaps (not related to a tty)
 
### priority
-20 = very hight
 
0 = normal
 
+19 = very low
 
nice -n -20 <process> = start a process with priority -20
 
renice -n -15 <pid> = change the process pid priority to -15


__________________________________
# LOCAL USERS AND GROUPS MANAGEMENT
local user
 
*Users that can login has a bash:
 
root:x:0:0:root:/root:/bin/bash
 
analista:x:1000:1000:analista,,,:/home/analista:/bin/bash
 
*fields:
login: passwd reserve: uid: primary gid:Comments:user home:shell

* /etc/shadow fields
 
user:passwd:days since last passwd change:min days for paswd change: max days for passwd change: alert for days before expired passwd:days for inactivate account after expired pwd:

uid 0 = root
 
uid 1-999 = system user account
 
uid =>1000 = user accounts
 
## important paths
 
/etc/passwd = user database
 
/etc/shadow = user passwords
 
/etc/login.defs = definitions defaults for new user accounts
 
/etc/skel = template for new users home
 

## commands
### Password/login management
getent passwd = read all internal user database in order of user creation
 
getent passwd user = bring user info
 
vipw = open /etc/passwd for edition on a safe mode (not good practice)

vigr = open /etc/group for edition on a safe mode (not good practice)

vigr -s = open /etc/gshadow for edition on a safe mode (not good practice)

getent shadow user = show user password
 
getent group user = show user group
 
getent gshadow group = show group password
 
chage -l user = show password parameters
 
chage user = change parameters interactive
 
chage -M 120 <other params>  user = change parameters 
 
passwd user = change user password
 
passwd -l user = lock user login
 
passwd -u user = unlock user login
 
### Add/ Change/Remove user
adduser user - create user interactivelly and copy home structure from /etc/skel
 
useradd -m -d /hoome/user -g primary _groupname -G other groups -u uid -s /bin/bash -k /etc/skel/ <...other parameters> username
 
usermod -aG group user = add group to user
 
usermod -g grou user = change user primary group
 
userdel -r user = remove user and /home/user directory
 
id user = show all groups a user is in
 
### Add / Change/Remove group
groupadd groupname = create group
 
groupdel groupname = remove group
 
groupmod -n newname oldname = rename group



### Consulta 
getent group <group> = usuarios no group group


### Create shadow file (old linux version)

pwconv (password converter) = create shadow file and add parameters from passwd (remove from passwd) 
 
pwunconv copy passwords from shadow file to passwd file. Removes the shadow file.

__________________________________
# File Management and Permissions

## Change group / owner
chgrp <groupname> <file> - change file main group
 
chown <user>:<group> <file>


## file types
- = common files
d = folders
l = link
s = socket

## std Permissions
umask

chmod examples
chmod ugo-rw <file> = remove read and write permissions from all 
 
chmod a-wr <file> = same

Octal example:
special user group others
    0     6     4     7

## Special permissions
### suid bit = executable will obey the owner of the application being running (not the user logged in)
s = s+x
 
S = s 
 
chmod u+s <executable>
 
or chmod 4777
 

### sgid bit = force folder group. New file or folders will be created on folder group (not user group)
s = s+x
 
S = s
 
chmod g+s <folder>
 
or chmod 2777

### stick bit = only owner may delete even if there is write permission to others. 
t = t+x
 
T = t
 
chmod o+t <file>
 
or chmod 1777

### umask
default 666
 
may be defined on .bashrc or .profile
 
666-660 = umask=006

__________________________________
# Shell Script
/etc/init.d = collection of scripts (chron, etc)
 
bash -x script.sh = show execution


## Commands
echo - display 
read ENVIRONMENTVARIABLE - read from stdin
$? - return last function return type

$0 - script name
$1 - first script parameters
$* - all parameters


## function declaration
name() {
       }
       
or

function name {
              }
              
## importing files onto other function
source <filepath> or . <filepath>

## conditionals
returns true or false
 
(-d = is a directory?)
if test -d /opt/trigger (help test)
or

if [ -d /opt/trigger ]; then 
echo "dir exists"
else
mkdir -pv /opt/trigger

## loop
while read ENVIRONMENTVARIABLE; do
done
 
until read ENVIRONMENTVARIABLE; do
done
 
for i in $(cat /var/scripts/nomes.txt); do
done


__________________________________
# Links
## soft link
ln -s source destin/name
 
most used. Shortcut
 
can be used for diferents partitions
 
## hard links
ln source destin
 
2 names pointing to the same address on the disk.
 
only works for files on the same partitions

__________________________________
# Systemd

Service manager
First process loaded on boot (PID=1)
 
Manage units:

systemd.service - Service units

systemd.socket - Sockets units

systemd.target - Group units

systemd.device - Device based activation

## run levels
ls -l /lib/systemd/system/runlevel*.target - targets for runlevels

systemctl get-default - returns the current run level (same as runlevel)
systemctl set-default multi-user.target - change the current level to non graphical on next restart (same as telinit 2)
set-default graphical.target - back to graphical interface (same as telinit 5)
set-default reboot.target (same as telinit 6 = reboot)


## important paths
/lib/systemd/system = place where all services are stored 
/lib/systemd/system/sysinit.target.wants = all services that will be started
/sbin/init = executables
/run/systemd/generator - mounts

## commands
systemctl list-units --all -> load all units loaded in memory
 list-units --type=service

 list-unit-files --type=service --state=enabled
(services will initialize on boot up)

 reload 
 restart
 start
 stop
 enable
 disable
 status
 mask
 umask
 reboot
 
### check if its enabled
systemctl is-enabled <service-name>

### remove from initialization
systemctl disable <service-name>

### add to initialization
systemctl enable <service-name>

### mask a service
mask <service-name> = creates a symlink to /dev/null, so service can not be used
umask <service-name>

 
### service state options
static - can not be changed (needed)
masked - can not be used. even for dependency calls from other services
enabled - will be load on initialization
alias - point to another service
disabled - will not load on initialization

__________________________________
# Encript partition

## creates and encript partition

### cryptsetup manages disk criptography
apt install cryptsetup

### main options:
 
choose type:
 
cryptsetup -c 
 
ask for password:
 
cryptosetup -y 
 
define critop key size (256, 2024, 4048,...)
cryptosetup -s

LUKS Format
luksformat

luksOpen / luksClose = open or closes device

### creating criptography layer

cryptsetup -y --cipher aes-cbc-essiv:sha256 --key-size 256 luksFormat /dev/sdc1

### Open device for use
cryptsetup luksOpen /dev/sdc1 <volume name>
 
this creates a volume on /dev/mapper/<voulme name>

## format a volume

### Install xfsprogs
apt install -y xfsprogs

mkfs -t xfs /dev/mapper/<volume name>

xfs_admin -L crypt /dev/mapper/<volume name>

### Mount device
mount /dev/mapper/<volume name> /destin

### Close device
cryptsetup luksClose <voulme name>

### Loading cripted drive on os initialization
file /etc/crypttab
 
add line
 
<volume name>   /dev/sdc1   none    luks

### Read /etc/crypttab and mount cripted partition
cryptdisks_start <volume name> (luksOpen)
 
cryptdisks_stop <volume name> (luksClose)

## Automatic mounting
### apt install autofs
 
### edit /etc/auto.master

line 8 add: (everytime /data was open, read /etc/auto.misc (keys)):
 
/data   /etc/auto.misc  --timeout 30 (unmount after 30 sec idle
 
### edit /etc/auto.misc
 
add line:

<volume name>    -fstype=xfs,rw   /dev/mapper/<volume name>

### enable autofs
 
systemctl restart autofs
 
systemctl enable autofs

###  mount auto :
cd /data/crypt

df -h shows folder

### umount auto:
leaves de folder for 30 sec

__________________________________
# SSH 
datacenter ip 20
storage ip 30


## install ssh (debian)
apt install openssh-server

## ssh config files
/etc/ssh/sshd_config = server
/etc/ssh/ssh_config = client

## view ssh connections status
ss -ntpl

## securing ssh
### change port
Port = 52001

### restrict only specific users sshd_config
PermitRootLogin no
AllowUsers analista
AllowGroups 
DenyUsers
DenyGroups

### authenticate with keys (no password)
PasswordAuthentication (yes/no)

### Banner
Banner /etc/issue.net

### restart service
systemctl restart sshd

### Using key pair
ssh-keygen -t rsa -b 2048 -C analista@interno.asf.com

### send keys to destin
ssh-copy-id -p 52001 analista@192.168.1.1
* key is added to .ssh/authorized_keys on destin

### copy files
scp -P 52030 source destin:/folder
-r if source is a folder

### to limit access for some systemd apps
#### Debian
tcpwapper
works only for /sbin/sshd - apps que carregam a libwapp
set apps on /etc/hosts.allow or deny

#### CentOS
tcpwapper is deprecated


----------------------------------------
# Firewall

## Firewall rules
iptables -L -show rules

/sbin/firewall.sh - regras do firewall


## firewall package flow diagram
http://4.bp.blogspot.com/-2bTL0JCdzFQ/UI6hdnTnkzI/AAAAAAAAARg/Wo9V6e8asxo/s1600/FLUXOGRAMA+com+tabelas+e+cadeias+do+IPTABLES.jpg

## Understanding firewall.sh
Table: Filter (table default)
iptables -P INPUT DROP (-P = policy. INPUT = table. DROP - block all
 
INPUT SESSION - packages that enters on firewall
iptables -A INPUT -p tcp --dport 52001 -j ACCEPT (-A = ADD, -p = protocol --dport =destin port -j = what to do)
__________________________________
# Scanning network
## open ports
nmap -v -sT 192.168.0.0/24 - scan 255 addresses against open ports

## all hosts connected
nmap -sn 192.168.0.1/24 

## check all ip routing table
netstat -rn
or 
routel
__________________________________

'
