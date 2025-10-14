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

![](/assets/images/thm-writeup-colddbox/colddbox_logo.png)

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

Ahora que tenemos las credenciales vamos a ingresarlas en el panel de login de WordPress.

![](/assets/images/htb-writeup-delivery/helpdesk1.png)

¡Bien! Ya estamos dentro del panel de administración del sitio web.

![](/assets/images/htb-writeup-delivery/helpdesk2.png)

Si desde el menú de la izquierda accedemos a '*Appearance*' y '*Editor*' nos salen una lista de archivos template.

![](/assets/images/htb-writeup-delivery/mm2.png)

Entre tantos archivos podemos encontrar un '404.php' el cual podemos tratar de explotar cambiando el contenido del archivo por una reverse shell de pentestmonkey.

![](/assets/images/htb-writeup-delivery/mm3.png)

Desde consola nos ponemos en escucha por el puerto que hayamos proporcionado anteriormente.

![](/assets/images/htb-writeup-delivery/mm4.png)

Ya tenemos todo preparado para obtener una shell inversa, desde el navegador ingresaremos al fichero .php cuyo contenido trae un payload, a este archivo se accede desde la ruta `10.10.10.10/wp-content/themes/twentyfifteen/404.php`.

![](/assets/images/htb-writeup-delivery/mm5.png)

Si se te queda cargando, vas por buen camino, no toques nada, ahora volvemos a la consola y podemos ver como ya estamos dentro del servidor.

![](/assets/images/htb-writeup-delivery/mm6.png)

Con el usuario en el que nos encontramos no tenemos privilegios ni si quiera para leer la primera flag.

![](/assets/images/htb-writeup-delivery/mm7.png)

Necesitamos escalar privilegios a 'c0ldd', investigando cada una de las carpetas y archivos encontramos un archivo de configuración de WordPress dentro de '/var/www/html/'. Si leemos el archivo wp-config.php vemos la contraseña del usuario 'c0ldd'.

![](/assets/images/htb-writeup-delivery/mm7.png)

Vamos a probar a acceder al otro usuario desde SSH.

![](/assets/images/htb-writeup-delivery/mm7.png)

Ahora ya estamos en otro usuario y podemos acceder a recursos que anteriormente no podiamos, vamos a leer la primera flag.

![](/assets/images/htb-writeup-delivery/mm7.png)

## Escalada de privilegios

Una vez hemos accedido a un usuario del sistema y conseguimos la flag de usuario, podemos ir a por el usuario root, podemos probar desde buscar permisos SUID a binarios con permisos SUDO.

![](/assets/images/htb-writeup-delivery/user.png)

Con `sudo -l` podemos ver que binarios o scripts podemos ejecutar como administrador, el problema es que hay algunos binarios que pueden ser explotados por un atacante.

![](/assets/images/htb-writeup-delivery/user.png)

Nos salen varios binarios, desde la web 'https://gtfobins.github.io/' podemos buscar cada binario y ver si son explotables, en este caso vamos a utilizar el binario VIM para escalar privilegios.

![](/assets/images/htb-writeup-delivery/user.png)

## Credentials in MySQL database

After doing some recon we find the MatterMost installation directory in `/opt/mattermost`:

```
maildeliverer@Delivery:/opt/mattermost/config$ ps waux | grep -i mattermost
matterm+   741  0.2  3.3 1649596 135112 ?      Ssl  20:00   0:07 /opt/mattermost/bin/mattermost
```

The `config.json` file contains the password for the MySQL database:

```
[...]
"SqlSettings": {
        "DriverName": "mysql",
        "DataSource": "mmuser:Crack_The_MM_Admin_PW@tcp(127.0.0.1:3306)/mattermost?charset=utf8mb4,utf8\u0026readTimeout=30s\u0026writeTimeout=30s",
[...]
```

We'll connect to the database server and poke around.

```
maildeliverer@Delivery:/$ mysql -u mmuser --password='Crack_The_MM_Admin_PW'
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 91
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mattermost         |
+--------------------+
```

MatterMost user accounts are stored in the `Users` table and hashed with bcrypt. We'll save the hashes then try to crack them offline.

```
MariaDB [(none)]> use mattermost;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mattermost]> select Username,Password from Users;
+----------------------------------+--------------------------------------------------------------+
| Username                         | Password                                                     |
+----------------------------------+--------------------------------------------------------------+
| surveybot                        |                                                              |
| c3ecacacc7b94f909d04dbfd308a9b93 | $2a$10$u5815SIBe2Fq1FZlv9S8I.VjU3zeSPBrIEg9wvpiLaS7ImuiItEiK |
| 5b785171bfb34762a933e127630c4860 | $2a$10$3m0quqyvCE8Z/R1gFcCOWO6tEj6FtqtBn8fRAXQXmaKmg.HDGpS/G |
| root                             | $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO |
| snowscan                         | $2a$10$spHk8ZGr54VWf4kNER/IReO.I63YH9d7WaYp9wjiRswDMR.P/Q9aa |
| ff0a21fc6fc2488195e16ea854c963ee | $2a$10$RnJsISTLc9W3iUcUggl1KOG9vqADED24CQcQ8zvUm1Ir9pxS.Pduq |
| channelexport                    |                                                              |
| 9ecfb4be145d47fda0724f697f35ffaf | $2a$10$s.cLPSjAVgawGOJwB7vrqenPg2lrDtOECRtjwWahOzHfq1CoFyFqm |
+----------------------------------+--------------------------------------------------------------+
8 rows in set (0.002 sec)
```

## Cracking with rules

There was a hint earlier that some variation of `PleaseSubscribe!` is used.

I'll use hashcat for this and since I don't know the hash ID for bcrypt by heart I can find it in the help.

```
C:\bin\hashcat>hashcat --help | findstr bcrypt
   3200 | bcrypt $2*$, Blowfish (Unix)                     | Operating System
```

My go-to rules is normally one of those two ruleset:

- [https://github.com/NSAKEY/nsa-rules/blob/master/_NSAKEY.v2.dive.rule](https://github.com/NSAKEY/nsa-rules/blob/master/_NSAKEY.v2.dive.rule)
- [https://github.com/NotSoSecure/password_cracking_rules/blob/master/OneRuleToRuleThemAll.rule](https://github.com/NotSoSecure/password_cracking_rules/blob/master/OneRuleToRuleThemAll.rule)

These will perform all sort of transformations on the wordlist and we can quickly crack the password: `PleaseSubscribe!21`

```
C:\bin\hashcat>hashcat -a 0 -m 3200 -w 3 -O -r rules\_NSAKEY.v2.dive.rule hash.txt wordlist.txt
[...]
$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO:PleaseSubscribe!21

Session..........: hashcat
Status...........: Cracked
Hash.Name........: bcrypt $2*$, Blowfish (Unix)
[...]
```

The root password from MatterMost is the same as the local root password so we can just su to root and get the system flag.

![](/assets/images/htb-writeup-delivery/root.png)
