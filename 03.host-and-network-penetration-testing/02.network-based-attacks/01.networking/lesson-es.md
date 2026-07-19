# Networking

El propósito de las redes es intercambiar información entre hosts. Los paquetes son simplemente bits (señales eléctricas) que, cuando un ordenador los traduce, forman información.

---

## Networking Fundamentals

Esta lección cubre los conceptos fundamentales de redes necesarios para las pruebas de penetración basadas en red — incluyendo cómo fluyen los datos, el modelo OSI, los protocolos clave de las capas de Red y Transporte, y los conceptos esenciales de direccionamiento IP.

### Cómo se Comunican las Redes

En una red informática, los hosts se comunican a través de **protocolos de red** — reglas estandarizadas que garantizan que diferentes sistemas puedan intercambiar información. Esta comunicación se lleva a cabo mediante **paquetes**.

Cada paquete tiene dos partes:

| Parte | Descripción |
|-------|-------------|
| **Cabecera** | Estructura específica del protocolo (TCP, UDP, IP) que indica al host destino cómo interpretar los datos. Contiene IPs origen/destino, versión, TTL, tipo de protocolo. |
| **Payload** | Los datos o información real que se transmiten. |

---

### El Modelo OSI

El modelo **OSI (Open Systems Interconnection)** estandariza la comunicación en red en 7 capas. No es un plano estricto, sino que ayuda a entender y diseñar la arquitectura de red.

| Capa | Nombre | Función | Protocolos Clave |
|------|--------|----------|-----------------|
| 7 | **Aplicación** | Servicios e interfaces orientados al usuario | HTTP, FTP, SMTP, DNS |
| 6 | **Presentación** | Formateo de datos, cifrado, compresión | SSL/TLS, JPEG |
| 5 | **Sesión** | Gestiona sesiones entre aplicaciones | NetBIOS, RPC |
| 4 | **Transporte** | Comunicación extremo a extremo, control de errores | TCP, UDP |
| 3 | **Red** | Direccionamiento lógico y enrutamiento | IP, ICMP, DHCP |
| 2 | **Enlace de Datos** | Direccionamiento físico, entrega de tramas | Ethernet, MAC, ARP |
| 1 | **Físico** | Transmisión de bits en bruto por el medio | Cables, Wi-Fi, fibra |

> Para el pentesting, las **Capas 3 (Red) y 4 (Transporte)** son las más críticas.

---

### Capa 3 — La Capa de Red

Responsable del **direccionamiento lógico**, el **enrutamiento** y el **reenvío de paquetes** entre dispositivos en diferentes redes.

#### Protocolos Clave

| Protocolo | Uso |
|-----------|-----|
| **IPv4** | Versión IP más usada — direcciones de 32 bits (ej. `192.168.1.1`) |
| **IPv6** | Sucesor de IPv4 — direcciones de 128 bits |
| **ICMP** | Protocolo de Mensajes de Control de Internet — diagnósticos y reporte de errores |
| **DHCP** | Protocolo de Configuración Dinámica de Host — asigna IPs automáticamente |

#### Funciones Principales de IP

| Función | Descripción |
|---------|-------------|
| **Direccionamiento lógico** | Las IPs identifican de forma única cada dispositivo |
| **Estructura de paquetes** | Datos organizados en paquetes con cabecera y payload |
| **Fragmentación y reensamblado** | Paquetes grandes divididos para ajustarse al MTU, reensamblados en el destino |
| **Enrutamiento** | Paquetes reenviados salto a salto hacia la IP destino |

#### Tipos de Direccionamiento IP

| Tipo | Descripción |
|------|-------------|
| **Unicast** | Comunicación de uno a uno |
| **Broadcast** | Comunicación de uno a todos dentro de una subred |
| **Multicast** | Comunicación de uno a muchos para un grupo selecto |

#### Direcciones IPv4 Reservadas (RFC 5735)

| Rango | Uso |
|-------|-----|
| `0.0.0.0 – 0.255.255.255` | Representa la red a la que estás conectado |
| `127.0.0.0 – 127.255.255.255` | Loopback / localhost (tu propia máquina) |
| `192.168.0.0 – 192.168.255.255` | Redes privadas (LAN) |
| `10.0.0.0 – 10.255.255.255` | Redes privadas (LAN) |
| `172.16.0.0 – 172.31.255.255` | Redes privadas (LAN) |

