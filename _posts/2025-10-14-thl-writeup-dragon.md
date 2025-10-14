---
layout: single
title: "Dragon: Easy - TheHackersLabs"
excerpt: "Dragon, así es el nombre de una página web la cual nos pone un simple reto: comprometer su sistema. Ellos mismos saben que han contratado desarrolladores principiantes, por lo que no deberían confiarse, es hora de ponernos manos a la obra y demostrar de que estamos hechos."
date: 2025-08-14
classes: wide
header:
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TheHackersLabs
tags:  
  - Easy
  - BruteForce
  - eJPTv2
  - PrivEsc
---

![](/assets/images/thl-writeup-dragon/banner.png)

En esta máquina utilizaremos técnicas de enumeración y ataques de fuerza bruta para extraer contraseñas de determinados servicios, una vez hayamos logrado acceder a la máquina tendremos que escalar privilegios al usuario root. Esta escalada se va a realizar de forma rápida ya que apenas utilizando técnicas sencillas de escalada de privilegios lograremos acceder como root.

## Escaneo de puertos

```
# Nmap 7.94SVN scan initiated Tue Oct 14 01:00:11 2025 as: nmap -p- --open -sS -sCV --min-rate=5000 -n -Pn -vvv -oN allports.txt 192.168.1.241
Nmap scan report for 192.168.1.241
Host is up, received arp-response (0.0027s latency).
Scanned at 2025-10-14 01:00:12 CEST for 42s
Not shown: 41330 filtered tcp ports (no-response), 24203 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:98:c6:f1:55:e6:30:8b:83:c4:69:60:d9:ed:11:4d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLRB0bNFpMig2oGPoN2EWsh1Ximm6bDgZu/Z9O0twiunyN9X/pMOAC2J9gxyQYQwRu7ey4QtLD4qSFx9PMW1mWc=
|   256 b5:d2:46:75:32:b0:98:b2:8f:61:02:95:cf:ba:19:c6 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOmNTndfAjNjQW4vXgoZ0sV+DLTbr9TdMa0mYQDPsstr
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: La M\xC3\xA1quina del Drag\xC3\xB3n
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 08:00:27:F0:F3:DB (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Oct 14 01:00:54 2025 -- 1 IP address (1 host up) scanned in 43.22 seconds
```

## Página web

La web no muestra ningún menú desplegable ni apartados donde podamos navegar a simple vista con la página, lo que podemos observar es el texto que nos muestra por pantalla, nos deja caer que hay algun directorio/archivo oculto y que la forma de navegar por la web no siempre es como esta acostumbrado un simple usuario.

![](/assets/images/thl-writeup-dragon/1.png)

Podemos aplicar fuzzing para encontrar directorios ocultos, utilizando la herramienta gobuster encontramos '/secret'.

![](/assets/images/thl-writeup-dragon/2.png)

Accedemos a '/secret' y a simple vista no encontramos nada interesante, pero si leemos el texto que muestra dice que intentemos sin pausa las llaves del dragon, además aclara que el mensaje va dedicado a Dragon, me dió a entender que está hablando de un ataque de fuerza bruta.

![](/assets/images/thl-writeup-dragon/3.png)

## Hydra - Fuerza bruta

A pesar de que intenté encontrar más directorios o archivos dentro del directorio '/secret' no logré encontrar ninguno mas, pero teniendo una posible pista y sabiendo que un posible usuario se llama dragon, intenté realizar un ataque de fuerza bruta con hydra, así resultando exitoso.

![](/assets/images/thl-writeup-dragon/4.png)

## Accedemos al usuario

Una vez que ya tenemos las credenciales del servicio SSH probamos a conectarnos y como se puede observar, accedemos sin ningún tipo de restricción.

![](/assets/images/thl-writeup-dragon/5.png)

Ahora tenemos que conseguir la flag del usuario, normalmente se encuentra en el directorio propio del usuario con el nombre de '*user.txt*'.

![](/assets/images/thl-writeup-dragon/6.png)

## Escalada de privilegios

Hemos obtenido acceso al sistema como usuario sin privilegios, ya hemos dado un gran paso, pero queda lo importante, que es acceder como usuario root para poder tener permisos elevados los cuales nos permiten funciones que un usuario sin privilegios no podría. Una simple busqueda de binarios con permisos SUDO con `sudo -l` nos muestra el editor de texto 'vim'. A pesar de haber encontrado esta puerta traté de buscar permisos SUID para hacer el ataque un poco más interesante, esta busqueda no tuvo éxito.

![](/assets/images/thl-writeup-dragon/7.png)

Existe una herramienta web (*gtfobins.github.io*) la cual nos permite verificar si un binario es vulnerable a determinados permisos, en este caso podemos comrpobar que 'vim' si tiene un apartado de SUDO, es decir, nos sirve para nuestra escalada de privilegios.

![](/assets/images/thl-writeup-dragon/8.png)

Si copiamos el comando que nos proporciona y lo pegamos en la terminal ganaremos acceso como root.

![](/assets/images/thl-writeup-dragon/9.png)

Una vez somos root podemos movernos libremente por el sistema, a nosotros nos interesa conseguir la flag de root la cual se almacena dentro de la carpeta '/root/'.

![](/assets/images/thl-writeup-dragon/10.png)
