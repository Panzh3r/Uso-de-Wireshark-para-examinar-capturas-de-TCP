# Uso de Wireshark para examinar capturas de TCP


<img width="757" height="208" alt="Uso de Wireshark para examinar capturas de TCP drawio" src="https://github.com/user-attachments/assets/65613aba-26f6-4230-a5ec-bfc89fe687ec" />   



## Definición
En este laboratorio se evidenciará una captura de TCP, en especifico de una sesión FTP. Esta topología consta de una VM Kali Linux (como cliente FTP) con acceso a Internet y un servidor FTP púbico.

## Objetivo
- Identificar campos de encabezado y operación TCP mediante una captura de sesión FTP de Wireshark.

## Aspectos básicos/situación
Esta práctica de laboratorio utilizará la herramienta de código abierto Wireshark para capturar y analizar campos de encabezado del protocolo TCP para las transferencias de archivos FTP entre el equipo host y un servidor FTP anónimo. Se utiliza la línea de comandos del terminal para establecer una conexión a un servidor FTP anónimo y descargar un archivo.

## Recursos necesarios
- VM Kali Linux
- Acceso a Internet

## Actividad: Identificar campos de encabezado y operación TCP mediante una captura de sesión FTP de Wireshark
Utilizaras Wireshark para capturar una sesión FTP e inspeccionar los campos de encabezado de TCP.

**Paso 1: Iniciar una captura de Wireshark.**
- Abre una ventana de la terminal de Kali Linux e inicia Wireshark.
- Inicia una captura de Wireshark correspondiente a la interfaz `eth0` (o la que corresponda a tu VM).
- Abre otra ventana de la terminal para acceder al sitio ftp externo. Escribe la dirección `ftp ftp.cs.brown.edu` en la línea de comandos. Conectate al sitio FTP con el usuario `anonymous` y sin contraseña.

`┌──(root㉿augusto)-[~]`  
`└─# ftp ftp.cs.brown.edu`  
*Connected to ftp.cs.brown.edu.*  
*220---------- Welcome to Pure-FTPd [privsep] [TLS] ----------*  
*220-You are user number 5 of 50 allowed.*  
*220-Local time is now 18:49. Server port: 21.*  
*220-IPv6 connections are also welcome on this server.*  
*220 You will be disconnected after 15 minutes of inactivity.*  
*Name (ftp.cs.brown.edu:kali): anonymous*  
*230 Anonymous user logged in*  
*Remote system type is UNIX.*  
*Using binary mode to transfer files.*  
*ftp>*

**Paso 2: Descargar el archivo pub.**
- Localiza y descarga el archivo *pub*; para ello, introduzca el comando `ls` para generar una lista de los archivos.

`ftp> ls`  
*229 Extended Passive mode OK (|||50485|)*  
*150 Accepted data connection*  
*drwxrwxrwt 2 0 0 19325154 May 6 18:48 incoming*  
*drwxr-xr-x 33 0 2 2323 Aug 13 2013 pub*  
*drwxr-xr-x 52 0 2 1087 Feb 11 2016 u*  
*226-Options: -l*  
*226 3 matches total*  

- Introduce el comando `get pub` para descargar el archivo. Cuando finalice la descarga, introduzca el comando `quit` para salir.

`ftp> get pub`  
*200 PORT command successful.*  
*125 Data connection already open; Transfer starting.*  
*WARNING! 36 bare linefeeds received in ASCII mode*  
*File may not have transferred correctly.*  
*226 Transfer complete.*  
*2323 bytes received in 0.056 seconds (24.9 kbytes/s)*  

- Una vez finalizada la transferencia, introduce el comando `quit` para salir de ftp.

`ftp> quit`

**Paso 3: Detener la captura Wireshark.**

**Paso 4: Ver la ventana principal de Wireshark.**
- Wireshark capturó muchos paquetes durante la sesión FTP para `ftp.cs.brown.edu`. Si quieres limitar la cantidad de datos para el análisis, aplica el filtro `tcp and ip.addr == 128.148.32.111` y haga clic en Aplicar.
<img width="1137" height="487" alt="image" src="https://github.com/user-attachments/assets/85d12c91-ceac-4ad2-92f1-3c41e76999a5" />

**Paso 5: Analizar los campos TCP**
Una vez aplicado el filtro de TCP, en los primeros tres paquetes (sección superior) se muestra la secuencia de [SYN], [SYN, ACK] y [ACK], que es el protocolo de enlace TCP de tres vías.
<img width="1100" height="74" alt="image" src="https://github.com/user-attachments/assets/46cb9d54-e297-4fbf-a55e-7bc53c3c6999" />

TCP se utiliza en forma continua durante una sesión para controlar la entrega de datagramas, verificar la llegada de datagramas y administrar el tamaño de la ventana. Para cada intercambio de datos entre el cliente FTP y el servidor FTP, se inicia una nueva sesión TCP. Al término de la transferencia de datos, se cierra la sesión TCP. Cuando finaliza la sesión FTP, TCP realiza un cierre y un apagado ordenado.
En Wireshark, se encuentra disponible información detallada sobre TCP en el panel de detalles del paquete (sección media). Resalte el primer datagrama TCP del host y expanda porciones del datagrama TCP, como se muestra a continuación.

<img width="1053" height="812" alt="image" src="https://github.com/user-attachments/assets/cd267327-ee2e-4769-a35d-e62f1205b32c" />


El datagrama TCP expandido se muestra de manera similar al panel de detalles de paquetes que se muestra a continuación:

<img width="1137" height="526" alt="image" src="https://github.com/user-attachments/assets/8cafd6f6-82dd-4535-a71c-4c2ec4c3769c" />