#### Campos de la Cabecera IPv4

| Campo | Tamaño | Descripción |
|-------|--------|-------------|
| **IP Origen** | 32 bits | Dirección de origen del paquete |
| **IP Destino** | 32 bits | Dirección de destino del paquete |
| **TTL** | 8 bits | Se decrementa en cada salto — paquete descartado cuando llega a 0 |
| **Protocolo** | 8 bits | Protocolo de transporte (6 = TCP, 17 = UDP, 1 = ICMP) |
| **ToS** | 8 bits | Prioridad del paquete para QoS |
| **Versión** | 4 bits | IPv4 o IPv6 |

#### ICMP — Protocolo de Mensajes de Control de Internet

| Mensaje ICMP | Tipo | Código | Uso |
|-------------|------|--------|-----|
| **Echo Request** | 8 | 0 | Enviado por `ping` para probar la accesibilidad del host |
| **Echo Reply** | 0 | 0 | Respuesta confirmando que el host está activo |
| **Destino Inaccesible** | 3 | — | Host o puerto inaccesible |
| **Tiempo Excedido** | 11 | 0 | TTL expirado en tránsito (usado por `traceroute`) |

---

### Capa 4 — La Capa de Transporte

Proporciona **comunicación extremo a extremo** entre aplicaciones. Maneja la detección de errores, el control de flujo y la segmentación de datos.

#### TCP — Protocolo de Control de Transmisión

**Orientado a conexión** — establece una conexión fiable antes de la transferencia de datos.

| Característica | Descripción |
|---------------|-------------|
| **Fiabilidad** | Datos enviados con precisión y en orden — usa ACK |
| **Retransmisión** | Segmentos perdidos reenviados automáticamente |
| **Ordenación** | Segmentos fuera de orden reordenados antes de la entrega |
| **Conexión** | Requiere handshake de 3 vías antes del intercambio de datos |

#### El Handshake TCP de 3 Vías

```
Cliente  →  SYN      →  Servidor    (Quiero conectarme)
Cliente  ←  SYN-ACK  ←  Servidor    (Confirmado, acepto)
Cliente  →  ACK      →  Servidor    (Conexión establecida)
```

> Los **escaneos SYN** (half-open) envían un SYN pero nunca completan el handshake — más rápidos y evitan el registro en el objetivo.

#### Flags de Control TCP

| Flag | Descripción |
|------|-------------|
| **SYN** | Inicia una solicitud de conexión |
| **ACK** | Confirma datos recibidos |
| **FIN** | Inicia la terminación ordenada de la conexión |
| **RST** | Resetea/termina abruptamente una conexión |
| **PSH** | Envía datos inmediatamente a la aplicación |
| **URG** | Marca datos como urgentes |

#### Rangos de Puertos TCP

| Rango | Tipo | Descripción |
|-------|------|-------------|
| **0 – 1023** | Bien conocidos | Reservados por IANA para servicios estándar |
| **1024 – 49151** | Registrados | Asignados por IANA para aplicaciones/vendedores |
| **49152 – 65535** | Dinámicos/efímeros | Puertos temporales usados por clientes |

#### Puertos Bien Conocidos Comunes

| Puerto | Protocolo | Servicio |
|--------|-----------|---------|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | HTTP Alternativo |

#### UDP — Protocolo de Datagramas de Usuario

**Sin conexión** — envía datos sin establecer una conexión primero.

| Característica | Descripción |
|---------------|-------------|
| **Velocidad** | Más rápido que TCP — sin sobrecarga de handshake |
| **Fiabilidad** | Sin garantía de entrega, ordenación ni corrección de errores |
| **Independencia** | Cada datagrama tratado de forma independiente |
| **Casos de uso** | DNS, DHCP, SNMP, VoIP, streaming, juegos online |

#### TCP vs UDP

| Característica | TCP | UDP |
|---------------|-----|-----|
| **Conexión** | Orientado a conexión (handshake 3 vías) | Sin conexión |
| **Fiabilidad** | Entrega garantizada con ACK | Sin garantía |
| **Velocidad** | Más lento | Más rápido |
| **Ordenación** | Datos en orden | Sin ordenación |
| **Corrección de errores** | Sí (retransmisión) | No |
| **Casos de uso** | HTTP, HTTPS, SSH, FTP, SMB | DNS, VoIP, streaming |

