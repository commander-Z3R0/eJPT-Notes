# Port Scanning

Port scanning es el proceso de identificar **puertos abiertos** en un host objetivo. Una vez que sabes que un host está activo (del host discovery), port scanning te dice **qué servicios se están ejecutando** y potencialmente vulnerables.

## ¿Qué es Port Scanning?

En pentesting y reconocimiento, port scanning te ayuda a:

- Descubrir **puertos abiertos** (servicios que aceptan conexiones).
- Identificar **servicios en ejecución** (HTTP, SSH, SMTP, SMB, etc.).
- Determinar **versiones de servicios** (para explotación).
- Mapear la **superficie de ataque** del objetivo.

Nmap soporta muchas técnicas de escaneo, cada una con diferentes ventajas dependiendo de la red y el firewall :

- Algunos escaneos son **sigilosos** (más difíciles de detectar).
- Algunos escaneos funcionan mejor **detrás de firewalls**.
- Algunos requieren **privilegios de root**, otros no.

---

## Técnicas de Port Scanning

### 1. TCP SYN Scan (Stealth Scan)

**TCP SYN scan** es el escaneo **por defecto y más popular** en Nmap. Envía paquetes TCP SYN y analiza las respuestas sin completar el handshake TCP. Por eso se llama **half-open scan**.

- **Opción de Nmap**: `-sS`
- **Requiere**: privilegios de root/administrador (acceso a paquetes crudos)
- **Cómo funciona**:
  1. Enviar paquete TCP SYN al puerto objetivo.
  2. Si el puerto está **abierto**: recibir **SYN/ACK** → puerto abierto.
  3. Si el puerto está **cerrado**: recibir **RST** → puerto cerrado.
  4. Si **filtrado**: sin respuesta o error ICMP → puerto filtrado.
  5. Nmap envía **RST** para cortar (nunca envía ACK).

- **Pros**:
  - **Rápido** y **sigiloso** (no completa conexiones).
  - Funciona en la mayoría de entornos.
  - Estándar para pentesting.
- **Contras**:
  - Requiere **root** en Unix/Linux.
  - Aún puede ser registrado por IDS/IPS.

Ejemplo:
```bash
nmap -sS <target_ip>
nmap -sS -p 22,80,443 <target_ip>
nmap -sS -p- <target_ip>        # escanear todos los 65535 puertos
```

---

### 2. TCP Connect Scan

**TCP connect scan** usa la función **connect()** del sistema para completar el handshake TCP completo. Esto se usa cuando **no tienes privilegios de root**.

- **Opción de Nmap**: `-sT`
- **Requiere**: sin privilegios de root (nivel de usuario)
- **Cómo funciona**:
  1. Enviar TCP SYN.
  2. Si el puerto está **abierto**: recibir SYN/ACK → enviar ACK → conexión establecida.
  3. Si el puerto está **cerrado**: recibir RST.
  4. Nmap cierra la conexión inmediatamente después.

- **Pros**:
  - Funciona **sin privilegios de root**.
  - Confiable y bien soportado.
- **Contras**:
  - **Más ruidoso** (completa conexiones completas, más fácil de registrar).
  - **Más lento** que SYN scan.

Ejemplo:
```bash
nmap -sT <target_ip>
nmap -sT -p 80,443 <target_ip>
```

---

### 3. UDP Scan

**UDP scan** sondea puertos UDP para encontrar servicios UDP abiertos. UDP es **sin conexión**, así que el escaneo es más lento y menos confiable que TCP.

- **Opción de Nmap**: `-sU`
- **Requiere**: privilegios de root (usualmente)
- **Cómo funciona**:
  1. Enviar paquete UDP vacío (o específico del protocolo) al puerto.
  2. Si recibes **ICMP port unreachable** → puerto **cerrado**.
  3. Si recibes **respuesta UDP** → puerto **abierto**.
  4. Si **sin respuesta** → puerto **open|filtered** (incierto).

- **Pros**:
  - Descubre **servicios UDP** (DNS, DHCP, SNMP, NTP, etc.).
  - Importante para reconocimiento completo.
- **Contras**:
  - **Muy lento** (muchos timeouts).
  - Menos confiable (muchos hosts no envían errores ICMP).
  - A menudo filtrado por firewalls.