La imagen anterior es un diagrama del datagrama TCP. Se proporciona una explicación de cada campo para referencia:
- El número de puerto de origen TCP pertenece al host de la sesión TCP que abrió una conexión. Generalmente el valor es un valor aleatorio superior a 1023.
- El número de puerto de destino TCP se utiliza para identificar el protocolo de capa superior o la aplicación en el sitio remoto. Los valores en el intervalo de 0 a 1023 representan los “puertos bien conocidos” y están asociados a servicios y aplicaciones populares (como se describe en la RFC 1700), por ejemplo, Telnet, FTP y HTTP. La combinación de la dirección IP de origen, el puerto de origen, la dirección IP de destino y el puerto de destino identifica de manera exclusiva la sesión para el remitente y para el destinatario.  
**Nota:** En la captura anterior de Wireshark, el puerto de destino es 21, que es el FTP. Los servidores FTP escuchan las conexiones de cliente FTP en el puerto 21.
- Sequence number (Número de secuencia) especifica el número del último octeto en un segmento.
- Acknowledgment number (Número de reconocimiento) especifica el siguiente octeto que espera el destinatario.
- Code bits (bits de código) tiene un significado especial en la administración de sesiones y en el tratamiento de los segmentos. Entre los valores interesantes se encuentran:
    - ACK: reconocimiento de la recepción de un segmento.
    - SYN: sincronizar, solo se define cuando se negocia una sesión de TCP nueva durante el protocolo de enlace de tres vías de TCP.
    - FIN: finalizar, la solicitud para cerrar la sesión de TCP.
- Window size (Tamaño de la ventana) es el valor de la ventana deslizante. Determina cuántos octetos pueden enviarse antes de esperar un reconocimiento.
- Urgent pointer (Puntero urgente) solo se utiliza con un marcador urgente (URG) cuando el remitente necesita enviar datos urgentes al destinatario.
- En Options (Opciones), hay una sola opción actualmente, y se define como el tamaño máximo del segmento TCP (valor opcional).

Utiliza la captura Wireshark del inicio de la primera sesión TCP (bit SYN fijado en 1) para completar la información acerca del encabezado TCP. Es posible que algunos campos no se apliquen a este paquete.
De la VM (Kali) al servidor FTP (solamente el bit SYN está definido en 1):
<img width="785" height="217" alt="image" src="https://github.com/user-attachments/assets/4e84f376-8157-4a85-abe4-c61216da22ee" />  

En la segunda captura filtrada de Wireshark, el servidor FTP confirma que recibió la solicitud de la VM. Observe los valores de los bits de SYN y ACK.
<img width="1091" height="20" alt="image" src="https://github.com/user-attachments/assets/827e0b54-3294-4ba0-9013-e759b5b729a2" />  

Completa la siguiente información sobre el mensaje de SYN-ACK:
<img width="787" height="221" alt="image" src="https://github.com/user-attachments/assets/5890acbe-e149-4987-bc5a-20a58d92e531" />  

En la etapa final de la negociación para establecer las comunicaciones, la VM envía un mensaje de acuse de recibo al servidor. Observen que solo el bit ACK está definido en 1, y que el número de secuencia se incrementó a 1.
<img width="1089" height="393" alt="image" src="https://github.com/user-attachments/assets/ad8e68d4-1ffd-4e2a-93af-99335d862c7c" />  

Complete la siguiente información sobre el mensaje de ACK.
<img width="787" height="221" alt="image" src="https://github.com/user-attachments/assets/981713df-cbe4-4898-8c96-9cd2a77de16a" />  

- ¿Cuántos otros datagramas TCP contenían un bit SYN?.

Una vez establecida una sesión TCP, puede haber tráfico FTP entre el cliente y el servidor FTP. El cliente y el servidor FTP se comunican entre ellos, sin saber que TCP controla y administra la sesión. Cuando el servidor FTP envía el mensaje Response: 220 (Respuesta:220) al cliente FTP, la sesión TCP en el cliente FTP envía un reconocimiento a la sesión TCP en el servidor. Esta secuencia es visible en la siguiente captura de Wireshark.
<img width="1137" height="380" alt="image" src="https://github.com/user-attachments/assets/0708d608-b56c-4a9b-89f8-fe900cdae8fa" />  

Cuando termina la sesión FTP, el cliente FTP envía un comando para “salir”. El servidor FTP reconoce la terminación de FTP con un mensaje Response: 221 Goodbye (Adiós). En este momento, la sesión TCP del servidor FTP envía un datagrama TCP al cliente FTP que anuncia la terminación de la sesión TCP. La sesión TCP del cliente FTP reconoce la recepción del datagrama de terminación y luego envía su propia terminación de sesión TCP. Cuando quien originó la terminación TCP (servidor FTP) recibe una terminación duplicada, se envía un datagrama ACK para reconocer la terminación y se cierra la sesión TCP. Esta secuencia es visible en la captura y el diagrama siguiente:
<img width="770" height="894" alt="image" src="https://github.com/user-attachments/assets/8b52066f-af02-48f6-840a-aa35ff71ea08" />  

Si se aplica un filtro `ftp`, puede examinarse la secuencia completa del tráfico FTP en Wireshark. Observe la secuencia de los eventos durante esta sesión FTP. Para recuperar el archivo `Pub`, se utilizó el nombre de usuario `anonymous` (anónimo). Una vez que se completó la transferencia de archivos, el usuario finalizó la sesión FTP:

<img width="1134" height="477" alt="image" src="https://github.com/user-attachments/assets/312da827-75ad-434b-b0f6-0f15935507d3" />