---

### Subnetting y CIDR

El **subnetting** divide una red IP grande en subredes más pequeñas — mejora la eficiencia y seguridad aislando el tráfico.

**CIDR (Classless Inter-Domain Routing)** expresa una dirección de red con su longitud de prefijo:

| CIDR | Máscara de Subred | Hosts Disponibles |
|------|------------------|------------------|
| `/8` | 255.0.0.0 | 16.777.214 |
| `/16` | 255.255.0.0 | 65.534 |
| `/24` | 255.255.255.0 | 254 |
| `/28` | 255.255.255.240 | 14 |
| `/30` | 255.255.255.252 | 2 |
| `/32` | 255.255.255.255 | 1 (host único) |

**Ejemplo**: Un scope de `200.200.0.0/16` = hasta 65.536 IPs. Durante un pentest, tu primer trabajo es determinar cuáles están activas.

---

### Puntos Clave — Networking Fundamentals

- Las redes se comunican mediante **paquetes** — cabecera (info de enrutamiento) + payload (datos).
- El **modelo OSI** tiene 7 capas — las Capas 3 y 4 son las más relevantes para el pentesting.
- **Capa 3 (IP)**: direccionamiento lógico, enrutamiento — protocolos clave: IPv4, IPv6, ICMP, DHCP.
- El **TTL** se decrementa en cada salto — llega a 0 y el paquete se descarta. Usado por `traceroute`.
- **ICMP**: Echo Request (tipo 8) / Echo Reply (tipo 0) = `ping`; tipo 11 = TTL expirado = `traceroute`.
- **TCP**: requiere handshake de 3 vías — fiable, ordenado, más lento. Los escaneos SYN explotan el handshake.
- **UDP**: sin conexión — más rápido, no fiable. Usado por DNS, DHCP, SNMP.
- **Flags TCP**: SYN, ACK, FIN, RST — cada uno desempeña un papel en los tipos de escaneo.
- Puertos **0–1023** = bien conocidos; **1024–49151** = registrados; **49152–65535** = efímeros.
- Un scope `/16` = hasta **65.534 hosts** — siempre empieza con descubrimiento de hosts antes del escaneo de puertos.

---

## Firewall Detection & IDS Evasion

Esta lección cubre cómo detectar firewalls y sistemas IDS/IPS durante un escaneo, y las técnicas de Nmap para evadir o bypassear el filtrado básico.

> ⚠️ **Nota**: Todos los comandos asumen que operas desde una máquina de ataque **Kali Linux**. Reemplaza `<target_ip>` con tu IP objetivo real.

---

### ¿Qué son los Firewalls y los IDS/IPS?

| Sistema | Nombre Completo | Propósito |
|---------|----------------|-----------|
| **Firewall** | — | Filtra tráfico según reglas — bloquea o permite paquetes por puerto, IP o protocolo |
| **IDS** | Sistema de Detección de Intrusos | Monitoriza el tráfico y **alerta** sobre actividad sospechosa — no bloquea |
| **IPS** | Sistema de Prevención de Intrusos | Monitoriza el tráfico y **bloquea activamente** la actividad sospechosa |

---

### Detección de Firewalls

#### Método 1: Análisis del Estado de Puertos

| Estado del Puerto | Significado | ¿Firewall Presente? |
|-----------------|-------------|---------------------|
| **Open** | El servicio está escuchando y respondiendo | No (o el puerto está permitido) |
| **Closed** | El host responde con RST — puerto no en uso | No hay firewall en este puerto |
| **Filtered** | Sin respuesta o ICMP inalcanzable | **Sí — un firewall está bloqueando** |

```bash
nmap -sS -p 80,443,22,3389 <target_ip>
```

#### Método 2: Escaneo ACK (`-sA`)

Diseñado para **mapear reglas de firewall** — no detecta puertos abiertos, detecta si están filtrados.

```bash
nmap -sA -p 80,443,22 <target_ip>
```

| Respuesta | Interpretación |
|-----------|----------------|
| **RST recibido** | Puerto no filtrado — el firewall permite los paquetes ACK |
| **Sin respuesta / ICMP inalcanzable** | Puerto filtrado — el firewall está bloqueando |

