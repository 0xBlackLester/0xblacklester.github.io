---
layout: single
title: Grep - TryHackMe
excerpt: "Grep es un sitio web en desarrollo el cual tenemos que tratar de vulnerar utilizando técnicas como OSINT. Se trata de un CaptureTheFlag nivel fácil, algo a destacar de este CTF es que no necesitaremos escalar privilegios para completar el ejercicio. Vamos a utilizar técnicas de escaneo de puertos, fuzzing web, descubrimientos de dominios manual y Remote File Inclusion (RFI) para ganar acceso desde la linea de comandos al servidor remoto."
date: 2024-05-19
classes: wide
header:
  teaser: /assets/images/htb-writeup-delivery/delivery_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - osticket
  - mysql
  - mattermost
  - hashcat
  - rules
---

![](/assets/images/htb-writeup-delivery/delivery_logo.png)
Como cualquier otro reto, comenzaremos con la fase de reconocimiento, para ellos utilizaremos la herramienta nmap para escanear los scripts/versiones y puertos abiertos del servidor web.

## Portscan

```
Nmap scan report for 10.10.250.61
Host is up, received syn-ack (0.083s latency).
Scanned at 2024-05-19 16:13:50 CEST for 79s
Not shown: 60869 closed ports, 4662 filtered ports
Reason: 60869 conn-refused and 4662 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE  REASON  VERSION
22/tcp    open  ssh      syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     syn-ack Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
443/tcp   open  ssl/http syn-ack Apache httpd 2.4.41
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
| Issuer: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-06-14T13:03:09
| Not valid after:  2024-06-13T13:03:09
| MD5:   7295 8ef0 7c16 221c 3b0a 40ee 913c 766c
| SHA-1: 38c2 3ba3 34b1 851a f1d4 ee0a 37bd 701a 830c 7dd8
| -----BEGIN CERTIFICATE-----
| MIIDFzCCAf8CFGTWwbbVKaNSN8fhUdtf0QT84zCSMA0GCSqGSIb3DQEBCwUAMEgx
| CzAJBgNVBAYTAlVTMRMwEQYDVQQIDApTb21lLVN0YXRlMREwDwYDVQQKDAhTZWFy
| Y2hNRTERMA8GA1UEAwwIZ3JlcC50aG0wHhcNMjMwNjE0MTMwMzA5WhcNMjQwNjEz
| MTMwMzA5WjBIMQswCQYDVQQGEwJVUzETMBEGA1UECAwKU29tZS1TdGF0ZTERMA8G
| A1UECgwIU2VhcmNoTUUxETAPBgNVBAMMCGdyZXAudGhtMIIBIjANBgkqhkiG9w0B
| AQEFAAOCAQ8AMIIBCgKCAQEAtiDNwwY9IR2HADMy6CRAwiPH0s8dIOFGPrbYCbLz
| fDKIWURlczzOlmgpscN/YHHpt6P5ywUPLGnMK3ukYag7xTUYl+vmledTnD9oebnJ
| 6qDweFFwdZ8hysITyvCyGgqcY52JE2nBtVNj6/L16iZ60KKko8opNsTE5IYj/sUt
| PsOxeNiV3oqpOUeKtZJbn7Kssd4KBwnRqTSUlXlPXzeRipAiW5SZZXo6K4YeLVht
| XlLPtPWsMC0fj16DDDtxLlZmvu3J5o9egp/eRpWmvKWIaKQ57Y0MKB8/gso8FxxX
| NiRY9Nru0C3DCUbc/xXywQ9pIGt/Xir++aXhyxCiIGh22QIDAQABMA0GCSqGSIb3
| DQEBCwUAA4IBAQCzhJu52dIY7V/qQleDMEQ1oBLrQoFhHD6+UbvH0ELMAtL5Dc8A
| LGDdyFkgsx04TaZtJ20dyrjYD+tcAgu9Yb7eEYbfqqD5w4XSzvdEuTW2aVL86aT6
| IBbN8SMkX2zfILjHTOR1F7WAoHaIssH0yZltg+lQEEnAeb+XoIZm9cIW2bTNKoO2
| MeHgvSKkQkjROO29XQQ3mTbxFG86UsTwyGHdddnkfiWilXqgfh+wGxbY/wCdhU0C
| TnuXn4IEVdCBn16rCg51kEZZC1EWPcJpv0/InUNfcgumcVY033EXF/HgW4eNDD6H
| XmLEGKfScUWcO0//STDZGZXwf9gt30DqoMSf
|_-----END CERTIFICATE-----
| tls-alpn: 
|_  http/1.1
51337/tcp open  ssl/http syn-ack Apache httpd 2.4.41
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: 400 Bad Request
| ssl-cert: Subject: commonName=leakchecker.grep.thm/organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Issuer: commonName=leakchecker.grep.thm/organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2023-06-14T12:58:31
| Not valid after:  2024-06-13T12:58:31
| MD5:   d9a2 b1f1 2ddd d1d5 f54f 98f4 d797 fd6a
| SHA-1: 3f70 53de 354b d41c 1d09 00b3 b603 3c55 b0dc 1390
| -----BEGIN CERTIFICATE-----
| MIIDTzCCAjcCFCzf/mtdaBGiKKpO7gdtpdVG9u6iMA0GCSqGSIb3DQEBCwUAMGQx
| CzAJBgNVBAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRl
| cm5ldCBXaWRnaXRzIFB0eSBMdGQxHTAbBgNVBAMMFGxlYWtjaGVja2VyLmdyZXAu
| dGhtMB4XDTIzMDYxNDEyNTgzMVoXDTI0MDYxMzEyNTgzMVowZDELMAkGA1UEBhMC
| QVUxEzARBgNVBAgMClNvbWUtU3RhdGUxITAfBgNVBAoMGEludGVybmV0IFdpZGdp
| dHMgUHR5IEx0ZDEdMBsGA1UEAwwUbGVha2NoZWNrZXIuZ3JlcC50aG0wggEiMA0G
| CSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC2IM3DBj0hHYcAMzLoJEDCI8fSzx0g
| 4UY+ttgJsvN8MohZRGVzPM6WaCmxw39gcem3o/nLBQ8sacwre6RhqDvFNRiX6+aV
| 51OcP2h5ucnqoPB4UXB1nyHKwhPK8LIaCpxjnYkTacG1U2Pr8vXqJnrQoqSjyik2
| xMTkhiP+xS0+w7F42JXeiqk5R4q1klufsqyx3goHCdGpNJSVeU9fN5GKkCJblJll
| ejorhh4tWG1eUs+09awwLR+PXoMMO3EuVma+7cnmj16Cn95Glaa8pYhopDntjQwo
| Hz+CyjwXHFc2JFj02u7QLcMJRtz/FfLBD2kga39eKv75peHLEKIgaHbZAgMBAAEw
| DQYJKoZIhvcNAQELBQADggEBAJIlfMmC0KqBPG7/54bkNknBCo+z6ck1oAOmHqrj
| IPUSCvSomgP/wuXuzOlspp9Qta8hA3DM+L0Q4/jE5Jt+IXU2TeBgvFsZx3IJGipf
| /LyO8C2MzoKWXO3CwP8WIREzCckaSZIXsrBMixWzKoGnLxl/zvYmhM00C8aJ/8cf
| 3gsVcFtiuudzfctad7a5gzcSGLhkATZTcuFDto3uzC4LszSXr8WiFBcSGbwyBkY9
| VZOfpN/kbjmxZIUXovF9BwHY9GWbDauuAkyCD4caUbbq7wdfTFH0A8lSw0ggzvzK
| hn9+V7DBwrPr4Y/gdkrUP6zWMZWnPybIYsmvbo2aKi9UJ+8=
|_-----END CERTIFICATE-----
| tls-alpn: 
|_  http/1.1
Service Info: Host: ip-10-10-250-61.eu-west-1.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Website

The Delivery website is pretty basic, there's a link to a vhost called helpdesk.delivery.htb and a contact us section. We'll add this entry to our local host before proceeding further.

![](/assets/images/htb-writeup-delivery/website1.png)

The contact us section tells us we need an @delivery.htb email address and tells us port 8065 is a MatterMost server. MatterMost is a Slack-like collaboration platform that can be self-hosted.

![](/assets/images/htb-writeup-delivery/website2.png)

Browsing to port 8065 we get the MatterMost login page but we don't have credentials yet

![](/assets/images/htb-writeup-delivery/mm1.png)

## Helpdesk

The Helpdesk page uses the OsTicket web application. It allows users to create and view the status of ticket.

![](/assets/images/htb-writeup-delivery/helpdesk3.png)

We can still open new tickets even if we only have a guest user.

![](/assets/images/htb-writeup-delivery/helpdesk1.png)

After a ticket has been created, the system generates a random @delivery.htb email account with the ticket ID.

![](/assets/images/htb-writeup-delivery/helpdesk2.png)

Now that we have an email account we can create a MatterMost account.

![](/assets/images/htb-writeup-delivery/mm2.png)

A confirmation email is then sent to our ticket status inbox.

![](/assets/images/htb-writeup-delivery/mm3.png)

We use the check ticket function on the OsTicket application and submit the original email address we used when creating the ticket and the ticket ID.

![](/assets/images/htb-writeup-delivery/mm4.png)

We're now logged in and we see that the MatterMost confirmation email has been added to the ticket information.

![](/assets/images/htb-writeup-delivery/mm5.png)

To confirm the creation of our account we'll just copy/paste the included link into a browser new tab.

![](/assets/images/htb-writeup-delivery/mm6.png)

After logging in to MatterMost we have access to the Internal channel where we see that credentials have been posted. There's also a hint that we'll have to use a variation of the `PleaseSubscribe!` password later.

![](/assets/images/htb-writeup-delivery/mm7.png)

## User shell

With the `maildeliverer / Youve_G0t_Mail!` credentials we can SSH in and get the user flag.

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
