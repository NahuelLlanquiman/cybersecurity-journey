# Bloque A — Redes

## Pila TCP/IP y diferencias con OSI

### Las 4 capas de TCP/IP

Capa 4 — Aplicación: donde viven los protocolos que usa directamente el programa o usuario. HTTP, HTTPS, DNS, SSH, FTP, SMTP. Esta capa no sabe cómo se transportan los datos, solo define el formato y el significado del mensaje. Equivale a las capas 5, 6 y 7 de OSI colapsadas en una sola.

Capa 3 — Transporte: toma el mensaje de Aplicación y lo prepara para viajar. Decide si usa TCP o UDP. Agrega los puertos de origen y destino. TCP divide el mensaje en segmentos numerados y garantiza entrega y orden. UDP manda el datagrama sin garantías.

Capa 2 — Internet: encapsula los segmentos en paquetes IP. Agrega las direcciones IP de origen y destino. Responsable del enrutamiento — decidir por qué camino viaja el paquete. No garantiza entrega ni orden.

Capa 1 — Acceso a red: convierte los paquetes en tramas para el medio físico concreto (Ethernet, Wi-Fi). Agrega direcciones MAC. Es lo que realmente sale por el cable o el aire. Equivale a las capas 1 y 2 de OSI.

### Comparación OSI vs TCP/IP

| OSI (7 capas)     | TCP/IP (4 capas) | Qué vive ahí                        |
|-------------------|------------------|-------------------------------------|
| 7 - Aplicación    |                  |                                     |
| 6 - Presentación  | Aplicación       | HTTP, DNS, SSH, TLS (parcialmente)  |
| 5 - Sesión        |                  |                                     |
| 4 - Transporte    | Transporte       | TCP, UDP, puertos                   |
| 3 - Red           | Internet         | IP v4/v6, ICMP, ARP                 |
| 2 - Enlace datos  | Acceso a red     | Ethernet, Wi-Fi, MAC                |
| 1 - Física        |                  |                                     |

```
OSI                    TCP/IP
─────────────────────────────────
7 - Aplicación    ┐
6 - Presentación  ├──► Aplicación
5 - Sesión        ┘
4 - Transporte    ────► Transporte
3 - Red           ────► Internet
2 - Enlace datos  ┐
1 - Física        ┴──► Acceso a red
```

OSI es el modelo de referencia y vocabulario estándar de la industria. TCP/IP es lo que corre realmente en los sistemas. Hay que moverse entre los dos sin confundirse.

### Encapsulación

Cada capa agrega su propio header al dato que recibe de la capa superior. En el destino, cada capa lee su header y lo retira pasando el resto hacia arriba.

```
[Aplicación]   genera:       [HTTP Request]
[Transporte]   encapsula:    [TCP header | HTTP Request]
[Internet]     encapsula:    [IP header | TCP header | HTTP Request]
[Acceso red]   encapsula:    [Ethernet header | IP header | TCP header | HTTPRequest | Ethernet trailer]
```

### Unidades de datos por capa (PDU)

| Capa TCP/IP  | PDU                          |
|--------------|------------------------------|
| Aplicación   | Mensaje                      |
| Transporte   | Segmento (TCP) / Datagrama (UDP) |
| Internet     | Paquete                      |
| Acceso a red | Trama (Frame)                |

### Posición de TLS en la pila

TLS no encaja perfectamente en ninguna capa de TCP/IP. Opera entre Transporte y Aplicación: recibe los datos de HTTP, los cifra, y los entrega a TCP. Por eso Wireshark lo muestra como su propia capa (TLSv1.3) separada de HTTP y TCP. En OSI teóricamente viviría en capa 6 (Presentación), pero en la práctica TCP/IP no tiene esa separación.

### Puertos — rangos oficiales (IANA)

| Rango         | Nombre             | Descripción                              |
|---------------|--------------------|------------------------------------------|
| 0 – 1023      | Well-known ports   | Asignados por IANA a protocolos estándar |
| 1024 – 49151  | Registered ports   | Aplicaciones registradas                 |
| 49152 – 65535 | Dynamic/ephemeral  | Asignados por el SO al cliente           |

Puertos well-known relevantes: HTTP 80, HTTPS 443, SSH 22, DNS 53, FTP 21.

### TCP — Handshake de 3 vías

Mecanismo que TCP usa para establecer una conexión antes de enviar datos. Intervienen los flags SYN y ACK.


| Cliente              | Hacia | Servidor                 |
| -------------------- | ----- | ------------------------ |
| SYN (seq=x)          | ----> | SYN-ACK (seq=Y, ack=X+1) |
| CONEXIÓN ESTABLECIDA | <---- | SYN-ACK (seq=Y, ack=X+1) |


- SYN: el cliente envía su ISN (Initial Sequence Number) y pide abrir conexión.
- SYN-ACK: el servidor confirma el SYN del cliente (ack=X+1) y envía su propio ISN (seq=Y).
- ACK: el cliente confirma el SYN del servidor (ack=Y+1). Conexión establecida.

### Términos clave

TCP (Transmission Control Protocol): protocolo de transporte orientado a conexión. Garantiza entrega, orden y control de flujo.

UDP (User Datagram Protocol): protocolo de transporte sin conexión. Sin garantías, más rápido. Usado en DNS, video streaming, gaming.

IP (Internet Protocol): protocolo de la capa Internet. Direcciona y enruta paquetes entre redes.

ISN (Initial Sequence Number): número aleatorio que cada extremo genera al iniciar una conexión TCP. Base para numerar todos los bytes del flujo.

SYN (Synchronize): flag TCP que indica solicitud de sincronización / inicio de conexión.

ACK (Acknowledgment): flag TCP que confirma recepción de datos.

FIN (Finish): flag TCP para cierre ordenado de conexión.

RST (Reset): flag TCP para corte abrupto de conexión.

IANA (Internet Assigned Numbers Authority): organismo que administra la asignación de puertos, IPs y otros identificadores de internet.

PDU (Protocol Data Unit): nombre genérico para la unidad de datos en cada capa.

### Validación

- Captura Wireshark de handshake TCP real contra 140.82.112.21 (GitHub): SYN/SYN-ACK/ACK identificados con flags y números de secuencia verificados.
- ISN cliente (raw): 2887821683 — ISN servidor (raw): 582919634.
- ACK del SYN-ACK verificado como ISN cliente + 1: 2887821684.
- FIN-ACK y RST identificados en el cierre de la misma sesión.