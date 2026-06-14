# Host Discovery

Host discovery es el proceso de identificar qué hosts están **activos/en línea** en una red. En lugar de escanear todos los puertos de todas las IPs, primero encontramos las máquinas activas para enfocar nuestros esfuerzos.

## ¿Qué es Host Discovery?

En pentesting y reconocimiento de redes, host discovery (también llamado **ping scan**) reduce un rango grande de IPs a una lista más pequeña de hosts activos. Nmap soporta muchas técnicas de descubrimiento además del simple ping ICMP :

- Algunos hosts bloquean solicitudes de echo ICMP (ping clásico).
- Los firewalls pueden filtrar ciertos tipos de tráfico.
- Diferentes técnicas funcionan mejor en diferentes entornos de red.

---

## Técnicas de Host Discovery

### 1. Ping Sweeps (Ping ICMP Echo)

Un **ping sweep** envía paquetes ICMP echo request (tipo 8) a múltiples hosts y espera respuestas ICMP echo reply (tipo 0). Si un host responde, está activo.

- **Opción de Nmap**: `-PE`
- **Cómo funciona**:
  - Enviar ICMP Echo Request → si Echo Reply → host está activo.
- **Pros**:
  - Simple, ampliamente entendido.
  - Rápido en redes internas.
- **Contras**:
  - Muchos hosts y firewalls **bloquean ICMP echo** ahora.
  - No es fiable en Internet contra objetivos desconocidos.

Ejemplo:
```bash
nmap -sn -PE 192.168.1.0/24
```

---

### 2. ARP Scanning (ARP Ping)

**ARP scanning** se usa en **redes Ethernet locales** (LANs). Envía solicitudes ARP preguntando "¿Quién tiene la IP X?" y espera respuestas ARP.

- **Opción de Nmap**: `-PR` (por defecto en Ethernet local)
- **Cómo funciona**:
  - Enviar solicitud ARP: "¿Quién tiene 192.168.1.5?"
  - Si el host responde con ARP response → host está activo.
- **Pros**:
  - Muy **rápido** y **preciso** en LANs.
  - Los hosts generalmente **no pueden bloquear ARP** y seguir comunicándose en la red.
- **Contras**:
  - Solo funciona en **redes locales** (misma subred).
  - No puede usarse para escaneo remoto a través de routers.

Ejemplo:
```bash
nmap -sn -PR 192.168.1.0/24
```

---

### 3. TCP SYN Ping (Half-Open Scan)

**TCP SYN ping** envía un paquete TCP con la bandera **SYN** puesta (como el primer paso del handshake TCP), pero nunca completa la conexión. Por eso se llama **half-open scan**.

- **Opción de Nmap**: `-PS<puerto>` (ej. `-PS80`, `-PS22,80,443`)
- **Cómo funciona**:
  1. Enviar paquete TCP SYN a un puerto objetivo.
  2. Si el puerto está **cerrado**: el host responde con **RST** → host está activo.
  3. Si el puerto está **abierto**: el host responde con **SYN/ACK** → host está activo.
  4. Nmap entonces envía **RST** para cortar la conexión (nunca envía ACK).

- **Pros**:
  - A menudo **bypassea firewalls** que bloquean ICMP.
  - Funciona bien cuando apuntamos a servicios comunes (HTTP puerto 80, HTTPS 443, SSH 22).
- **Contras**:
  - Requiere capacidad de **paquetes crudos** (normalmente root en Unix).
  - Usuarios sin privilegios usan workaround `connect()` (menos eficiente).

Ejemplo:
```bash
nmap -sn -PS80,443 192.168.1.0/24
```

Esta técnica se llama **half-open** porque nunca completamos el handshake TCP de 3 vías.

---

### 4. UDP Ping

**UDP ping** envía paquetes UDP a puertos específicos. Si un puerto está **cerrado**, el objetivo típicamente responde con un mensaje **ICMP port unreachable**, lo que le dice a Nmap que el host está activo.