Ejemplo:
```bash
nmap -sU <target_ip>
nmap -sU -p 53,161,123 <target_ip>      # DNS, SNMP, NTP
nmap -sU --top-ports 100 <target_ip>    # top 100 puertos UDP
```

---

### 4. TCP ACK Scan

**TCP ACK scan** envía paquetes TCP con la bandera **ACK** puesta. Se usa principalmente para **mapear reglas de firewall**, no para encontrar puertos abiertos.

- **Opción de Nmap**: `-sA`
- **Requiere**: privilegios de root
- **Cómo funciona**:
  1. Enviar paquete TCP ACK al puerto.
  2. Si recibes **RST** → puerto **unfiltered**.
  3. Si **sin respuesta** o error ICMP → puerto **filtrado**.

- **Pros**:
  - Bueno para **mapear firewalls**.
  - Bypassea algunos **firewalls stateless**.
- **Contras**:
  - **No puede determinar abierto vs cerrado** (solo unfiltered vs filtered).
  - No útil para encontrar servicios.

Ejemplo:
```bash
nmap -sA <target_ip>
nmap -sA -p 80,443 <target_ip>
```

---

### 5. TCP Window Scan

**TCP window scan** es similar a ACK scan pero verifica el **tamaño de ventana TCP** en la respuesta RST para determinar puertos abiertos vs cerrados.

- **Opción de Nmap**: `-sW`
- **Requiere**: privilegios de root
- **Cómo funciona**:
  1. Enviar paquete TCP ACK.
  2. Si RST tiene **ventana no cero** → puerto **abierto**.
  3. Si RST tiene **ventana cero** → puerto **cerrado**.

- **Pros**:
  - Puede detectar puertos abiertos (a diferencia de ACK scan).
  - A veces bypassea firewalls.
- **Contras**:
  - Raramente más útil que SYN scan.
  - No confiable en todos los sistemas.

Ejemplo:
```bash
nmap -sW <target_ip>
```

---

### 6. TCP NULL, FIN y Xmas Scans

Estos son **escaneos sigilosos** que envían paquetes **sin flags** (NULL), **solo FIN**, o **FIN+URG+PSH** (Xmas). Explotan cómo los sistemas manejan paquetes malformados.

- **Opciones de Nmap**:
  - `-sN` (NULL scan)
  - `-sF` (FIN scan)
  - `-sX` (Xmas scan)
- **Requiere**: privilegios de root
- **Cómo funciona**:
  - Enviar paquete **sin SYN** (bypassea algunos firewalls stateful).
  - Si recibes **RST** → puerto **cerrado**.
  - Si **sin respuesta** → puerto **open|filtered**.

- **Pros**:
  - **Sigiloso** (sin flag SYN, más difícil de detectar).
  - Puede bypassar algunos firewalls.
- **Contras**:
  - Solo funciona en **Unix/Linux** (Windows responde diferente).
  - No confiable; muchos sistemas modernos responden con RST de todos modos.

Ejemplo:
```bash
nmap -sN <target_ip>
nmap -sF <target_ip>
nmap -sX <target_ip>
```

---

## Plantillas de Tiempo de Nmap (`-T`)

Las plantillas de tiempo de Nmap controlan la **velocidad y agresividad** del escaneo. Valores más altos son más rápidos pero más propensos a ser detectados.

| Opción | Nombre | Descripción | Caso de Uso |
|--------|--------|-------------|-------------|
| `-T0` | **Paranoid** | Extremadamente lento; enfocado en evasión | Evasión de IDS, muy sigiloso |
| `-T1` | **Sneaky** | Muy lento; evita detección | Escaneo sigiloso |
| `-T2` | **Polite** | Lento; reduce uso de ancho de banda | Redes con bajo ancho de banda |
| `-T3` | **Normal** | Velocidad por defecto | Escaneo estándar |
| `-T4` | **Aggressive** | Rápido; asume red moderna | Escaneos rápidos en redes confiables |
| `-T5` | **Insane** | Muy rápido; alto riesgo de errores | Velocidad sobre precisión |

---

## Selección de Puertos

Puedes especificar qué puertos escanear usando `-p` :

| Opción | Descripción |
|--------|-------------|
| `-p 80,443,22` | Escanear puertos específicos |
| `-p 1-1000` | Escanear puertos 1 a 1000 |
| `-p-` | Escanear **todos los 65535 puertos** |
| `--top-ports 100` | Escanear **top 100 puertos más comunes** |
| `-p U:53,T:80,443` | Escanear UDP 53 + TCP 80,443 |

