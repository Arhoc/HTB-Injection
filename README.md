# HTB-Injection

Informe (WriteUp) detallado sobre la resolución de la máquina de HackTheBox, Injection.

Lo primero es comprobar la conectividad con la máquina, enviando un paquete de sondeo ICMP, mediante el comando ping

```bash
[root@ArchLinux ~ ]$ ping 10.10.11.204 -c 1
PING 10.10.11.204 (10.10.11.204) 56(84) bytes of data.
64 bytes from 10.10.11.204: icmp_seq=1 ttl=46 time=57.8 ms

--- 10.10.11.204 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 57.847/57.847/57.847/0.000 ms
```
Como podemos apreciar, el servidor responde a nuestro paquete de sondeo ICMP, el paquete incluye una solicitud de eco (echo request) al host de destino, y espera una respuesta de eco (echo reply) del host de destino.

Si el host de destino está activo y es alcanzable, responderá al paquete de solicitud de eco con un paquete de respuesta de eco, como dato extra, el TTL en sistemas POSIX suele ser de 64, mientras que en sistemas NT (Windows), suele ser de 128, así que es posible intuír el sistema operativo del servidor mediante el TTL, el TTL (Time to Live) es un campo en el encabezado IP que indica el número máximo de saltos que puede hacer un paquete antes de ser descartado.

En sistemas POSIX, el valor predeterminado del TTL es de 64, mientras que en sistemas NT (Windows), el valor predeterminado es de 128, por lo tanto, en base a proximidad, intuímos que el sistema operativo de la máquina es POSIX (sistemas operativos basados en Unix).

Cabe señalar que este método no es completamente fiable, ya que el valor predeterminado del TTL se puede configurar en diferentes sistemas operativos, y algunos sistemas operativos incluso cambian el valor del TTL a medida que los paquetes atraviesan la red, por lo que se necesitaría confirmación adicional para determinar el sistema operativo del host de destino de manera precisa.


## Reconocimiento

Una vez comprobada la conectividad con la máquina, lo primero será realizar reconocimiento básico mediante NMAP
```
nmap -sS --min-rate 5000 -T5 -n -Pn -sCV 10.10.11.204 -vvv
```
### Parametros explicados
- -Ss: especifica el tipo de escaneo que se realizará. En este caso, se utiliza un escaneo de tipo SYN (también conocido como "half-open") que envía un paquete SYN al puerto objetivo y espera una respuesta SYN-ACK. Este tipo de escaneo es útil para identificar puertos abiertos sin establecer una conexión completa, lo que aumenta la velocidad del escaneo.

- --min-rate 5000: establece la tasa mínima de paquetes por segundo que se enviarán durante el escaneo. En este caso, se establece en 5000 paquetes por segundo.

- -T5: establece el tiempo de espera máximo para las respuestas de los hosts y los puertos. En este caso, se establece en el nivel más alto de tiempo de espera (T5), lo que significa que se espera una respuesta en un plazo de 900 segundos (15 minutos) como máximo.

- -n: deshabilita la resolución de nombres DNS inversa durante el escaneo. Esto puede acelerar el escaneo al evitar el retraso en la resolución de nombres.

- -Pn: trata todos los hosts como si estuvieran activos y evita el envío de pings de sondeo para determinar la conectividad del host. Esto se utiliza para escanear hosts que no responden a los sondeos de ping.

- -sCV: especifica el tipo de escaneo de versiones que se realizará en los puertos abiertos. En este caso, se realiza un escaneo que intenta determinar la versión y el nombre del servicio en los puertos abiertos.

- 10.10.11.204: especifica la dirección IP del objetivo que se va a escanear, en este caso la máquina.

- -vvv: aumenta el nivel de verbosidad de la salida del escaneo. En este caso, se establece en el nivel más alto de verbosidad, lo que significa que se mostrarán detalles muy detallados del escaneo.

Todas estas opciones están diseñadas para ayudar a descubrir hosts, servicios y detalles básicos de los servicios en una red, lo que corresponde a la fase de reconocimiento en el proceso de análisis de vulnerabilidades y pruebas de penetración.