> El escaneo ACK distingue entre firewalls **stateful** (descarta ACK inesperados) y **stateless** (puede dejarlos pasar).

#### Método 3: Escaneo de Ventana (`-sW`)

Analiza el **tamaño de la ventana TCP** en las respuestas RST para diferenciar puertos abiertos de cerrados.

```bash
nmap -sW -p 80,443 <target_ip>
```

#### Método 4: Prueba Rápida

```bash
nmap -sS -p 80 <target_ip>   # Estado del puerto con escaneo SYN
nmap -sA -p 80 <target_ip>   # Filtrado/no filtrado con escaneo ACK
```

SYN = **filtered** + ACK = **unfiltered** → firewall stateful descartando SYN pero no ACK.

---

### Técnicas de Evasión de IDS/IPS

#### 1. Fragmentación de Paquetes (`-f`, `--mtu`)

Divide las sondas en fragmentos IP más pequeños — los IDS/firewalls más antiguos pueden fallar al inspeccionarlos.

```bash
nmap -sS -f <target_ip>           # Fragmentos de 8 bytes
nmap -sS -ff <target_ip>          # Fragmentos de 16 bytes
nmap -sS --mtu 24 <target_ip>     # Tamaño personalizado (múltiplo de 8)
```

#### 2. Escaneo con Señuelos (`-D`)

Mezcla IPs de origen falsas con el escaneo real — el objetivo ve tráfico de muchas IPs.

```bash
nmap -sS -D 10.10.10.5,10.10.10.6,ME <target_ip>   # Señuelos manuales
nmap -sS -D RND:5 <target_ip>                        # 5 señuelos aleatorios
```

#### 3. Falsificación del Puerto de Origen (`-g` / `--source-port`)

Algunos firewalls permiten tráfico de puertos de confianza (DNS 53, HTTP 80) sin inspección profunda.

```bash
nmap -sS -g 53 <target_ip>     # Parecer venir del puerto DNS
nmap -sS -g 80 <target_ip>     # Parecer venir del puerto HTTP
```

#### 4. Longitud de Datos Aleatoria (`--data-length`)

Añade bytes aleatorios a los paquetes — más difícil para los IDS basados en firmas.

```bash
nmap -sS --data-length 25 <target_ip>
```

#### 5. Deshabilitar Resolución DNS (`-n`)

Evita las consultas DNS — cada consulta es un posible punto de detección.

```bash
nmap -sS -n <target_ip>
```

#### 6. Falsificación de IP de Origen (`-S`)

Hace que el tráfico parezca venir de otro host. Las respuestas van a la IP falsificada — principalmente para **pruebas de reglas de IDS**.

```bash
nmap -sS -S 192.168.1.50 -e eth0 <target_ip>
```

#### 7. Falsificación de Dirección MAC (`--spoof-mac`)

Falsifica la MAC de origen — útil en LANs con filtrado basado en MAC.

```bash
nmap -sS --spoof-mac 0 <target_ip>            # MAC aleatoria
nmap -sS --spoof-mac Apple <target_ip>        # MAC de proveedor
nmap -sS --spoof-mac 00:11:22:33:44:55 <target_ip>  # MAC específica
```

---

### Evasión Basada en Tiempo

#### Plantillas de Tiempo (`-T`)

| Plantilla | Nombre | Velocidad | Sigilo | Caso de Uso |
|----------|--------|-----------|--------|-------------|
| `-T0` | Paranoid | Extremadamente lento | Máximo | Evitar la mayoría de IDS |
| `-T1` | Sneaky | Muy lento | Alto | Escaneos sigilosos |
| `-T2` | Polite | Lento | Medio | Reducir carga en la red |
| `-T3` | Normal | Por defecto | Por defecto | Punto de partida estándar |
| `-T4` | Aggressive | Rápido | Bajo | Redes internas confiables |
| `-T5` | Insane | Muy rápido | Ninguno | ⚠️ Alto riesgo de pérdida de paquetes |

```bash
nmap -sS -T1 <target_ip>    # Escaneo sigiloso lento
nmap -sS -T3 <target_ip>    # Escaneo estándar
nmap -sS -T4 <target_ip>    # Escaneo rápido interno
```

#### Controles de Tiempo Manuales

```bash
nmap -sS --scan-delay 200ms <target_ip>   # Retraso entre sondas
nmap --min-rate 50 <target_ip>             # Mínimo 50 paquetes/seg
nmap --host-timeout 5m <target_ip>         # Rendirse tras 5 minutos por host
```

