---
layout: post
title: Usar dos interfaces de red para separar la comunicación de una máquina virtual y un sistema anfitrión.
autor: S3cureroy
date: 2022-12-02
categories: [Tutoriales, Virtualización]
tags: [windows, virtualbox, powershell, red]
---
## Introducción.

Cuando se trabaja como analista de seguridad, a diario se necesita hacer ciertas pruebas, o contactar con algunos dominios, potencialmente maliciosos, para esclarecer si se está sufriendo un incidente o no. 

Para evitar generar alertas adicionales realizando las pruebas, o que se bloquee la conexión en el *proxy* o *firewall* se decide usar un ordenador portátil que disponga de interfaz cableada y de interfaz Wi-fi, conectando la máquina anfitrión a la red cableada con fines corporativos, usando una máquina virtual conectada a la red Wi-fi.

En este caso, se dispone de un acceso Wi-fi, que conecta con otra red diferente, la cual no tiene los mecanismos de defensa que dispone la red corporativa a la que se está conectado mediante el cable de red.

Virtualbox, el conocido software de virtualización, dispone de muchas configuraciones posibles para adaptar a las necesidades de cada usuario el tipo de red que requiera.

Estas configuraciones son las siguientes:
* NAT: Es usada para que la máquina virtual utilice la misma dirección IP que usa la máquina anfitrión.
* Bridge: En la red, la máquina virtual obtendría (si se tiene activado el protocolo DHCP) una dirección de la misma red donde se encuentre conectada la interfaz del equipo anfitrión.
* Red Interna: Permite crear una red donde conectar diferentes máquinas virtuales entre ella, pero sin contacto con "el exterior".
* *Host-only*: Mediante esta configuración, se puede hacer que una máquina virtual contacte unicamente con la máquina anfitrión.

![diagrama de red](https://i.imgur.com/K8eXFzF.png)

El mayor problema presentado, es que si se conectan ambas interfaces a la vez, la gestión de las conexiones que hace el sistema Windows de la máquina anfitrión no es del todo satisfactoria.

## Procedimiento manual. ##

Por ello, la manera sencilla para configurar el entorno, es la siguiente:

1. Conectar la máquina virtual a una red "puente", y configurar la interfaz Wi-fi.
2. Acceder en el sistema anfitrión a las Opciones de adaptador y presionar en la interfaz Wi-fi con el botón derecho, pulsando sobre Propiedades después.
3. En este punto, se deben desmarcar los checks pertenecientes a los componentes a "Protocolo de Internet versión 4 (TCP/IPv4)" y "Protocolo de Internet versión 6 (TCP/IPv6)".
4. Cuando se realice la deshabilitación ya se puede conectar a la red Wi-fi en el sistema anfitrión, y en caso de que la red use el protocolo DHCP se observará que la máquina virtual obtiene la IP.


## Ahora mediante terminal. ##

Conociendo el procedimiento, se puede realizar esto cada vez que se necesite usar dos interfaces a la vez, pero en caso que sólo se tenga que usar la red Wi-fi, ésta, no obtendrá IP, por lo que se debería hacer el procedimiento inverso, cada vez que se necesite cambiar.

Esto puede resultar muy tedioso, y por ello, se puede automatizar mediante powershell.

Para ello, se pueden usar los siguientes comandos:

* Obtener las interfaces de red y el estado de los componentes de los protocolos IPv4.

~~~
PS C:\> Get-NetAdapterBinding -ComponentID ms_tcpip

Name                           DisplayName                                        ComponentID          Enabled
----                           -----------                                        -----------          -------
VMware Network Adapte...1      Protocolo de Internet versión 4 (TCP/IPv4)         ms_tcpip             True
VMware Network Adapte...8      Protocolo de Internet versión 4 (TCP/IPv4)         ms_tcpip             True
Ethernet 2                     Protocolo de Internet versión 4 (TCP/IPv4)         ms_tcpip             True
Wi-Fi                          Protocolo de Internet versión 4 (TCP/IPv4)         ms_tcpip             True
~~~
> Para realizar esta consulta con el protocolo IPv6, sólo se debe cambiar el componenten por **ms_tcpip6**.
{: .prompt-info} 

* Cambiar el estado del componente a "*Disabled*" y por tanto, retirar la posibilidad de obtener IP.

~~~
PS C:\> Disable-NetAdapterBinding -ComponentID ms_tcpip -Name "nombre interfaz"
~~~

* Habilitar el protocolo IP en la interfaz que se necesite.

~~~
PS C:\> Enable-NetAdapterBinding -ComponentID ms_tcpip -Name "nombre interfaz"
~~~

> Tanto el comando de habilitación, como el de deshabilitación, no devolverán un mensaje informativo, por lo que se debe volver a verificar el estado de los componentes para ver si se realizó correctamente.
{: .prompt-info} 


