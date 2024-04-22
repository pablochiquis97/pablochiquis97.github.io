---
title: Vulnhub - My Expense
date: 2024-04-19 14:10:00 +0800
author: pablo
categories: [Vulnhub, Linux]
tags:
  [Web Enumeration, XSS, CSRF, Cookie Hijacking, SQL Injection, Cracking Hashes]
---

## Descripción

> MyExpense is a deliberately vulnerable web application that allows you to train in detecting and exploiting different web vulnerabilities. Unlike a more traditional "challenge" application (which allows you to train on a single specific vulnerability), MyExpense contains a set of vulnerabilities you need to exploit to achieve the whole scenario.

Your credentials were: `samuel:fzghn4lw`

>

## Resolution summary

1. En la enumeración web se encuentra un panel `admin/admin.php` el cual da vulnerable para XSS
2. Interceptamos la cookie de sesión de varios usuarios
3. En el panel de usuario administrador existe vulnerabilidad SQLi

---

## Configuración máquina VMware

Configurar la máquina para que pueda ser detectada en la red local. Presionar `e` y Agregar

![Untitled](/assets/img/Machine/Untitled.png)

Y presionar `ctrl + x`

Dirigirnos al directorio `/etc/network` y editar el archivo `interfaces`

![Untitled](/assets/img/Machine/Untitled1.png)

Ir al directorio `/opt` y editar los archivos con la IP de la máquina

![Untitled](/assets/img/Machine/Untitled2.png)

Cambiar en los 4 scripts

![Untitled](/assets/img/Machine/Untitled3.png)

---

## Reconocimiento

Se identifica la IP de la máquina a través de `arp-scan`. Comando: `sudo arp-scan -l`

![Untitled](/assets/img/Machine/Untitled4.png)

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

![Untitled](/assets/img/Machine/Untitled5.png)

El sistema operativo es una `Linux`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de nmap.
Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn 192.168.1.20 -oG allPorts`

![Untitled](/assets/img/Machine/Untitled6.png)

Puertos abiertos son: `80,36859,40275,53159,59793`

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p80,3306,33060 192.168.1.20 -oN targeted`

Obteniendo:

![Untitled](/assets/img/Machine/Untitled7.png)

### Port 80 - HTTP (Apache)

Visitando el sitio web se obtiene:

![Untitled](/assets/img/Machine/Untitled8.png)

Enumerando a través del script `http-enum` de `nmap` se obtiene la ruta:

![Untitled](/assets/img/Machine/Untitled9.png)

Esto también se pudo haber obtenido a través de `gobuster`:

![Untitled](/assets/img/Machine/Untitled10.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled11.png)

Hay una opción que nos permite registrarnos:

![Untitled](/assets/img/Machine/Untitled12.png)

Pero la opción `Sign up!` se encuentra bloqueada, la podemos liberar modificando su fuente html

![Untitled](/assets/img/Machine/Untitled13.png)

Borrando la opción `disabled=””`

Agregando el usuario

![Untitled](/assets/img/Machine/Untitled14.png)

Dirigiéndonos nuevamente a la ruta `admin/admin.php`, obteniendo:

![Untitled](/assets/img/Machine/Untitled15.png)

Nuestro usuario ha sido creado, pero se encuentra inactivo.

La data que ingresamos está reflejada en el formulario por lo que se podría probar una vulnerabilidad del tipo XSS

Se procede agregar otro usuario pero esta vez con una inyección XSS

![Untitled](/assets/img/Machine/Untitled16.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled17.png)

Se comprueba que es vulnerable para `XSS`

---

## Exploitation via XSS

Podemos habilitar un servidor a través de python y tratar de cargar un archivo `pwned.js`

Primero vamos a tratar de comprobar si hay peticiones que se realicen al servidor

![Untitled](/assets/img/Machine/Untitled18.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled19.png)

Se observa que hay interacción con el servidor creado

Vamos a crear un archivo `pwned.js` con un código para tratar de obtener la cookie de sesión del usuario actual que este interactuando con el servidor:

![Untitled](/assets/img/Machine/Untitled20.png)

Levantado el servidor web, se obtiene:

![Untitled](/assets/img/Machine/Untitled21.png)

