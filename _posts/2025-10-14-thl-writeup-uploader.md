<img width="1939" height="952" alt="image" src="https://github.com/user-attachments/assets/11f12598-f221-4bf5-9d57-a516d1287167" /><img width="1923" height="946" alt="image" src="https://github.com/user-attachments/assets/57362ecb-4e8d-4f08-9b9b-38d0acbac411" />---
layout: single
title: "Uploader: Easy - TheHackersLabs"
excerpt: "Uploader, un sitio web el cual ofrece almacenamiento en la nube a cualquier usuario que acceda a ella sin necesidad de registrarse previamente, no solo no saben hacer páginas web, si no que además ponen en peligro a los usuarios que utilicen su servicio. Es hora de darles una lección."
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

Vamos a probar a acceder al otro desde www-data con el comando 'su operatorx'.

![](/assets/images/thl-writeup-uploader/17.png)

Ahora ya estamos en otro usuario y podemos acceder a recursos que anteriormente no podiamos, vamos a leer la primera flag.

![](/assets/images/thl-writeup-uploader/18.png)

## Escalada de privilegios

Una vez hemos accedido al usuario y conseguimos la flag de usuario, debemos ir a por el usuario root, podemos probar desde buscar permisos SUID a binarios con permisos SUDO, también existen distintas técnicas más pero no se va a dar el caso en esta máquina.

Probamos con `sudo -l` y podemos observar que binarios o scripts podemos ejecutar como administrador.

![](/assets/images/thl-writeup-uploader/19.png)

Hay un binario el cual podemos ejecutar como root, desde la web 'https://gtfobins.github.io/' podemos buscar el binario, en este caso TAR si es un binario el cual podemos aprovechar para elevar privilegios.

![](/assets/images/thl-writeup-uploader/20.png)

Ahora que somos root podemos leer la flag que nos quedaba dentro de su usuario.

![](/assets/images/thl-writeup-uploader/21.png)
