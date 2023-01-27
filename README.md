
Como assistir a aula ao vivo do curso previsto para começar dia 09/01/23:

Passo 1 - Acesse o PORTAL DO ALUNO através do link: https://forceclass-00.linuxforce.com.br/

Passo 2 - Clique em: LINK PARA ACESSO À SALA DE AULA INÍCIO ÀS 19:30H, e informe seu nome e sobrenome.

Obs: Sempre faça o download das aulas, pois seu acesso em nossa plataforma será de 1 ano à partir de hoje, pois dessa forma você terá as aulas salvas em seu PC.

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

## enviar msg ao usuario'
wall 

## HOSTNAMECTL
hostnamectl set-hostname invent.com.br

## VISUDO - configura o sudo

## WHO or w -> check what users are currently connected

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

~/.bashrc -> usuario (mais para funções e alias)

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

# TR
tr : a-z A-Z translate all letters to uppercase
tr : at AT translate all a to A and t to T
tr -d abc - delete chars abc

# GREP
grep -i --color -n -r(recursive) word /etc/ search for word on all files on /etc/

grep -v(inverte a busca) --color -n <palavra> /etc/config

REGEX
egrep --color -r [dD]ebian /etc/
egrep --color -r [!dD] /etc/ -> todos exceto iniciando com d ou D

# Comandos avançados
dd -conversões e cópias de arquivos
criar usb bootavel
ex:
dd if=/var/log/messages of=/tmp/logs
clone partition
dd if=/dev/sda  of=/dev/sdb 

adding file as swap file
dd if=/dev/zero of=/var/swapfile bs=1M count=500
lsblk = lista blocos do disco

mkswap /var/swapfile
swapon /var/swapfile
swapon -v -> lista a composição do swap
swapoff /var/swapfi
/proc/swaps = swap info file

free -h show mem status

split -b 4M -d /tmp/arquivo /tmp/arq
split -l (line)
juntar: cat arq* >arquivonovo

cat > arquivo.txt -> espera entrada de texto e grava no arquivo

sort
uniq -cd = count repeated lines in sequence
uniq retorna não repetidas


PASTE
JOIN
junta 2 arquivos linha por linha

estrutura arquivo /etc/passwd
1 - login
2 - senha
3 - uid
4 - gid
5 - comentários
6 - diretório home
7 - shell

CUT
head /etc/passwd | cut -d : -f 1,7 (retorna o login e shell)
... | cut -d : -f 1-4 (retorna items 1 a 4)
ls -l /etc/*.conf | cut -c 1-10,43-70 (por caractereres)

AWK
head /etc/passwd |awk -F : '{ print "Login:" $1,"\tDiretório:", $7 }'

FIND
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

STAT - show folder/ files statistics

WATCH 'find...' -> Executa o comando a cada 2segs e mostra na tela

LOCATE
locate filename
não faz buscas em dirs temporários
have to run: updatedb before use locate

DEFINIR EDITOR PADRÃO
update-alternatives --config editor


SED
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

======================================================================

SCRIPT <filename> 
Grava stdin e stdout em arquivo
-a = append on existing file
exit para sair


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
free
cat /proc/meminfo

/proc/sys/vm/drop_caches = cache 
echo 3 > /proc/sys/vm/drop_caches = clean cache

top 
M - order by memory

## USB
lsusb = show ports and devices connected
-v show port details 

## PCI
lspci = show pci hardware
lspci -k = show drivers / kernel modules
lspci -v = show memory addresses and IO.

## INVENTORY
lshw -html (or xml or json) > result.html

other options:
-short
ocs inventory
kacy



# Disk management
===================================================================

# 1- Particionar
fdisk /dev/sdb1


# 2- Formatar

lsblk -f

## Formats
            ext4*            xfs
max files: 4bilhoes         2^64
max file    16Tb            8Eb
max volume  1Eb             8Eb
xfs
btrfs
* mais utilizado

### Format to xfs:
mkfs -t xfs  /dev/sdb1

### add label para partiçõs xfs
xfs_admin -L express /dev/sdb1
xfs_admin -L log /dev/sdb2
xfs_admin -L backup /dev/sdc

### add label para partições ext4
tune2fs -L git /dev/sdb1
tune2fs -L backup /dev/sdb2

# 3- Montar

### temporary mount
mount /dev/sdb1 /srv/www/

### definitive mount
man fstab

vim /etc/fstab
adding
/dev/sdb1      /srv/www    xfs   defaults 0* 0**
or
LABEL=git      /home/analista/git ext4 defaults 0 0

defaults = WR
noexec = block execution
* dump = opção para backup com dump. 0 = No Dump
** pass = To check disk on initialization. 1 = Do check

Mount all defined on /etc/fstab

mount -a should mount or show the errors if there is on fstab
cd /

# Convert ext3 to ext4 format
sha1sum - return hash for file
1- umount /backups

2- convert partition to ext4
tune2fs -O extents,uninit_bg,dir_index /dev/sdb2


# MANAGING FILES PACKAGES
==========================

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