---

### Formatos de Salida

Siempre guarda los resultados del escaneo para análisis, informes e importación a Metasploit.

| Flag | Formato | Uso |
|------|---------|-----|
| `-oN` | Texto normal | Notas legibles por humanos |
| `-oX` | XML | Importación a Metasploit, integración con herramientas |
| `-oG` | Grepable | Análisis con shell y scripts |
| `-oA` | Todos los formatos | Guarda los tres a la vez |

```bash
nmap -sS -oN scan.txt <target_ip>       # Texto
nmap -sS -oX scan.xml <target_ip>       # XML
nmap -sS -oA scan_results <target_ip>   # Todos los formatos
```

#### Importar XML en Metasploit

```bash
nmap -sS -sV -oX scan.xml <target_ip>
msfconsole
msf6 > db_import scan.xml
msf6 > hosts
msf6 > services
```

---

### Flujo de Trabajo de Evasión Combinado

```bash
# Fragmentación + señuelos + puerto de origen + sin DNS + temporización lenta + salida XML
nmap -sS -Pn -f -D RND:5 -g 53 -n -T2 --data-length 15 -oX evasion_scan.xml <target_ip>
```

| Flag | Propósito |
|------|-----------|
| `-sS` | Escaneo SYN sigiloso |
| `-Pn` | Omitir descubrimiento de hosts |
| `-f` | Fragmentar paquetes |
| `-D RND:5` | 5 IPs señuelo aleatorias |
| `-g 53` | Puerto de origen 53 (DNS) |
| `-n` | Sin resolución DNS |
| `-T2` | Temporización educada |
| `--data-length 15` | 15 bytes aleatorios añadidos |
| `-oX` | Guardar como XML |

---

### Resumen de Técnicas de Evasión

| Técnica | Flag | Evade |
|---------|------|-------|
| Fragmentación de paquetes | `-f`, `--mtu` | Filtros simples, IDS más antiguos |
| Escaneo con señuelos | `-D RND:n` | Detección basada en IP |
| Falsificación de puerto de origen | `-g <puerto>` | Reglas de firewall basadas en puertos |
| Longitud de datos aleatoria | `--data-length` | IDS basados en firmas |
| Deshabilitar DNS | `-n` | Detección basada en DNS |
| Falsificación de IP de origen | `-S` | Pruebas de reglas de IDS |
| Falsificación de MAC | `--spoof-mac` | Filtrado basado en MAC (LAN) |
| Temporización lenta | `-T0`, `-T1`, `--scan-delay` | Umbrales de IDS basados en tasa |
| Omitir descubrimiento de hosts | `-Pn` | Firewalls que bloquean ICMP |

---

### Puntos Clave — Firewall Detection & IDS Evasion

- Los **firewalls** filtran por puerto/IP/protocolo — detectados por estados **filtered** en la salida de Nmap.
- Los **IDS** alertan; los **IPS** bloquean — ambos analizan patrones y tasas de paquetes.
- **Escaneo ACK** (`-sA`): RST = no filtrado, sin respuesta = filtrado — mejor herramienta para mapear reglas de firewall.
- Los firewalls **stateful** descartan los ACK inesperados; los **stateless** pueden dejarlos pasar.
- La **fragmentación** (`-f`) = trozos de 8 bytes — efectiva contra sistemas más antiguos, no contra firewalls stateful modernos.
- Los **señuelos** (`-D RND:5`) ocultan al escáner real entre IPs de origen aleatorias.
- La **falsificación del puerto de origen** (`-g 53`) bypasea reglas de firewall que confían en el tráfico del puerto 53 o 80.
- La **temporización lenta** (`-T0`, `-T1`, `--scan-delay`) evade los IDS basados en tasa reduciendo paquetes por segundo.
- **`-Pn`** omite el descubrimiento de hosts por ICMP — usar cuando el objetivo bloquea los pings.
- **Nunca uses `-T5`** en evaluaciones reales — causa pérdida de paquetes y resultados inexactos.
- Mejor combinación de evasión: `-f -D RND:5 -g 53 -n -T2 --data-length 15`.
- Siempre guarda con **`-oX`** (XML) para importar a Metasploit con `db_import`.
