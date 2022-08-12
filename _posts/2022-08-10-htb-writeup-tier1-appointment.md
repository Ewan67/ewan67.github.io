---
title: HTB - Starting Point - Tier 1 - Appointment Writeup
date: 2022-08-10 09:00:00 +0800
categories: [HTB, Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Con estra entrada iniciamos una serie de posts en los que vamos a resolver 5 máquinas que conforman el **Tier 1** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point).

Las máquinas que veremos en esta serie son Appointment, [Sequel]({% post_url 2022-08-10-htb-writeup-tier1-sequel %}), [Crocodile]({% post_url 2022-08-10-htb-writeup-tier1-crocodile %}), [Responder]({% post_url 2022-08-10-htb-writeup-tier1-responder %}) y [Three]({% post_url 2022-08-10-htb-writeup-tier1-three %}).

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
└─$ mkdir Tier1

┌──(ewan67㉿kali)-[~/Cybersecurity/HTB]
└─$ cd Tier1

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ echo "Notas Tier1" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ cat notas
Notas Tier1
```

## Appointment

El primer paso será iniciar la máquina correspondiente (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220810/img06.png)

Copiamos la IP del equipo remoto, en mi caso *10.129.217.36*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A 10.129.217.36 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 19:57 CEST
Nmap scan report for 10.129.217.36
Host is up (0.083s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.14 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Tenemos el puerto *80/tcp* con un *Apache httpd 2.4.38 ((Debian))* escuchando. Abrimos el navegador y cargamos la IP.

![](/assets/posts/20220810/img07.png)

Formulario de *Login* al canto.

Intentamos enviarlo vacío. Se queja. Metemos cualquier valor para *Username* y *Password*. Se envía pero no nos dice nada, no nos devuelve mensaje alguno.

Siendo que la página es un simple formulario, pasamos a probar si la app que esta detrás es vulnerable a SQLi.

> **Nota:** Para los que estéis un poco verdes con este tema os recomiendo echar un ojo al tuto que nos ofrece **Portswigger** (los fabricantes de Burp Suite) [aquí](https://portswigger.net/web-security/sql-injection). Está en inglés pero vale la pena leerlo entero y darse de alta en el sitio (es gratis) para practicar con los laboratorios que nos proponen (que si bien estan bastante centrados en su aplicación, viendo el lado bueno, ésto nos permite aprender a manejar Burp Suite en el mismo viaje que profundizamos en SQLi).

Probamos con los clásicos en el campo *Username*, metiendo cualquier cadena en *Password* para sortear el script que valida el envío. Por ejemplo:

```
buenas' or 1=1;#
hola' or 1=1;-- q tal
```

Bingo !

## Respuestas:

* <strong>Task 1</strong>: Structured Query Language
* <strong>Task 2</strong>: SQL injection
* <strong>Task 3</strong>: Personally Identifiable Information
* <strong>Task 4</strong>: A03:2021-Injection
* <strong>Task 5</strong>: Apache httpd 2.4.38 ((Debian))
* <strong>Task 6</strong>: 443
* <strong>Task 7</strong>: brute-forcing
* <strong>Task 8</strong>: directory
* <strong>Task 9</strong>: 404
* <strong>Task 10</strong>: dir
* <strong>Task 11</strong>: #

