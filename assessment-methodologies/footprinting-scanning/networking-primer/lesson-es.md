# Networking Primer (Bases de Redes)

## 1. Modelo OSI (7 Capas)

El modelo OSI (Open Systems Interconnection) es un marco conceptual que describe cómo viajan los datos por una red. Tiene **7 capas** [web:21][web:24]:

| Capa | Nombre         | UDI       | Función principal                                      | Ejemplos de protocolos/dispositivos     |
|------|----------------|-----------|--------------------------------------------------------|-----------------------------------------|
| 7    | Aplicación     | Datos     | Servicios面向 usuario, aplicaciones de red             | HTTP, FTP, DNS, SMTP                    |
| 6    | Presentación   | Datos     | Formateo, cifrado, compresión de datos                 | SSL/TLS, JPEG, ASCII                    |
| 5    | Sesión         | Datos     | Gestiona sesiones entre aplicaciones                   | NetBIOS, RPC                            |
| 4    | Transporte     | Segmento  | Comunicación extremo a extremo, fiabilidad, control    | TCP, UDP                                |
| 3    | Red            | Paquete   | Enrutamiento, direccionamiento lógico (IP)             | IP, ICMP, routers                       |
| 2    | Enlace de Datos| Trama     | Direccionamiento MAC, detección de errores, switches   | Ethernet, switches, MAC                 |
| 1    | Física         | Bits      | Transmisión física (cables, señales)                   | Cables, hubs, tarjetas de red, radio    |

Los datos bajan por las capas en el remitente (7 → 1) y suben en el receptor (1 → 7).

---

## 2. IP (Protocolo de Internet)

IP es un protocolo de la **capa de red (capa 3)** responsable de enrutar paquetes desde el origen hasta el destino usando direcciones IP.

### Estructura del paquete (general)

Un paquete IP consta de:

- **Cabecera IP** – información de control (IP origen/destino, TTL, protocolo, etc.)
- **Payload** – los datos transportados por el paquete IP (por ejemplo, segmento TCP, datagrama UDP)

```text
+---------------------+----------------------+
|      Cabecera IP    |      Payload         |
| (20–60 bytes)       | (TCP/UDP + datos)    |
+---------------------+----------------------+
```

El payload normalmente contiene un **segmento de la capa de transporte** (TCP o UDP) más datos de la aplicación.

---

## 3. TCP vs UDP

### TCP (Protocolo de Control de Transmisión)

- **Capa**: Transporte (capa 4)
- **Orientado a conexión**: establece una conexión antes de enviar datos
- **Fiabilidad**: garantiza entrega, orden y retransmisión
- **Control de flujo y congestión**
- Usado por: HTTP, HTTPS, SSH, FTP, SMTP

### UDP (Protocolo de Datagramas de Usuario)

- **Capa**: Transporte (capa 4)
- **No orientado a conexión**: no hace handshake, solo envía paquetes
- **No fiable**: no garantiza entrega ni orden
- **Más rápido**, menos overhead
- Usado por: DNS, DHCP, VoIP, streaming, SNMP

---

## 4. Cabecera IPv4 en detalle

Un datagrama IPv4 tiene una **cabecera de longitud variable** (mínimo 20 bytes, hasta 60 bytes con opciones) [web:22][web:25][web:28].

### Campos de la cabecera IPv4

| Campo                  | Tamaño (bits) | Descripción                                                                 |
|------------------------|---------------|-----------------------------------------------------------------------------|
| Versión                | 4             | Versión de IP; para IPv4 es `4`                                           |
| IHL (Longitud Cabecera)| 4             | Longitud de cabecera en palabras de 32 bits; min 5 (20 bytes), max 15 (60) |
| Tipo de Servicio       | 8             | QoS: prioridad, retraso, throughput, fiabilidad                            |
| Longitud Total         | 16            | Longitud total de cabecera + datos (mín 20, máx 65.535 bytes)              |
| Identificación         | 16            | ID único para fragmentación/reensamblaje                                   |
| Flags                  | 3             | Reservado (0), No Fragmentar (DF), Más Fragmentos (MF)                     |
| Desplazamiento Fragmento| 13           | Desplazamiento de este fragmento en unidades de 8 bytes                    |
| TTL (Time to Live)     | 8             | Máximo número de saltos antes de que el paquete se descarte                |
| Protocolo              | 8             | Siguiente protocolo: `1=ICMP`, `6=TCP`, `17=UDP`                           |
| Checksum Cabecera      | 16            | Checksum solo de la cabecera (no del payload)                              |
| IP Origen              | 32            | Dirección IP del remitente                                                 |
| IP Destino             | 32            | Dirección IP del receptor                                                  |
| Opciones (opcional)    | variable      | Características extra (ej. enrutamiento fuente) + relleno                  |