```ruby
# Nmap 7.93 scan initiated Tue Apr 25 17:16:34 2023 as: nmap -sS --min-rate 5000 -T5 -n -Pn -sCV -oN Scanning.nmap -vvv 10.10.11.204
Increasing send delay for 10.10.11.204 from 0 to 5 due to 179 out of 446 dropped probes since last increase.
Warning: 10.10.11.204 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.11.204
Host is up, received user-set (0.20s latency).
Scanned at 2023-04-25 17:16:35 CST for 23s
Not shown: 651 closed tcp ports (reset), 347 filtered tcp ports (no-response)
PORT     STATE SERVICE     REASON         VERSION
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 caf10c515a596277f0a80c5c7c8ddaf8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDKZNtFBY2xMX8oDH/EtIMngGHpVX5fyuJLp9ig7NIC9XooaPtK60FoxOLcRr4iccW/9L2GWpp6kT777UzcKtYoijOCtctNClc6tG1hvohEAyXeNunG7GN+Lftc8eb4C6DooZY7oSeO++PgK5oRi3/tg+FSFSi6UZCsjci1NRj/0ywqzl/ytMzq5YoGfzRzIN3HYdFF8RHoW8qs8vcPsEMsbdsy1aGRbslKA2l1qmejyU9cukyGkFjYZsyVj1hEPn9V/uVafdgzNOvopQlg/yozTzN+LZ2rJO7/CCK3cjchnnPZZfeck85k5sw1G5uVGq38qcusfIfCnZlsn2FZzP2BXo5VEoO2IIRudCgJWTzb8urJ6JAWc1h0r6cUlxGdOvSSQQO6Yz1MhN9omUD9r4A5ag4cbI09c1KOnjzIM8hAWlwUDOKlaohgPtSbnZoGuyyHV/oyZu+/1w4HJWJy6urA43u1PFTonOyMkzJZihWNnkHhqrjeVsHTywFPUmTODb8=
|   256 d51c81c97b076b1cc1b429254b52219f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIUJSpBOORoHb6HHQkePUztvh85c2F5k5zMDp+hjFhD8VRC2uKJni1FLYkxVPc/yY3Km7Sg1GzTyoGUxvy+EIsg=
|   256 db1d8ceb9472b0d3ed44b96c93a7f91d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICZzUvDL0INOklR7AH+iFw+uX+nkJtcw7V+1AsMO9P7p
8080/tcp open  nagios-nsca syn-ack ttl 63 Nagios NSCA
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Apr 25 17:16:58 2023 -- 1 IP address (1 host up) scanned in 23.98 seconds
```

Este escaneo muestra que el puerto 22 está abierto y ejecutando el servicio SSH (Secure Shell). También se puede ver información detallada sobre la versión de SSH que se está ejecutando en este puerto, que es "OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)".

Además, el escaneo también indica que el puerto 8080 está abierto y ejecutando el servicio Nagios NSCA. Nagios es un popular software de monitoreo de red, y NSCA (Nagios Service Check Acceptor) es un componente que permite enviar resultados de monitoreo a un servidor Nagios centralizado.

También se puede observar que el servidor está ejecutando una distribución de Linux Ubuntu con el kernel de Linux, y se proporciona información sobre las claves de host SSH que se utilizan para la autenticación del servidor vía SSH

Entonces, sabido esto, intuímos que, así como hay un servicio SSH coriendo sobre el puerto 22 en la máquina, también podemos intuír que se encuentra un servidor web corriendo sobre el puerto 8080, así que intentamos acceder por medio del navegador.

<img src="https://github.com/Arhoc/HTB-Injection/blob/main/Screenshot%20from%202023-04-26%2014-55-19.png?raw=true">

Nos econtramos con una pagina web, que aparentemente sirve para almacenar archivos y carpetas, así como realizar colaboraciones con estos desde cualquier dispositivo.

Al intentar acceder a la ruta "Upload", se nos redirige a un formulario para poder subir archivos.

<img src="https://github.com/Arhoc/HTB-Injection/blob/main/Screenshot%20from%202023-04-26%2014-58-57.png?raw=true">

sin embargo, la web realiza verificación del tipo y tamaño de archivo, lo cual imposibilita explotar una vulnerabilidad "file upload vulnerability"