Ejemplo:
```bash
nmap -sS -p 22,80,443 <target_ip>
nmap -sS -p- <target_ip>
nmap -sS --top-ports 1000 <target_ip>
```

---

## Detección de Versión de Servicio

**-sV** sondea puertos abiertos para determinar la **versión del servicio** (ej. Apache 2.4.49, OpenSSH 8.4):

- **Opción de Nmap**: `-sV`
- **Uso**: cuando necesitas saber versiones exactas para explotación.

Ejemplo:
```bash
nmap -sS -sV <target_ip>
nmap -sS -sV --version-intensity 5 <target_ip>   # intensidad máxima
```

---

## Detección de OS

**-O** intenta detectar el **sistema operativo** del objetivo:

- **Opción de Nmap**: `-O`
- **Requiere**: privilegios de root
- **Cómo funciona**: Analiza huellas digitales de pila TCP/IP.

Ejemplo:
```bash
nmap -sS -O <target_ip>
```

---

## Escaneo Agresivo (A)

**-A** habilita **detección de OS, detección de versión, escaneo de scripts y traceroute** todo a la vez :

- **Opción de Nmap**: `-A`
- **Equivalente a**: `-O -sV -sC --traceroute`
- **Uso**: cuando quieres máxima información.

Ejemplo:
```bash
nmap -A <target_ip>
```

---

## Nmap Scripting Engine (NSE)

**-sC** ejecuta **scripts NSE por defecto** para reconocimiento adicional (vulnerabilidades, configs, etc.):

- **Opción de Nmap**: `-sC`
- **Uso**: cuando quieres ejecutar verificaciones comunes de vulnerabilidades.

Ejemplo:
```bash
nmap -sS -sC <target_ip>
nmap --script vuln <target_ip>
nmap --script http-enum <target_ip>
```

---

## Combinaciones Comunes de Escaneo

| Escenario | Comando |
|-----------|---------|
| **Top 100 puertos rápido** | `nmap -sS --top-ports 100 <objetivo>` |
| **Escaneo TCP completo** | `nmap -sS -p- <objetivo>` |
| **Escaneo completo + versión + OS** | `nmap -sS -sV -O -p- <objetivo>` |
| **Escaneo agresivo** | `nmap -A <objetivo>` |
| **Escaneo UDP (top 100)** | `nmap -sU --top-ports 100 <objetivo>` |
| **TCP + UDP completo** | `nmap -sS -sU -p- <objetivo>` |
| **Sin root** | `nmap -sT -p 80,443 <objetivo>` |
| **Mapping de firewall** | `nmap -sA <objetivo>` |
| **Omitir descubrimiento de host (-Pn)** | `nmap -sS -Pn <objetivo>` |

---

## Estados de Puertos

Nmap reporta puertos en estos estados :

| Estado | Significado |
|--------|-------------|
| **open** | Servicio está aceptando conexiones |
| **closed** | Host es alcanzable, pero no hay servicio en el puerto |
| **filtered** | Firewall/IDS está bloqueando el escaneo |
| **unfiltered** | Puerto es accesible, pero abierto/cerrado desconocido |
| **open\|filtered** | Nmap no puede determinar si abierto o filtrado |
| **closed\|filtered** | Nmap no puede determinar si cerrado o filtrado |

---

## Puntos Clave

- **TCP SYN scan (`-sS`)** es el por defecto; rápido, sigiloso, requiere root.
- **TCP Connect scan (`-sT`)** es para usuarios sin root; más lento, más visible.
- **UDP scan (`-sU`)** es lento pero necesario para servicios UDP (DNS, SNMP).
- **ACK scan (`-sA`)** mapea reglas de firewall, no puertos abiertos.
- **NULL/FIN/Xmas scans** son sigilosos pero poco confiables en Windows.
- **`-sV`** sondea versiones de servicios para explotación.
- **`-O`** detecta el OS del objetivo.
- **`-A`** es agresivo: OS + versión + scripts + traceroute.
- **`-sC`** ejecuta scripts NSE por defecto para verificaciones de vulnerabilidades.
- Usa **`-p-`** para escaneo completo de puertos (todos los 65535 puertos).
- Usa **`--top-ports 100`** para escaneo rápido de los puertos más comunes.
