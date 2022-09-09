---
title: Hack The Box - Starting Point - Tier 1 - Crocodile Writeup
date: 2022-08-10 09:20:00 +0800
categories: ["Hack The Box"]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 1** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-10-htb-writeup-tier1-appointment %}).

## Crocodile

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

Copiamos la IP del equipo remoto, en mi caso *10.129.58.242*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A 10.129.58.242 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 20:32 CEST
Nmap scan report for 10.129.58.242
Host is up (0.049s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.xx.xx
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.70 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

OK. La salida nos cuenta que tenemos dos puertos abiertos con servicios escuchando:

* puerto *21/tcp* (FTP) con un *vsFTPd 3.0.3* detrás y lo más bonito: 

```
[...]
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
[...]
```

* puerto *80/tcp* (HTTP) con un *Apache/2.4.41 (Ubuntu)* detrás.

Aprovechamos el *ftp-anon* y salimos de pesca.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ ftp 10.129.58.242
Connected to 10.129.58.242.
220 (vsFTPd 3.0.3)
Name (10.129.58.242:ewan67): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||49464|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||43256|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |*************************************************************************************************************************|    33        0.42 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.17 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||41270|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |*************************************************************************************************************************|    62        1.66 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (0.25 KiB/s)
ftp>
```

Echamos un ojo a los ficheros que nos hemos descargado.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ cat allowed.userlist.passwd
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Una relación de credenciales que podemos expresar así:

```
aron:root
pwnmeow:Supersecretpassword1
egotisticalsw:@BaASD&9032123sADS
admin:rKXM59ESxesUFHAd
```

El FTP ha dado todo de sí. Abrimos un browser y apuntamos a la IP a ver qué nos cuenta el Apache del puerto 80.

![](/assets/posts/20220810/img09.png)

Bonita web. Trasteamos un poco con ella para conocerla mejor. Lanzamos un **whatweb** para más info.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB]
└─$ whatweb -a 3 -v 10.129.58.242
WhatWeb report for http://10.129.58.242
Status    : 200 OK
Title     : Smash - Bootstrap Business Template
IP        : 10.129.58.242
Country   : RESERVED, ZZ

Summary   : Apache[2.4.41], Bootstrap[3.3.7], Email[hello@ayroui.com,support@uideck.com], Frame, HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], JQuery[1.12.4], Modernizr[3.7.1.min], Script

Detected Plugins:
[ Apache ]
        The Apache HTTP Server Project is an effort to develop and
        maintain an open-source HTTP server for modern operating
        systems including UNIX and Windows NT. The goal of this
        project is to provide a secure, efficient and extensible
        server that provides HTTP services in sync with the current
        HTTP standards.

        Version      : 2.4.41 (from HTTP Server Header)
        Google Dorks: (3)
        Website     : http://httpd.apache.org/

[ Bootstrap ]
        Bootstrap is an open source toolkit for developing with
        HTML, CSS, and JS.

        Version      : 3.3.7
        Version      : 3.3.7
        Website     : https://getbootstrap.com/
[...]
```

Significado de las flags:

* `-a`&nbsp;: Set the aggression level. Default: 1. Nosotros le pasamos un 3 (Aggressive).
* `-v`&nbsp;: Verbose output includes plugin descriptions.

Bien. Interesante info (un par de cuentas de correo incluidas) pero poco donde tirar. Pasamos a enumerar directorios a ver qué aparece.

Lo primero, localizar en nuestra máquina una lista adecuada para utilizar de diccionario.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ locate directory-list
/usr/share/dirbuster/wordlists/directory-list-1.0.txt
/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
/usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
/usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
/usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-1.0.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt
/usr/share/whatweb/plugins/simple-directory-listing.rb
```

Nos quedamos con esta:

```
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
```

Para estas tareas suelo utilizar **feroxbuster** (personal y profesionalmente creo que esta herramienta es bestial), pero os dejo los comandos de **gobuster** y **ffuf** para los que quieran probar otras alternativas.

Atentos a las tags del box vamos a aprovechar el escaneo para pedirle a nuestras tools que busquen ficheros *.php*.

Lo prometido sobre **gobuster** y **ffuf**.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ gobuster dir -u http://10.129.58.242 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x .php
```

Significado de las flags:

* `dir`&nbsp;: Uses directory/file enumeration mode.
* `-u`&nbsp;: The target URL.
* `-w`&nbsp;: Path to the wordlist.
* `-x`&nbsp;: File extension(s) to search for.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ ffuf -u http://10.129.58.242/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -c -t 100 -e .php -o ffuf_output -of md
```

Significado de las flags:

* `-u`&nbsp;: la URL que queremos escanear.
* `-w`&nbsp;: la ubicación del diccionario.
* `-c`&nbsp;: para colorear la salida.
* `-t`&nbsp;: nro de hilos concurrentes (por defecto son 40).
* `-e`&nbsp;: listado de extensiones que queremos rasrear, separadas por comas en caso de ser más de una.
* `-o`&nbsp;: nombre del fichero donde almacenaremos la salida.
* `-of`&nbsp;: formato del fichero de salida, md en nuestro caso.

Vamos con **feroxbuster**. En aras de la sencillez, el volcado que pego a continuación esta editado para resaltar la información más relevante.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ feroxbuster -u http://10.129.58.242/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -e -t 100 -x php -o feroxbuster_output

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.58.242/
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💾  Output File           │ feroxbuster_output
 💲  Extensions            │ [php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
[...]
200      GET      999l     3031w    58565c http://10.129.58.242/index.html
[...]
200      GET       39l      115w     1577c http://10.129.58.242/login.php
302      GET        0l        0w        0c http://10.129.58.242/logout.php => login.php
200      GET        0l        0w        0c http://10.129.58.242/config.php
301      GET        9l       28w      316c http://10.129.58.242/dashboard => http://10.129.58.242/dashboard/
[...]
302      GET        0l        0w        0c http://10.129.58.242/dashboard/index.php => /login.php
[...]
[####################] - 2m    351170/351170  0s      found:112     errors:27
[####################] - 2m    175300/175300  1018/s  http://10.129.58.242/
[...]
[####################] - 2m    175300/175300  1080/s  http://10.129.58.242/dashboard
[...]
```

Significado de las flags:

* `-u`&nbsp;: la URL que queremos escanear.
* `-w`&nbsp;: la ubicación de nuestro fichero de wordlist.
* `-e`&nbsp;: le pedimos que extraiga enlaces del cuerpo de la respuesta y realice nuevas peticiones en función de los resultados.
* `-t`&nbsp;: nro de hilos concurrentes.
* `-x`&nbsp;: la extensión de los ficheros que queremos buscar.
* `-o`&nbsp;: nombre del fichero donde almacenaremos la salida.

Resumiendo:

```
200      GET       39l      115w     1577c http://10.129.58.242/login.php
302      GET        0l        0w        0c http://10.129.58.242/logout.php => login.php
301      GET        9l       28w      316c http://10.129.58.242/dashboard => http://10.129.58.242/dashboard/
302      GET        0l        0w        0c http://10.129.58.242/dashboard/index.php => /login.php
```

Una página *login.php* colgando del raiz y un directorio *dashboard*.

Cargamos la URL *http://10.129.58.242/login.php* en el navegador.

![](/assets/posts/20220810/img10.png)

OK. Utilizamos la relación de credenciales que conseguimos a partir del los ficheros del FTP y probamos con cada una. (Spoiler: es la última)

Accedemos al Dashboard y nos muestra la flag.

## Respuestas:

* <strong>Task 1</strong>: -sC
* <strong>Task 2</strong>: vsftpd 3.0.3
* <strong>Task 3</strong>: 230
* <strong>Task 4</strong>: get
* <strong>Task 5</strong>: admin
* <strong>Task 6</strong>: 2.4.41
* <strong>Task 7</strong>: Wappalyzer
* <strong>Task 8</strong>: -x
* <strong>Task 9</strong>: login.php