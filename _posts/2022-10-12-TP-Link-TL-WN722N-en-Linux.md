---
layout: post
title: Habilitar modo monitor con adaptador TP-Link TL-WN722N (Realtek)
autor: S3cureroy
date: 2022-10-12
categories: [Tutoriales, Hardware]
tags: [tl-wn722n, wi-fi, linux, aircrack-ng, rtl8188eus, firmware]
---

Hace unos días, adquirí un adaptador Wi-fi, el cual quería usar para realizar auditorías a redes inalámbricas.

Tras obtener algunas previamente con resultados bastante infructuosos, me decanté por la **TP-Link TL-WN722N**, debido a su precio ya que no espero usarla demasiado, y a que era recomendada para realizar un taller del evento [Mundohacker Academy 2022](https://mundohackeracademy.com/#agenda). 

![caja antena](https://i.imgur.com/cnI15YH.jpg)

La primera sorpresa que me llevé fue que al conectarla a la máquina virtual de Kali Linux, reconocía todas las redes automáticamente, sin necesidad de instalar ningún *driver*, ni nada parecido. Algo que no me ocurría con otros modelos comprados previamente.

![redes disponibles](https://i.imgur.com/BGgs6rv.png)

Entonces, se intenta cambiar el modo de la antena de modo *Managed* a modo *Monitor* pero ésta, no parece reconocer dicho comando.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ sudo airmon-ng start wlan0

PHY     Interface       Driver          Chipset

null    wlan0           r8188eu         TP-Link TL-WN722N v2/v3 [Realtek RTL8188EUS]

╭─s3cureroy at kali in /home/s3cureroy
╰─○ iwconfig              
lo        no wireless extensions.

eth0      no wireless extensions.

eth1      no wireless extensions.

wlan0     unassociated  ESSID:""  Nickname:"<WIFI@REALTEK>"
          Mode:Managed  Frequency=2.412 GHz  Access Point: Not-Associated   
          Sensitivity:0/0  
          Retry:off   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=0/100  Signal level=0 dBm  Noise level=0 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

~~~
> Se puede ver, que pese a cambiar la interfaz a modo *Monitor*, se mantiene en modo *Managed*.
{: .prompt-info} 

## Modificando el firmware.

Revisando entonces los *drivers* cargados en el sistema para la antena, se puede ver que es el **r8188eu**, del que se informa que es cargado desde el directorio de arranque y no se conoce la calidad de dicho firmaware para el dispositivo.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ dmesg | grep wifi  
[   27.593638] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[   50.451326] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[  370.552452] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[  604.471103] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
╭─s3cureroy at kali in /home/s3cureroy
╰─○ dmesg | grep 8188eu
[   26.133520] r8188eu: module is from the staging directory, the quality is unknown, you have been warned.
[   27.199667] usbcore: registered new interface driver r8188eu
[   27.593638] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[   50.451326] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[  370.552452] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[  604.471103] r8188eu 1-2:1.0: firmware: direct-loading firmware rtlwifi/rtl8188eufw.bin
[  846.757874]  ? rtl8188eu_hal_deinit+0x199/0x1f0 [r8188eu]
[  846.757920]  rtw_cmd_thread+0x4b/0x1b0 [r8188eu]
[  846.757950]  ? rtw_setassocsta_cmdrsp_callback+0x80/0x80 [r8188eu]
[  967.338370]  ? rtl8188eu_hal_deinit+0x199/0x1f0 [r8188eu]
[  967.338467]  rtw_cmd_thread+0x4b/0x1b0 [r8188eu]
[  967.338493]  ? rtw_setassocsta_cmdrsp_callback+0x80/0x80 [r8188eu]
[ 1088.129644]  ? rtl8188eu_hal_deinit+0x199/0x1f0 [r8188eu]
[ 1088.129684]  rtw_cmd_thread+0x4b/0x1b0 [r8188eu]
[ 1088.129708]  ? rtw_setassocsta_cmdrsp_callback+0x80/0x80 [r8188eu]
~~~

Entonces, al buscar por el código del *driver* en Internet, se obtiene que el propio Github de [aircrack-ng](https://github.com/aircrack-ng/rtl8188eus) dispone de un repositorio con los *drivers* modificados para poder cambiar el modo de escucha de la antena.

Por tanto, se comienza el procedimiento para cambiar el *firmware*, empezando por actualizar los repositorios y el sistema, reiniciando posteriormente.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ sudo apt update && sudo apt upgrade
~~~

Una vez reiniciado el sistema y cargado el nuevo kernel, si hubiera instalado una nueva versión, se deben instalar las cabeceras de dicho kernel y las dependencias del *firmware*.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ sudo apt-get install linux-headers-$(uname -r) bc build-essential libelf-dev dkms git
~~~
> Los paquetes linux-headers-$(uname -r), build-essential y dkms, son las dependencias necesarias para instalar *Guest Additions* de VirtualBox
{: .prompt-tip}

Instaladas todas las dependencias, se puede comenzar a sustituir el *firmware* por el modificado por aircrack-ng, para ello, se desactiva el *driver* actual.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ sudo rmmod r8188eu.ko
~~~
> En este momento, las redes disponibles desaparecerán y el dispositivo dejará de reconocerse.
{: .prompt-info}

Se clona el repositorio nuevo desde Github.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─○ git clone https://github.com/aircrack-ng/rtl8188eus.git
~~~

Se accede al directorio y se escala privilegios para modificar el *firmware* en siguientes pasos.

~~~zsh 
╭─s3cureroy at kali in /home/s3cureroy
╰─○ cd rtl8188eus
~~~

~~~zsh 
╭─s3cureroy at kali in /home/s3cureroy
╰─⠠⠵ sudo -i
~~~

Con permisos de root, se debe modificar el *firmware* que cargará en el arranque del sistema, modificando el fichero de Realtek, reiniciando posteriormente.

~~~zsh 
╭─root at kali in /home/s3cureroy
╰─⠠⠵ echo 'blacklist r8188eu' | sudo tee -a '/etc/modprobe.d/realtek.conf'
~~~

Cuando la máquina haya reiniciado correctamente, se continua el procedimiento para compilar el nuevo *driver* (paciencia, la compilación puede tardar unos minutos), entrando de nuevo en el repositorio clonado.

~~~zsh 
╭─s3cureroy at kali in /home/s3cureroy
╰─○ cd rtl8188eus
~~~

~~~zsh 
╭─s3cureroy at kali in /home/s3cureroy
╰─⠠⠵ sudo make && make install
~~~

Una vez terminado, se puede reiniciar el equipo, o se puede cargar el *firmware* directamente mediante ```sudo modprobe 8818eu```

> Se puede comprobar que el dispositivo vuelve a tener un *driver* cargado, porque vuelven a aparece las redes disponibles.
{: .prompt-info}

## Activar modo monitor.

Con el *firmware* cargado correctamente, se vuelve a intentar activar nuevamente el modo *Monitor* en la interfaz con los pasos anteriores. 

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─⠠⠵ sudo airmon-ng start wlan0

Found 2 processes that could cause trouble.
Kill them using 'airmon-ng check kill' before putting
the card in monitor mode, they will interfere by changing channels
and sometimes putting the interface back in managed mode

    PID Name
    526 NetworkManager
   6336 wpa_supplicant

PHY     Interface       Driver          Chipset

phy0    wlan0           8188eu          TP-Link TL-WN722N v2/v3 [Realtek RTL8188EUS]
                (monitor mode enabled)
~~~

Como informa el mensaje anterior, se deben cerrar algunos procesos que pueden interferir en los cambios de canal, para ello, se ejecuta el comando que indica, indicando que se ha "*matado*" el proceso correctamente.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy
╰─⠠⠵ sudo airmon-ng check kill 

Killing these processes:

    PID Name
   6336 wpa_supplicant
~~~

Se lanza de nuevo el comando que activa el modo *Monitor*, aunque éste, había sido activado previamente, viendo que esta vez, no arroja ningún aviso.

~~~zsh
╭─s3cureroy at kali in /home/s3cureroy 
╰─⠠⠵ sudo airmon-ng start wlan0


PHY     Interface       Driver          Chipset

phy0    wlan0           8188eu          TP-Link TL-WN722N v2/v3 [Realtek RTL8188EUS]
                (monitor mode enabled)
~~~

Si se ejecuta de nuevo el comando ```iwconfig``` para verificar la configuración de la interfaz, se observa, que el modo *Monitor* se encuentra activo.

![Modo monitor](https://i.imgur.com/l9mnof4.png)

Ya se puede realizar capturas de paquetes para intentar capturar algún *handshake*.

Espero que sea de ayuda, ¡hasta la próxima!

