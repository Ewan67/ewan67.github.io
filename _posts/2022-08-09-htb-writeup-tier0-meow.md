---
title: Hack The Box - Starting Point - Tier 0 - Meow Writeup
date: 2022-08-09 09:00:00 +0800
categories: ["Hack The Box", Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Con esta entrada iniciamos una serie de posts en los que vamos a resolver las 4 máquinas que conforman el **Tier 0** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point), la puerta de entrada y requisito para poder avanzar dentro de esta plataforma.


Las resolveremos en el orden en que aparecen listadas en HTB.

![](/assets/posts/20220809/img01.png)

Dentro música.

## Inicio

> **Disclaimer:** algunos de estos primeros pasos que menciono a continuación son opcionales y solo reflejan mi modo de trabajar. Cada uno debería aplicar el propio.

Lo primero es conectarnos via VPN con HTB.

```console
┌──(ewan67㉿kali)-[~]
└─$ sudo openvpn /home/ewan67/VPN_certs/HTB_lab_ewan67.ovpn
```

Abrimos otro terminal en local, creamos la carpeta de trabajo, nos posicionamos en ella y creamos un fichero de notas en el que iremos apuntando información relevante a lo largo del proceso.

```console
┌──(ewan67㉿kali)-[~]
└─$ cd Cybersecurity/HTB

┌──(ewan67㉿kali)-[~/Cybersecurity/HTB]
└─$ mkdir Tier0

┌──(ewan67㉿kali)-[~/Cybersecurity/HTB]
└─$ cd Tier0

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ echo "Notas Tier0" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ cat notas
Notas Tier0
```

## Meow

El primer paso será iniciar la máquina correspondiente (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220809/img02.png)

Copiamos la IP del equipo remoto, en mi caso *10.129.70.144*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ nmap -A 10.129.70.144 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 10:27 CEST
Nmap scan report for 10.129.70.144
Host is up (0.056s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.71 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

**nmap** nos cuenta que el puerto *23/tcp* esta abierto y con un servicio *telnet* escuchando. Intentamos conectarnos, utilizando *root* como user.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ telnet 10.129.70.144
Trying 10.129.70.144...
Connected to 10.129.70.144.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: root
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 09 Aug 2022 08:29:49 AM UTC

  System load:           0.0
  Usage of /:            41.7% of 7.75GB
  Memory usage:          4%
  Swap usage:            0%
  Processes:             137
  Users logged in:       0
  IPv4 address for eth0: 10.129.70.144
  IPv6 address for eth0: dead:beef::250:56ff:fe96:156d

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

75 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
root@Meow:~# 
```

Bien. Hacemos un *ls -la*, vemos que hay un fichero *flag.txt*, y lo abrimos para hacernos con la primera bandera.

```console
root@Meow:~# ls -la
total 36
drwx------  5 root root 4096 Jun 18  2021 .
drwxr-xr-x 20 root root 4096 Jul  7  2021 ..
lrwxrwxrwx  1 root root    9 Jun  4  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root 3132 Oct  6  2020 .bashrc
drwx------  2 root root 4096 Apr 21  2021 .cache
-rw-r--r--  1 root root   33 Jun 17  2021 flag.txt
drwxr-xr-x  3 root root 4096 Apr 21  2021 .local
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-r--r--  1 root root   75 Mar 26  2021 .selected_editor
drwxr-xr-x  3 root root 4096 Apr 21  2021 snap
root@Meow:~# cat flag.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
root@Meow:~#
```

## Respuestas de este primer desafío:

* <strong>Task 1</strong>: Virtual Machine
* <strong>Task 2</strong>: terminal
* <strong>Task 3</strong>: openvpn
* <strong>Task 4</strong>: tun
* <strong>Task 5</strong>: ping
* <strong>Task 6</strong>: nmap
* <strong>Task 7</strong>: telnet
* <strong>Task 8</strong>: root