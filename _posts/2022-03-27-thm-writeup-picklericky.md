---
layout: post
title: Tryhackme - Pickle Ricky
autor: S3cureroy
date: 2022-03-27
categories: [CTF, Tryhackme]
tags: [ webshell , enumeración , web , dirsearch , robots.txt]
---

La máquina homenajea a la famosa serie Rick y Morty, y en ella se tienen que buscar ciertos ingredientes para que Rick pueda hacer sus experimentos.  

La máquina es sencilla, pero permite practicar técnicas básicas de enumeración web y la ejecución de una *Webshell* para obtener mediante comandos, los diferentes ficheros que contienen los ingredientes.  


## Enumeración. ##

Como en todas las máquinas, se debe descubrir que puertos tiene abiertos y por donde se debe atacar, por ello, se lanza el comando nmap con los parámetros de versionado y los scripts *nse*, descubriendo el puerto 22 (SSH) y el 80 (http).

~~~console
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-19 11:57 GMT
Nmap scan report for 10.10.33.136
Host is up (0.13s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:b2:83:56:59:53:ae:cd:08:18:bb:ce:b2:fd:98:fe (RSA)
|   256 cd:40:93:91:74:2d:1a:f1:d0:c3:6e:da:b6:0f:d1:7f (ECDSA)
|_  256 6b:17:73:a4:e3:c9:03:ee:56:f3:1f:f9:dd:94:f0:6a (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.56 seconds
~~~

## Enumeración WEB. ## 

Al descubrir sólo esos dos puertos, se decide empezar por el puerto 80 y enumerar el código de las web, *fuzzear* directorios, etc.

Se puede observar al entrar que en lugar de las típicas flag de las máquinas de CTF, se deben buscar "ingredientes" para que Rick, que se ha vuelto a convertir en pepinillo, vuelva a su forma humana.  

![](https://i.imgur.com/JXoG18v.png)


Por tanto, se empieza a enumerar, comenzando por el código de la propia página, donde se obtiene un posible usuario (**R1ckRul3s**).

![](https://i.imgur.com/XAA6W1f.png)

Otra de las técnicas de la fase de descubrimiento es mirar el fichero **robots.txt** para ver si existe algún directorio o recurso que no se quiera indexar de los robots de los buscadores.

En dicho fichero, se encuentra la palabra **Wubbalubbadubdub** usada mucho por Rick en la serie, y por tanto, una posible contraseña.

Como no se dispone de más información, se procede a hacer *fuzzing* en la web para descubrir nuevos recursos, directorios, etc.

~~~console
dirsearch -u http://10.10.33.136 -t50 -e "html,php,txt" 

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )


Target: http://10.10.33.136/

[11:39:36] Starting: 
[11:39:40] 403 -  300B, /.htaccessBAK                                    
[11:39:40] 403 -  302B, /.htpasswd_test                                  
[11:39:40] 403 -  301B, /.htaccessOLD2
[11:39:40] 403 -  299B, /.httr-oauth
[11:39:40] 403 -  292B, /.htm                                            
[11:39:40] 403 -  303B, /.htaccess_extra
[11:39:40] 403 -  304B, /.htaccess.sample                                
[11:39:40] 403 -  302B, /.htaccess_orig                                  
[11:39:40] 403 -  300B, /.htaccess_sc
[11:39:40] 403 -  298B, /.htpasswds                                      
[11:39:40] 403 -  302B, /.htaccess.save                                  
[11:39:40] 403 -  302B, /.htaccess.orig                                  
[11:39:41] 403 -  302B, /.htaccess.bak1                                  
[11:39:41] 403 -  299B, /.ht_wsr.txt                                     
[11:39:41] 403 -  300B, /.htaccessOLD
[11:39:41] 403 -  293B, /.html                                           
[11:39:43] 403 -  292B, /.php                                            
[11:39:43] 403 -  293B, /.php3                                           
[11:40:12] 301 -  315B, /assets  ->  http://10.10.33.136/assets/         
[11:40:12] 200 -    2KB - /assets/                                          
[11:40:37] 200 -    1KB - /index.html                                       
[11:40:43] 200 -  882B, /login.php                                        
[11:41:05] 200 -   17B, /robots.txt                                       
[11:41:07] 403 -  302B, /server-status/                                   
[11:41:07] 403 -  301B, /server-status                                    
                                                                             
Task Completed
~~~

Se descubren varios directorios, que se proceden a verificar.

El primero es el directorio ```/assets```, que dispone de listado de directorios pero no se encuentra nada relevante, más allá de esta apreciación.  

![](image.png)

El recurso ```login.php```, al acceder solicita una credenciales de lo que parece un panel de administración.

## Acceso inicial ## 

Se intenta acceder al panel mediante el usuario obtenido en el código de la página y como contraseña se introduce la palabra que siempre usa Rick, encontrada en el fichero robots. *Obteniendo acceso*.

Si se cambia a la pestaña *Commands* en el panel, se puede obtener ejecución de comandos en la máquina y por lo tanto, se puede comenzar a buscar los ingredientes que necesita Rick.  

![](https://i.imgur.com/EGpk7BO.png)

> Todos los comandos que muestran contenidos de ficheros están restringidos, menos strings.

### Primer ingrediente.

Dentro de la ruta donde se encuentra la *webshell* alojada existe un fichero llamado **Sup3rS3cretPickl3Ingred.txt** que se obtiene mediante la ejecución de un comando ```ls```.

![](https://i.imgur.com/uJYImpK.png)

Se accede al fichero mediante la ruta en el navegador, obteniendo el primer ingrediente (**mr. meeseek hair**)

![](https://i.imgur.com/uNeIRbx.png)


### Segundo ingrediente. ##

Dentro de la misma ruta donde se aloja el primer ingrediente, existe un fichero llamado **clue.txt** que insta a mirar por todo el filesystem para encontrar más ingredientes.

![](https://i.imgur.com/s4HoQCi.png)

Se hace la búsqueda mediante la palabra "ingred", encontrando el obtenido previamente, y otro en el directorio *home* del usuario de Rick, que se muestra mediante el comando strings, obteniendo **1 jerry tear**.

![](https://i.imgur.com/t6Lkqg5.png)

![](https://i.imgur.com/JC3NJOC.png)

### Tercer ingrediente. ## 

Se intenta verificar si existen más usuarios para poder obtener el tercer ingrediente, pero mirando en el fichero ```/etc/passwd``` no se ven más usuarios (ni siquiera está el usuario rick visto previamente).

Se verifican los permisos que tiene el usuario que ejecuta comandos mediante la *webshell* mediante el comando ```sudo -l```  y se puede ver que el usuario tiene la posibilidad de ejecutar comandos como root sin contraseña, por tanto, se puede verificar que existe en la ruta del usuario root.

![](https://i.imgur.com/oSYvXgv.png)

Se listan los ficheros del directorio *home* del usuario root, obteniendo un fichero que hace alusión al tercer ingrediente (**3rd.txt**)

![](https://i.imgur.com/vvCtaTQ.png)

Se obtiene el contenido del fichero mediante el comanto strings, consiguiendo el tercer ingrediente y completanto la máquina.

![](https://i.imgur.com/XOEqmWF.png)
