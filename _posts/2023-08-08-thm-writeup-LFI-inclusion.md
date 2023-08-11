---
layout: post
title: Tryhackme - LFI inclusion
autor: S3cureroy
date: 2022-08-08
categories: [CTF, Tryhackme]
tags: [ LFI , enumeración , web , passwd ]
---

La entrada de la máquina LFI Inclusión de Tryhackme es corta, puesto que realmente, no se deben seguir los pasos habituales, como la intrusión y posterior escalada de privilegios.

Esta máquina sirve para desarrollar una técnica en concreto, y además, no dispone de medidas de seguridad que permitan filtrar la petición que se envía al servidor, por lo que facilita mucho el compromiso.

---

## Enumeración.

Como la mayoría de las máquinas, se comienza ejecutando una enumeración de los servicios y puertos mediante la herramienta **nmap**, obteniendo los puertos pertenecientes a SSH y HTTP.

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-27 09:31 EDT
Stats: 0:00:02 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 50.60% done; ETC: 09:31 (0:00:01 remaining)
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 09:31 (0:00:06 remaining)
Nmap scan report for 10.10.57.141
Host is up (0.052s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-server-header: Werkzeug/0.16.0 Python/3.6.9
|_http-title: My blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.23 seconds
```

### WEB.
 
Entrando mediante el protocolo HTTP se observa que hay desplegado un blog, y revisando todos los enlaces disponibles, se accede a http://10.10.143.249/article?name=lfiattack

El parámetro "lfiattack" da pistas de qué hay que modificar, y cambiándolo por la ruta relativa de **/etc/passwd**, se obtiene el contenido del fichero.  
![LFI1][LFI1] 

```
root:x:0:0:root:/root:/bin/bash 
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin 
bin:x:2:2:bin:/bin:/usr/sbin/nologin 
sys:x:3:3:sys:/dev:/usr/sbin/nologin 
sync:x:4:65534:sync:/bin:/bin/sync 
games:x:5:60:games:/usr/games:/usr/sbin/nologin 
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin 
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin 
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin 
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin 
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin 
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin 
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin 
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin 
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin 
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin 
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin 
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin 
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin 
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin 
syslog:x:102:106::/home/syslog:/usr/sbin/nologin 
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin 
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin 
lxd:x:105:65534::/var/lib/lxd/:/bin/false 
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin 
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin 
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin 
pollinate:x:109:1::/var/cache/pollinate:/bin/false 
falconfeast:x:1000:1000:falconfeast,,,:/home/falconfeast:/bin/bash 
#falconfeast:rootpassword sshd:x:110:65534::/run/sshd:/usr/sbin/nologin 
mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false 
```

En el listado de usuarios se observa que existe el usuario **falconfeast** con uid 1000 y que además, tiene el directorio principal en la ruta /home/falconfeast. 

Se busca la flag del usuario en dicho directorio (se presupone que el fichero se llama user.txt, teniendo suerte y obteniendo **60989655118397345799**)   

![LFI2][LFI2]

## Flag de root.

Habiendo tenido suerte, se decide buscar la flag de root, suponiendo que ésta se encuentra en el directorio principal que aparece en el listado, y que el nombre del fichero es **root.txt**, obteniendo el siguiente flag (**42964104845495153909**)  

![LFI3][LFI3]

Como se puede ver, en este caso no es necesario escalar privilegios, ya que el propio usuario que corre el servicio web tiene permisos para ver el contenido del directorio root.

Sé que en esta situación, se podría intentar obtener una clave privada SSH, probar la contraseña "rootpassword" del comentario que se encuentra en el fichero passwd, o incluso probar otros métodos de acceso a la máquina.

Como se obtienen las dos flags mediante el método para el que está concebida la máquina, no lo considero necesario.

Espero que os haya gustado, y recordad las redes sociales si queréis contactar conmigo.

Salu2!!!

[LFI1]: https://i.imgur.com/BNTwheR.png
[LFI2]: https://i.imgur.com/NkRmwtQ.png
[LFI3]: https://i.imgur.com/tpTBd3M.png