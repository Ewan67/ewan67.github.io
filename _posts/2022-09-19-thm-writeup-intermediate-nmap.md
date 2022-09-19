---
title: Try Hack Me - Intermediate Nmap - Writeup
date: 2022-09-19 09:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con el room [Intermediate Nmap](https://tryhackme.com/room/intermediatenmap) de [THM](https://tryhackme.com) cuyo autor es [cmnatic](https://tryhackme.com/p/cmnatic). Se trata de un CTF de nivel **Easy** realmente sencillo en el que utilizaremos herramientas como **nmap**, **netcat** y sobre todo pondremos a prueba nuestra capacidad de observación.

Dentro música.

## Inicio

Para empezar, nos leemos con atención la descripción que acompaña el room. Os la dejo traducida:

> Has aprendido algunas habilidades de <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">nmap</code>&nbsp;. ¿Ahora puedes combinar eso con otras habilidades con <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">netcat</code>&nbsp; y protocolos, para entrar en ésta máquina y encontrar la bandera? ¡Esta VM está escuchando en un puerto alto, y si te conectas a él puede darte alguna información que puedes usar para conectarte a un puerto más bajo comúnmente usado para acceso remoto!

Bueno, lo cierto es que parecen pocas pistas, pero en breve veremos que son todas las que necesitamos para resolver el room. El autor nos da la siguiente información:

* La máquina esta escuchando en un puerto alto.
* Si nos conectamos a él obtendremos información para conectarnos a un servicio comúnmente usado para acceso remoto corriendo en un puerto más bajo.
* Debemos aprovechar nuestros conocimientos de **nmap** y **netcat**.

## Enumeración

Pues vamos a ello. Empezamos lanzando un escaneo con **nmap**:

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/IntermediateNmap]
└─$ nmap -sC -sV -p- 10.10.227.18 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-19 15:53 CEST
Nmap scan report for 10.10.227.18
Host is up (0.035s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7d:dc:eb:90:e4:af:33:d9:9f:0b:21:9a:fc:d5:77:f2 (RSA)
|   256 83:a7:4a:61:ef:93:a3:57:1a:57:38:5c:48:2a:eb:16 (ECDSA)
|_  256 30:bf:ef:94:08:86:07:00:f7:fc:df:e8:ed:fe:07:af (ED25519)
2222/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 28:06:ca:9e:c1:d2:39:07:f6:b3:54:99:2c:e4:c9:0e (RSA)
|   256 25:6c:68:0c:2d:ec:fa:2e:78:12:03:a1:aa:5c:b9:16 (ECDSA)
|_  256 98:e9:a2:b3:bf:33:45:cb:77:3b:c6:9c:60:30:f2:b4 (ED25519)
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe: 
|     In case I forget - user:pass
|_    <REDACTADO>:<REDACTADO>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.92%I=7%D=9/19%Time=63287472%P=x86_64-pc-linux-gnu%r(N
SF:ULL,35,"In\x20case\x20I\x20forget\x20-\x20user:pass\n<REDACTADO>:<REDACTADO>
[...]
SF:<REDACTADO>\n\n")%r(TerminalServer,35,"In\x20case\x20I\x20forget\x2
SF:0-\x20user:pass\n<REDACTADO>:<REDACTADO>\n\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.92 seconds
```

Significado de las flags:

* `-sC`&nbsp;: devuelve información sobre la versión de los servicios corriendo en los puertos abiertos.
* `-sV`&nbsp;: realiza un escaneo utilizando el conjunto de scripts por defecto de **nmap**.
* `-p-`&nbsp;: escanea los 65535 puertos.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Atentos a la descripción, incluímos la flag `-p-`&nbsp; para que **nmap** escanée todos los puertos.

El volcado de **nmap** nos cuenta que tenemos tres puertos abiertos:

* *22/tcp* con un servicio **ssh** escuchando
* *2222/tcp* también con **ssh** escuchando
* *31337/tcp* con algo llamado *Elite* del otro lado (?)

Sobre éste último puerto, el propio **nmap** nos dice que se trata de *"un servicio no reconocido a pesar de devolver datos ..."*&nbsp; y que si sabemos de qué se trata les enviémos la huella correspondiente.

*31337/tcp* ? ... *Elite* ? ... la costumbre y la curiosidad me llevaron a emprender un par de búsquedas en Google para arrojar algo de luz sobre el asunto, buen hábito que os recomiendo (nunca es mal momento para adquirir nuevos conocimientos y hay información interesante asociada a este puerto como por ejemplo [aquí](https://www.speedguide.net/port.php?port=31337)) si bien en este caso, y perdonarme el spoiler ... la cosa no iba por ahí.

Leyendo con atención la salida asociada a este puerto vemos un dato interesante:

```
|     In case I forget - user:pass
|_    <REDACTADO>:<REDACTADO>
```

De verdad !? ... un juego de credenciales como salida de un servicio misterioso ??

Siguiendo el consejo del autor, conectamos con **netcat** contra la IP y el puerto para confirmar.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/IntermediateNmap]
└─$ nc 10.10.227.18 31337 
In case I forget - user:pass
<REDACTADO>:<REDACTADO>
```

Pues toca probar. Lo intentamos vía **ssh** empezando por el *22/tcp* (recordar que tenemos 2 puertos con **ssh** abiertos).

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/IntermediateNmap]
└─$ ssh <REDACTADO>@10.10.227.18
<REDACTADO>@10.10.227.18's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.13.0-1014-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ pwd
/home/<REDACTADO>
$ whoami
<REDACTADO>
$ ls -la
total 28
drwxr-xr-x 1 <REDACTADO> <REDACTADO> 4096 Sep 19 14:05 .
drwxr-xr-x 1 root   root   4096 Mar  2  2022 ..
-rw-r--r-- 1 <REDACTADO> <REDACTADO>  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 <REDACTADO> <REDACTADO> 3771 Feb 25  2020 .bashrc
drwx------ 2 <REDACTADO> <REDACTADO> 4096 Sep 19 14:05 .cache
-rw-r--r-- 1 <REDACTADO> <REDACTADO>  807 Feb 25  2020 .profile
$ cd ..
$ ls -la
total 20
drwxr-xr-x 1 root   root   4096 Mar  2  2022 .
drwxr-xr-x 1 root   root   4096 Mar  2  2022 ..
drwxr-xr-x 1 <REDACTADO> <REDACTADO> 4096 Sep 19 14:05 <REDACTADO>
drwxr-xr-x 2 root   root   4096 Mar  2  2022 user
$ cd user
$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Mar  2  2022 .
drwxr-xr-x 1 root root 4096 Mar  2  2022 ..
-rw-rw-r-- 1 root root   38 Mar  2  2022 flag.txt
$ cat flag.txt
flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
```

Máquina resuelta.

Espero que os haya servido de ayuda.

Saludos he intentar ser felices.