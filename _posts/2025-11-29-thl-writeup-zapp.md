---
layout: single
title: "ZAPP: Easy - TheHackersLabs"
excerpt: "En desarrollo..."
date: 2025-08-14
classes: wide
header:
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TheHackersLabs
tags:
  - Easy
---

![](/assets/images/thl-writeup-ZAPP/banner.png)

En desarrollo...

## Escaneo de puertos

```
# Nmap 7.94SVN scan initiated Sat Nov 29 21:31:15 2025 as: nmap -sCV -p- --open -vvv -oN allports.txt 192.168.1.188
Nmap scan report for 192.168.1.188
Host is up, received syn-ack (0.00098s latency).
Scanned at 2025-11-29 21:31:15 CET for 17s
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              28 Oct 29 20:59 login.txt
|_-rw-r--r--    1 0        0              65 Oct 29 21:23 secret.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.239
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey: 
|   3072 a3:23:b3:aa:df:c6:51:cb:a2:0c:92:8e:6b:fe:96:ee (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC+S08UpCAAkfkyQOXYhnapAf8NrUa2NloM2dzeUSjsTixJ7qJM3FhoHN/5rvBJb98svP7Rs8V4x6A2axiiD64mUsfNTX6GC5JemJIWole/yyW4Uulo0rKaHdCvHOgOlWFphHtU+ZklG5sIqtJRHaZy5xiXZnVy2OalnBv0Exvby2h6ARvMlQys020yZAVYVXh2Dp0Kk2XHrDNljvQPikGjo6deC8gUENqGSHlYKax6FFK8+6qTcMYDzkcQJVo9+I/6u+EE9EiPk+hm/mVh9x3Cd/F01GcRp5QHGkHma3vKE7vEIlDTZS0Ha5PJYFAq8AjvZHdbKcBcONlja8jQ1gwTu+nrtmpOsM2uaAYslH4i5D6OedWEVLktNELBbC+AywdWcHzvHw1mQdjZCNYHY3o+w8V3PV8u9wiH4JVf4GFYjKWwQ++6flQnUcpXtMGfn1y0fY6AU9FZXUQaRHKNV8rf7K1eWoNnauf7QO7to84KOtotjAa5vQvjR18i5AQd8a8=
|   256 fd:95:2f:2f:7f:5a:21:b5:0e:75:2c:da:18:c9:52:35 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJIzG68W0Lc1RVYcz4gQi1bKaj/Ur/r63yiySE/FV55MYADfcgmo5LQa4m5LnHzBWRdkN5RLwqqXqSXZSkzd56E=
|   256 a1:0e:0d:79:8e:54:3e:0e:ed:2f:96:d6:d3:9a:9f:a6 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINS0D9TRt7d/C319AC8Q7AILAzjw1jn0a64IticO1vDt
80/tcp open  http    syn-ack Apache httpd 2.4.65 ((Debian))
|_http-title: zappskred - CTF Challenge
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.65 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Nov 29 21:31:32 2025 -- 1 IP address (1 host up) scanned in 17.09 seconds
```

Para nuestra sorpresa, el servicio FTP se encuentra configurado para que podamos iniciar sesión sin credenciales (anonymous/anonymous).

## Exploración FTP

En lo personal, siempre me gusta realizar un escaneo de puertos y acto seguido visualizar el contenido de la página web, pero en este caso como he visto que puedo acceder con credenciales anónimas al FTP voy a hechar un vistazo.

![](/assets/images/thl-writeup-ZAPP/1.png)

Dentro del FTP habían 2 ficheros misteriosos los cuales los bajé a mi equipo para poder leer su contenido, a simple vista tenemos lo siguiente:

• login.txt
• secret.txt

![](/assets/images/thl-writeup-ZAPP/2.png)

## Página web

Los ficheros me dieron falsas pistas que en lo personal no me ayudaron hasta este entonces, luego quizás podremos enlazar las pistas. Ahora si que podemos pasar a ver la página web y como se encuentra estructurada.

![](/assets/images/thl-writeup-ZAPP/3.png)

Como se puede observar en la imagen anterior la página a simple vista es muy sencilla, pero si accedemos a su código fuente podemos ver en la linea 180 un número de puerto y una cadena en Base64.

![](/assets/images/thl-writeup-ZAPP/4.png)

Vamos a decodear la cadena 