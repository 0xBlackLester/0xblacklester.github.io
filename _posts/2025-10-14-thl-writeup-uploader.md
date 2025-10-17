---
layout: single
title: "Uploader: Easy - TheHackersLabs"
excerpt: "ColddBox, tomalo como una empresa la cual dispone de sitio web como la gran mayoria. Nosotros como antiguos clientes de la empresa no estamos conformes con la atención proporcionada, entonces... es hora de utilizar nuestras habilidades de Hacking Etico para comprometer el sitio web."
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

![](/assets/images/thl-writeup-uploader/banner.png)

En esta máquina estaremos viendo como obtenemos acceso inicial al servidor como www-data, gracias a una serie de archivos visibles por parte de este usuario podremos elevar los privilegios un nivel mas a un usuario con menores restricciones. Una vez encontremos la flag del usuario utilizaremos técnicas de PrivEsc para convertirnos en root y así poder leer la flag de root.

## Escaneo de puertos

```
# Nmap 7.94SVN scan initiated Mon Oct 13 15:37:49 2025 as: nmap -sCV -p- --open -vvv -oN allports.txt 192.168.1.238
Nmap scan report for 192.168.1.238
Host is up, received syn-ack (0.0013s latency).
Scanned at 2025-10-13 15:37:49 CEST for 13s
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON  VERSION
80/tcp open  http    syn-ack Apache httpd 2.4.58 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Uploader File Storage
|_http-server-header: Apache/2.4.58 (Ubuntu)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 13 15:38:02 2025 -- 1 IP address (1 host up) scanned in 13.05 seconds
```

## Página web

Como siempre comenzamos hechando un vistazo a la web para explorar sus funciones, directorios, incluso su codigo fuente por si encontramos comentarios del desarrollador, en este caso únicamente encontramos un apartado donde supuestamente nos deja subir archivos.

![](/assets/images/thl-writeup-uploader/1.png)

Este es el apartado en cuestión, mi intriga me llegó a comprobar si existe algún tipo de filtro a la hora de seleccionar y subir un archivo, en nuestro caso nos interesa un `.php` ya que de esta forma podríamos inyectar código malicioso y establecer una reverse shell con el servidor.

![](/assets/images/thl-writeup-uploader/2.png)

Utilizamos la plantilla de rshells de pentest monkey y lo modificamos para que apunte a nuestra dirección IP y puerto.

![](/assets/images/thl-writeup-uploader/3.png)

Subimos el archivo y... vualá, parece que nos ha dejado sin problema.

![](/assets/images/thl-writeup-uploader/4.png)

## Fuzzing

La captura anterior nos muestra que nuestro archivo ha sido almacenado en algún lugar el cual no nos muestra ni sabemos actualmente, es por ello que podemos aplicar fuzzing para encontrar directorios que no se encuentran a simple vista. Tengo que decir que al realizar el CTF probé manualmente el directorio '/uploads' y lo encontré sin necesidad de aplicar fuzzing, pero de todas formas lo ejecuté por si existian mas directorios o subdirectorios dentro de la carpeta '/uploads'.

![](/assets/images/thl-writeup-uploader/5.png)

Si accedemos al directorio '/uploads' podemos ver que se guarda nuestro archivo que acabamos de subir dentro de una carpeta, en mi caso tengo más carpetas por anteriores pruebas, cada carpeta corresponde a una determinada hora y día, por lo que es fácil de localizar.

![](/assets/images/thl-writeup-uploader/6.png)

## Accedemos al servidor

Ahora sabemos que nuestro archivo se encuentra almacenado en el servidor, dado que el servidor va a tratar de establecer una conexión con nuestro equipo atacante tenemos que ponernos en escucha con el comando `nc -lnvp 4444`.

![](/assets/images/thl-writeup-uploader/7.png)

Volvemos al sitio web, hacemos click sobre nuestro archivo y veremos que se ha establecido una conexión con el servidor.

![](/assets/images/thl-writeup-uploader/8.png)

Ya tenemos una shell del lado del servidor, pero somos www-data, podemos tratar la TTY para movernos con más facilidad sobre la shell.

![](/assets/images/thl-writeup-uploader/9.png)

Con un analisis manual de cada una de las carpetas que se encontraban en el directorio raiz encontré que dentro de la carpeta '*srv*' se encuentra otra carpeta llamada '*secret*', la cual si ingresamos en ella encontramos un archivo .zip.

![](/assets/images/thl-writeup-uploader/10.png)

Dentro de la máquina víctima no tenemos herramientas las cuales nos permitan descomprimir el archivo, pero si tenemos python instalado, por lo que podemos levantar un servidor en python para desde nuestra máquina atacante podamos descargar el archivo comprimido.

![](/assets/images/thl-writeup-uploader/11.png)

Ahora que tenemos el comprimido en nuestro equipo, podemos analizarlo. Si nos fijamos, con el comando `zipinfo -v File.zip` vemos que nos detecta 2 archivos, una carpeta la cual dice que está desencriptada, es decir, nos deja descomprimirla, y luego un fichero .txt el cual está encriptado, por lo cual no va a dejar descomprimirlo sin contraseña.

![](/assets/images/thl-writeup-uploader/12.png)

Si intentamos descomprimir el .zip nos pedirá una contraseña.

![](/assets/images/thl-writeup-uploader/13.png)

Una herramienta util para crackear contraseñas de los .zip es *zip2john*, el cual permite sacar el hash del comprimido y haciendo uso de *john* podemos crackear ese respectivo hash.

![](/assets/images/thl-writeup-uploader/14.png)

Ya tenemos la contraseña, ahora con el comando `7z x File.zip` descomprimimos el archivo poniendo la contraseña que acabamos de crackear. Ya podemos ver el fichero que antes estaba encriptado, si lo leemos vemos un usuario y un hash.

![](/assets/images/thl-writeup-uploader/15.png)

Este hash lo podemos poner en [https://crackstation.net]*crackstation.net* y nos devolverá la contraseña en texto plano.

![](/assets/images/thl-writeup-uploader/16.png)

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