### Estructura del paquete IPv4

```text
+------------------+-------------------+
|   Cabecera IPv4  |    Payload        |
| (20–60 bytes)    | (TCP/UDP + datos) |
+------------------+-------------------+
```

El campo **Protocolo** indica al receptor qué hay dentro del payload:
- `1` → ICMP
- `6` → TCP
- `17` → UDP

---

## 5. TCP 3-Way Handshake (Triálogo de 3 Vías)

TCP es **orientado a conexión**. Antes de transferir datos, se establece una conexión fiable mediante un **handshake de 3 vías** [web:23][web:26][web:29]:

### Pasos

1. **SYN**  
   - El cliente envía un segmento con `SYN=1`, `ACK=0`.
   - Elige un **Número de Secuencia Inicial (ISN)**, ej. `Seq = 1001`.

2. **SYN-ACK**  
   - El servidor responde con `SYN=1`, `ACK=1`.
   - El servidor elige su propio ISN, ej. `Seq = 2001`.
   - Acusa recibo del ISN del cliente: `Ack = 1002` (ISN cliente + 1).

3. **ACK**  
   - El cliente envía `ACK=1`, `SYN=0`.
   - `Seq = 1002` (ISN cliente + 1).
   - `Ack = 2002` (ISN servidor + 1).

Tras esto, la conexión está **establecida** y comienza la transferencia de datos.

```text
Cliente                      Servidor
  | SYN (Seq=1001)            |
  |-------------------------> |
  |                           |
  |<------------------------- |
  | SYN-ACK (Seq=2001, Ack=1002)
  |                           |
  | SYN-ACK (Seq=1002, Ack=2002)
  |-------------------------> |
  |                           |
  |   Conexión establecida    |
```

---

## 6. Puertos TCP y Rangos

TCP usa **números de puerto de 16 bits** (0–65535) para identificar aplicaciones/servicios [web:23][web:29].

### Rangos de puertos

| Rango            | Nombre              | Descripción                              |
|------------------|---------------------|------------------------------------------|
| 0–1023           | **Bien conocidos**  | Servicios del sistema/privilegiados (root)|
| 1024–49151       | **Registrados**     | Aplicaciones de usuario, servicios comunes|
| 49152–65535      | **Dinámicos/Privados**| Puertos efímeros para conexiones cliente |

### Puertos bien conocidos comunes

| Puerto | Protocolo | Servicio        |
|--------|-----------|-----------------|
| 20     | TCP       | FTP (datos)     |
| 21     | TCP       | FTP (control)   |
| 22     | TCP       | SSH             |
| 23     | TCP       | Telnet          |
| 25     | TCP       | SMTP            |
| 53     | TCP/UDP   | DNS             |
| 80     | TCP       | HTTP            |
| 443    | TCP       | HTTPS           |
| 3389   | TCP       | RDP             |

Los clientes suelen usar **puertos efímeros** (rango dinámico) como puertos origen al conectarse a servidores.

---

## 7. Resumen rápido

- **Modelo OSI**: 7 capas desde Física (1) hasta Aplicación (7).
- **IP (capa 3)**: enruta paquetes usando IP origen/destino.
- **Cabecera IPv4**: contiene versión, longitud, TTL, protocolo, checksum, IPs, etc.
- **TCP**: fiable, orientado a conexión, usa handshake de 3 vías.
- **UDP**: rápido, no orientado a conexión, sin garantía de entrega.
- **Puertos**: identifican servicios; bien conocidos (0–1023), registrados (1024–49151), dinámicos (49152–65535).