Para verificar en qué usuario podemos iniciar sesión, ingresamos la cookie de sesión correspondiente.

Cambiando cookie:

![Untitled](/assets/img/Machine/Untitled22.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled23.png)

Se observa un mensaje donde nos dice que el usuario administrador solo puede estar autenticado en simultáneo una vez

Como verificamos que el usuario `administrador` es el que está interactuando con el servidor nos podemos aprovechar y hacer que este active la cuenta del usuario `slamotte`

Inspeccionando el código fuente podemos saber el número de `id` del usuario `slamotte` Obteniendo:

![Untitled](/assets/img/Machine/Untitled24.png)

A través de:

![Untitled](/assets/img/Machine/Untitled25.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled26.png)

Se habrá activado la cuenta

En la descripción de la máquina habíamos obtenido las credenciales:

Your credentials were: `slamotte/fzghn4lw`

Ingresando en la cuenta:

![Untitled](/assets/img/Machine/Untitled27.png)

Dirigiéndonos al apartado `Expense Reports` y mandando el report. Obteniendo:

![Untitled](/assets/img/Machine/Untitled28.png)

![Untitled](/assets/img/Machine/Untitled29.png)

En la página principal tenemos un chat, donde se ven varios usuarios, lo que hace pensar que si creamos un archivo `pwned.js` podremos robar sus cookies de sesión:

![Untitled](/assets/img/Machine/Untitled30.png)

Levantando servidor, obteniendo:

![Untitled](/assets/img/Machine/Untitled31.png)

Probando la cookies, podemos ingresar como otros usuarios:

![Untitled](/assets/img/Machine/Untitled32.png)

Obteniendo la sesión del usuario `Manon Riviere (mriviere)`

![Untitled](/assets/img/Machine/Untitled33.png)

Aceptando el ticket:

![Untitled](/assets/img/Machine/Untitled34.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled35.png)

El `manager` del usuario `Manon Riviere (mriviere)` es `Paul Baudouin`

![Untitled](/assets/img/Machine/Untitled36.png)

No disponemos la cookie del usuario `Paul Baudouin`

Dirigiéndonos al panel `Rennes` el cual es vulnerable para `SQLi`

![Untitled](/assets/img/Machine/Untitled37.png)

---

## SQLi

Mientras nos encontramos con la sesión iniciada como el usuario `mriviere` y nos dirigimos al panel de Rennes, procedemos a realizar pruebas de `SQLi`.

Para explotar una SQLi:

1. Encontrar el número de columnas a través de la query `order by 100-- -`

   ![Untitled](/assets/img/Machine/Untitled38.png)

   En este caso dispone de **dos** columnas

2. Listando todas las bases de datos `union select 1,schema_name from information_schema.schemata-- -`

![Untitled](/assets/img/Machine/Untitled39.png)

Listando base de datos actual en uso:

![Untitled](/assets/img/Machine/Untitled40.png)

1. Listando tablas `union select 1,table_name from information_schema.tables where table_schema='myexpense'-- -`

![Untitled](/assets/img/Machine/Untitled41.png)

1. Listar columnas: `union select 1,column_name from information_schema.columns where table_schema='myexpense' and table_name='user'-- -`

![Untitled](/assets/img/Machine/Untitled42.png)

1. Listando data: `union select 1,group_concat(username,0x3a,password) from user-- -`

![Untitled](/assets/img/Machine/Untitled43.png)

Obteniendo:

![Untitled](/assets/img/Machine/Untitled44.png)

Utilizando en vim `%s/,/\r/g` obteniendo:

![Untitled](/assets/img/Machine/Untitled45.png)

Utilizando [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash) para romper la contraseña del usuario `pbaudovin`

Obteniendo:

![Untitled](/assets/img/Machine/Untitled46.png)

Ingresando al usuario `Paul Badouin (pbaudouin)`

`pbaudouin:HackMe`

![Untitled](/assets/img/Machine/Untitled47.png)

Aceptando el pago:

![Untitled](/assets/img/Machine/Untitled48.png)

![Untitled](/assets/img/Machine/Untitled49.png)

Si nos dirigimos de nuevo al usuario: `slamotte/fzghn4lw`

Obteniendo la flag:

![Untitled](/assets/img/Machine/Untitled50.png)
