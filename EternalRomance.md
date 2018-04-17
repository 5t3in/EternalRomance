## ETERNALROMANCE: DESPLIEGUE EXPLOIT EN WINDOWS SERVER 2003 SP1 X86

El exploit de SMB EternalRomance de la lista de vulnerabilidades filtradas utilizadas por la NSA/FuzzBunch que tiene como objetivo Windows XP/ Vista/ 7, también Windows Server 2003/ 2008, su principal característica es que ataca mediante el SMB (Puerto 445), teniendo como resultado el control de la maquina objetivo. En el paper de Sheila Berta se demostró como un ataque no autentificado puede explotar un objetivo Windows 7/2008 vulnerable a EternalBlue, DoublePulsar y Empire. En esta guía se mostrará como explotar un Windows Server 2003 SP1 x86 usando el exploit EternalRomance de FuzzBunch. Se debe considerar que el proceso de explotación es bastante similar al de EternalBlue, excepto que se usará DoublePulsar para generar un shellcode que será utilizado por EternalRomance. Se realizará siguiendo la guía de Mario Díaz Lira para el despliegue de FuzzBunch en Kali.

## MOTIVACIÓN

Luego de haber utilizado FuzzBunch "Metasploit de la NSA", para lanzar el exploit EternalBlue en cojunto con DoublePulsar y Empire (este ultimo para la creación del DLL maliciosa) en un Sistema Operativo Kali, me propuse la meta de lograr utilizar EternalRomance uno de los exploit para Microsoft Windows ya que no vi muchas guias para EternalBlue pero no para este, lo hice de la forma mas simple y rapida para que todos pudieran lograr llegar al resultado final.

## CONFIGURACIÓN DEL LABORATORIO

Antes de comenzar veremos la configuración del laboratorio que vamos a utilizar, posteriormente se usara el modulo auxiliar de Metasploit para comprobar si el objetivo se encuentra parchado. Finalizando con la instalación del backdoor de DoublePulsar con la creación del Payload en Msfvenom para ser utilizado por EternalRomance, para proporcionarnos una Shell para ser utilizada en Meterpreter

Para el laboratorio se usarán las siguientes maquinas:

- Microsoft Windows 2003 SP1 32-bits como host vulnerable. 192.168.0.111
- Kali GNU/Linux Rolling como host atacante. 192.168.0.108

Para esta guía se utilizarán las siguientes herramientas proporcionadas por Kali:

- FuzzBunch (NSA-Metasploit).
- Msfvenom (Framework que combina Msfpayload y Msfencode para crear el Payload).
- Metasploit (Suite diseñada para explotar vulnerabilidades).

## TRABAJO ORIGINAL Y AGRADECIMIENTOS:
- **ShadowBroker:** https://github.com/misterch0c/shadowbroker.git
- **Sheila Berta Paper:** https://www.exploit-db.com/docs/41897.pdf
- **Miguel Diaz Lira:** https://github.com/mdiazcl/fuzzbunch-debian.git
- **Countercepr:** https://github.com/countercept/doublepulsar-detection-script.git

