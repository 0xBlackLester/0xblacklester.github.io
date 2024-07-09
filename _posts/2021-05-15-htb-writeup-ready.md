---
layout: single
title: Ready - Hack The Box
excerpt: "Ready was a pretty straighforward box to get an initial shell on: We identify that's it running a vulnerable instance of Gitlab and we use an exploit against version 11.4.7 to land a shell. Once inside, we quickly figure out we're in a container and by looking at the docker compose file we can see the container is running in privileged mode. We then mount the host filesystem within the container then we can access the flag or add our SSH keys to the host root user home directory."
date: 2021-05-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-ready/ready_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
tags:
  - linux
  - apache
  - cve
  - privesc
  - easy
---

![](/assets/images/htb-writeup-ready/ready_logo.png)

Ready was a pretty straighforward box to get an initial shell on: We identify that's it running a vulnerable instance of Gitlab and we use an exploit against version 11.4.7 to land a shell. Once inside, we quickly figure out we're in a container and by looking at the docker compose file we can see the container is running in privileged mode. We then mount the host filesystem within the container then we can access the flag or add our SSH keys to the host root user home directory.

## Escaneo de puertos (NMAP)

```
# Nmap 7.80 scan initiated Tue Jul  9 11:33:17 2024 as: nmap -sCV -p- --open -vvv -oN allports.txt 10.10.11.23
Nmap scan report for 10.10.11.23
Host is up, received syn-ack (0.046s latency).
Scanned at 2024-07-09 11:33:18 CEST for 23s
Not shown: 65533 closed ports
Reason: 65533 conn-refused
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://permx.htb
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Sitio web

Ahora que ya conocemos los puertos abiertos que se ejecutan pasaremos a inspeccionar el sitio web, apartados, subapartados, etc. Es importante destacar que para acceder al sitio web debemos añadir la IP al fichero ```/etc/hosts```

![](/assets/images/htb-writeup-ready/index.png)

Personalmente no he encontrado nada en la página web y código fuente.

## Descubrimiento de directorios y subdominios

Siguiendo por la fase de reconocimiento utilizaremos ```WFUZZ``` para enumerar los directorios del dominio.

![](/assets/images/htb-writeup-ready/fuzzing1.png)