- **Opción de Nmap**: `-PU<puerto>` (ej. `-PU53`, `-PU161`)
- Puerto por defecto: **40125** (puerto poco común).
- **Cómo funciona**:
  1. Enviar paquete UDP a un puerto.
  2. Si el puerto está **cerrado**: recibir **ICMP port unreachable** → host está activo.
  3. Si el puerto está **abierto**: la mayoría de servicios ignoran el paquete → sin respuesta.
  4. Falta de respuesta o ciertos errores ICMP → host puede estar down/inaccesible.

- **Pros**:
  - Bypassea firewalls que **filtran TCP pero permiten UDP**.
  - Útil cuando los probes TCP están bloqueados.
- **Contras**:
  - Más lento que TCP (UDP es connectionless, muchos timeouts).
  - Menos fiable porque muchos hosts no envían errores ICMP.

Ejemplo:
```bash
nmap -sn -PU53,161,40125 192.168.1.0/24
```

---

### 5. TCP ACK Ping

**TCP ACK ping** envía un paquete TCP con la bandera **ACK** puesta. Este paquete parece estar reconociendo datos en una **conexión existente**, pero no existe tal conexión.

- **Opción de Nmap**: `-PA<puerto>` (ej. `-PA80`, `-PA443`)
- **Cómo funciona**:
  1. Enviar paquete TCP ACK a un puerto.
  2. El host debería responder con **RST** (porque no existe conexión) → host está activo.
- **Pros**:
  - Puede **bypassar firewalls stateless** que bloquean SYN entrante pero permiten ACK.
  - Útil cuando SYN ping está filtrado.
- **Contras**:
  - Menos efectivo contra **firewalls stateful** que descartan paquetes ACK inesperados.
  - Como SYN ping, normalmente requiere root en Unix.

Ejemplo:
```bash
nmap -sn -PA80,443 192.168.1.0/24
```

---

## Comparación de Técnicas de Host Discovery

| Técnica              | Tipo de Paquete     | Opción Nmap | Funciona Mejor en             | Bypassea Bloqueo ICMP? |
|----------------------|---------------------|-------------|-------------------------------|------------------------|
| Ping Sweep (ICMP)    | ICMP Echo Request   | `-PE`       | Redes internas                | No                     |
| ARP Scan             | ARP Request         | `-PR`       | Ethernet local (LAN)          | N/A (no usa ICMP)      |
| TCP SYN Ping         | TCP SYN             | `-PS`       | Internet, servicios comunes   | Sí                     |
| TCP ACK Ping         | TCP ACK             | `-PA`       | Firewalls stateless           | Sí                     |
| UDP Ping             | UDP                 | `-PU`       | Firewalls que permiten UDP    | Sí                     |

---

## Host Discovery por Defecto en Nmap

Si no especificas ningún tipo de ping, Nmap usa una **combinación por defecto**:

- Para **usuarios con privilegios** (root):
  - `-PE -PS443 -PA80 -PP`
  - ICMP echo + TCP SYN al 443 + TCP ACK al 80 + ICMP timestamp.
- Para **Ethernet local**:
  - Usa **ARP scan** por defecto (`-PR`), incluso si especificas otras opciones.
- Para **usuarios sin privilegios**:
  - `-PS80,443` usando llamada al sistema `connect()`.

Ejemplo de host discovery solo (sin escaneo de puertos):
```bash
nmap -sn 192.168.1.0/24
```

La opción `-sn` le dice a Nmap: **solo hacer host discovery, sin escanear puertos**.

---

## Puntos Clave

- **Ping sweeps** (ICMP) son simples pero a menudo bloqueados.
- **ARP scanning** es el más fiable en LANs locales.
- **TCP SYN ping** es un half-open scan; genial para bypassar filtros ICMP.
- **UDP ping** es útil cuando TCP está filtrado pero UDP permitido.
- **TCP ACK ping** puede bypassar firewalls stateless que bloquean SYN.
- En la práctica, usa **múltiples técnicas** juntas para maximizar la precisión de host discovery.