## ESCANEANDO EL OBJETIVO
### DETECCIÓN DE CONECTIVIDAD DEL OBJETIVO CON NMAP
Utilizaremos nmap para verificar la conectividad del objetivo y los puertos que tiene abiertos.
```
nmap –Ss –Pn –O 192.168.0.111
```
![Imgur](http://i.imgur.com/EAAxr1t.jpg)

### DETECCIÓN A TRAVÉS DE METASPLOIT DE LA VULNERABILIDAD MS17-010 SMB

Para determinar si un objetivo tiene un parche MS17-010, podemos usar un módulo auxiliar de Metasploit MS17-010 SMB.
Comenzamos desde Msfconsole y ejecutamos el siguiente comando para comprobar si nuestro objetivo ha sido parcheado MS17_010:

![Imgur](http://i.imgur.com/iIRNWaJ.jpg)

## PREPARANDO EL BACKDOOR
### CREACIÓN DE UN SHELLCODE A TRAVÉS DOUBLEPULSAR

Antes de ejecutar el exploit EternalRomance necesitamos generar un shellcode con DoublePulsar. El archivo de salida que contiene este será utilizado por el exploit de EternalRomance para infectar el objetivo. Cuando la backdoor este instalada en el sistema de destino, podemos usarla para ejecutar una sesión con Meterpreter. Comencemos con FuzzBunch ingresando la información solicitada sobre la IP de destino y la IP del host atacante. Elija no utilizar la redirección y mantener el directorio de Base Log predeterminado:

![Imgur](http://i.imgur.com/CU1Uuko.jpg)

```
use doublepulsar
```

![Imgur](http://i.imgur.com/zC8xn6x.jpg)

Ahora especificaremos las variables como la arquitectura, el protocolo y el archivo de salida. En nuestro caso, como es un laboratorio podemos dejar la mayoría de las opciones predeterminadas porque la arquitectura de destino es x86 de 32 bits, el protocolo de destino es SMB y necesitamos dar salida a shellcode como archivo binario. El único parámetro que debemos modificar es el de la ruta de salida.

![Imgur](http://i.imgur.com/D0DtbDt.jpg)

Ejecutamos DoublePulsar.

![Imgur](http://i.imgur.com/rhtWlFv.jpg)

Teniendo la ruta lista de salida del shellcode, lo creamos y lo podemos visualizar a través de otra terminal con wine explorer.exe en el disco C:\

![Imgur](http://i.imgur.com/49osvAN.jpg)

## EXPLOTANDO VULNERABILIDAD
### CONFIGURANDO Y EJECUTANDO ETERNALROMANCE

Ahora utilizaremos EternalRomance, para ello mantendremos los datos por defecto, y se utilizara el plugin Smbtouch, este escaneara el objetivo a traves del protocolo SMB y el puerto 445 para comprobar si el objetivo es vulnerables.

![Imgur](http://i.imgur.com/ggzRKxY.jpg)

Ejecutamos Smbtouch.

![Imgur](http://i.imgur.com/iKKSOXp.jpg)

Si el plugin Smbtouch se ejecuta con exito debiera de mostrarnos los datos del objetivo como tipo de Sistema Operativo, que version de Service Pack tiene, el tipo de arquitectura, a que tipo de vulnerabilidad no es soportada y a cual si.

![Imgur](http://i.imgur.com/6GGisVl.jpg)

El siguiente paso es establecer los ajustes de EternalRomance:

![Imgur](http://i.imgur.com/lOyRMK0.jpg)

Ahora se le pedirá de nuevo la configuración de la variable EternalRomance. Ingresamos los datos de nuestro objetivo e ingresamos la ruta donde tenemos el shellcode creado con DoublePulsar.

![Imgur](http://i.imgur.com/ChJQJj9.jpg)

Elegimos todas las opciones predeterminadas hasta que se le solicite el Sistema Operativo de destino. El sistema operativo de destino correcto aquí (en nuestro caso elegimos la opción 5 - Windows Server 2003 SP1):

![Imgur](http://i.imgur.com/0wZTLUF.jpg)

Ahora preparamos el exploit de EternalRomance para la ejecución, elegir la IP y el puerto de destino y ejecutar:

![Imgur](http://i.imgur.com/Aa346sl.jpg)

Si todo esta correcto nos aparecerá lo siguiente:

![Imgur](http://i.imgur.com/8AogBCm.jpg)
**SUCCESS!**

Como se indica en la última línea, el exploit de EternalRomance se ha ejecutado correctamente en el equipo vulnerable Windows Server 2003 SP1 x86. Ahora debemos inyectar un DLL que crearemos con Msfvenom con esta herramienta simplificaremos la creacion del Payload. Usaremos la backdoor de DoublePulsar para este propósito. Esto es algo que ya se ha usado para los que lograron explotar con Eternalblue usando el framework Empire.

## REVERSE SHELL PAYLOAD CON MSFVENOM
Para crear el DLL que utilizaremos colocaremos los parametros que necesitamos para su creación entre ellos los mas importantes que se deben considerar son el tipo de extensión en este caso DLL, la arquitectura y la plataforma, eligiendo la ruta como se ve en la imagen, para que se pueda seleccionar desde DoublePulsar posteriormente.

![Imgur](http://i.imgur.com/2GCE00k.jpg)

## CONFIGURACIÓN DEL LISTENER EN MSFCONSOLE

Ahora abriremos Metasploit y utilizaremos el handler, con el proposito de esperar conexión del equipo vulnerable.

![Imgur](http://i.imgur.com/J5CWVHL.jpg)

## INYECTAR EL DLL SHELL INVERSO CON DOUBLEPULSAR

Ahora que tenemos a nuestro handler esperando conexión y corriendo en el puerto 4444, generamos el Payload para enviar a nuestro objetivo infectado. Para inyectar el Payload, primero debemos seleccionar DoublePulsar:

```
use doublepulsar
```

Se le pedirá de nuevo la configuración para DoublePulsar. En lugar de la opción predeterminada que genera un archivo binario shellcode, elegimos la opción 2 para inyectar un archivo DLL. A continuación, especificamos la ruta de acceso completa al archivo DLL para inyectar. Suponiendo que cuando la creamos con Msfvenom con la ruta hasta nuestro disco de C:\ de wine, en caso contrario debemos moverla hasta esa ruta.

![Imgur](http://i.imgur.com/v6VkWBI.jpg)

Finalmente ejecutamos DoublePulsar, si todo salió bien obtendríamos lo siguiente:

![Imgur](http://i.imgur.com/MXUbBmY.jpg)

# POST EXPLOTACIÓN

Luego de inyectamos el DLL en el objetivo, obtendriamos la sesión con Meterpreter:

![Imgur](http://i.imgur.com/soh0RAf.jpg)

![Imgur](http://i.imgur.com/6FX2rpt.jpg)

# CONCLUSIÓN

Finalmente, se ha obtenido una sesión en Meterpreter sobre un Sistema Operativo Windows Server 2003, tan solo conociendo su IP. Como se sabe el soporte para Windows Server 2003 y XP finalizo el año 2015, es por esto que estos no recibiran actualizaciones que corrijan esta y otras vulnerabilidades de SMB.

Que hacer si se tiene uno de estos Sistemas Operativos, hay tres soluciones para mitigar el problema:

- **Desactivar SMB:** https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/
- **Detectar Doublepulsar:** https://github.com/countercept/doublepulsar-detection-script.git.

Microsoft solo ha lanzado parches para los otros Sistemas Operativos de Microsoft, en el siguiente enlace se podrá saber mayor información acercade estos:

https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

### Stay safe!