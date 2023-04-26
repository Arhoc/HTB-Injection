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
