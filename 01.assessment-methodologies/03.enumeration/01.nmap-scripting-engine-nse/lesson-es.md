# Nmap Scripting Engine (NSE)

El **Nmap Scripting Engine (NSE)** es una de las funciones más poderosas de Nmap. Te permite ejecutar **scripts** contra hosts objetivo para realizar **enumeración avanzada de servicios**, **detección de vulnerabilidades** y **recolección de información** de manera automatizada y personalizable.

## ¿Qué es NSE?

NSE usa **scripts en Lua** para extender las capacidades de Nmap más allá del escaneo simple de puertos. Cada script está diseñado para realizar una tarea específica, como:

- Enumerar **usuarios** (FTP, SMTP, SMB, SSH).
- Detectar **versiones de servicios** y configuraciones.
- Encontrar **vulnerabilidades** (CVEs, malconfiguraciones).
- Recopilar estructuras de **directorios web**.
- Extraer **registros DNS** e información de red.

Los scripts de NSE se almacenan en `/usr/share/nmap/scripts/` en la mayoría de sistemas Linux y están categorizados por función (default, discovery, vuln, brute, auth, etc.).

---

## Ejecutar Scripts NSE

### Scripts por Defecto (`-sC`)
El flag `-sC` ejecuta la **categoría predeterminada de scripts NSE**, que incluye scripts seguros y comúnmente útiles para enumeración básica:

```bash
nmap -sS -sC <target_ip>
```

### Scripts Específicos (`--script`)
Puedes ejecutar scripts o categorías de scripts específicos por nombre:

```bash
nmap -sS --script http-enum <target_ip>
nmap -sS --script vuln <target_ip>
nmap -sS --script ftp-anon,ftp-bounce <target_ip>
nmap -sS --script smb-enum-shares,smb-enum-users <target_ip>
```

### Categorías de Scripts
Los scripts NSE están organizados en categorías:

| Categoría | Propósito |
|-----------|-----------|
| **default** | Scripts seguros activados con `-sC` |
| **discovery** | Descubrimiento de servicios y hosts |
| **vuln** | Detección de vulnerabilidades |
| **auth** | Pruebas de autenticación |
| **brute** | Ataques de fuerza bruta |
| **http** | Enumeración de servidores web |
| **smb** | Enumeración SMB |
| **ftp** | Enumeración FTP |
| **ssh** | Enumeración SSH |
| **smtp** | Enumeración SMTP |

Ejemplo:
```bash
nmap -sS --script default <target_ip>
nmap -sS --script discovery <target_ip>
nmap -sS --script vuln <target_ip>
```

---

## Scripts NSE Comunes por Servicio

### Enumeración FTP
```bash
nmap -sS --script ftp-anon <target_ip>
nmap -sS --script ftp-bounce,ftp-libopie <target_ip>
```

### Enumeración SMB
```bash
nmap -sS --script smb-enum-shares,smb-enum-users <target_ip>
nmap -sS --script smb-security-mode,smb-os-discovery <target_ip>
```

### Enumeración SSH
```bash
nmap -sS --script ssh2-enum-algos,ssh-hostkey <target_ip>
```

### Enumeración SMTP
```bash
nmap -sS --script smtp-enum-users,smtp-commands <target_ip>
```

### Enumeración de Servidor Web
```bash
nmap -sS --script http-enum,http-headers <target_ip>
nmap -sS --script http-sql-injection,xss <target_ip>
```

### Enumeración MySQL
```bash
nmap -sS --script mysql-enum,mysql-info <target_ip>
```

---

## Metodología Completa de Enumeración (IP → Servicios → Metasploit)

Esta metodología muestra cómo ir desde **enumeración cruda de IP** hasta un **workspace organizado de Metasploit** con datos de escaneo importados, todo solo para propósitos de **footprinting y reconocimiento** (sin explotación).

---

### Paso 1: Escaneo Inicial de IP con NSE

Comienza con un escaneo básico para descubrir puertos abiertos y servicios:

```bash
nmap -sS -sV -O -p- -sC -oX initial_scan.xml <target_ip>
```

Qué hace esto:
- `-sS`: Escaneo TCP SYN (sigiloso, por defecto)
- `-sV`: Detectar versiones de servicios
- `-O`: Intentar detección de OS
- `-p-`: Escanear los 65535 puertos
- `-sC`: Ejecutar scripts NSE por defecto
- `-oX`: Exportar resultados a XML para importación en Metasploit

Esto te da puertos, servicios, versiones, huella digital del OS y enumeración básica desde scripts NSE.

---

### Paso 2: Enumeración NSE Avanzada por Servicio

Basado en lo que encontraste, ejecuta scripts NSE dirigidos:

```bash
# Chequeo de acceso anónimo FTP
nmap -sS --script ftp-anon -p 21 <target_ip>

# Shares y usuarios SMB
nmap -sS --script smb-enum-shares,smb-enum-users -p 139,445 <target_ip>

# Directorios web y vulnerabilidades
nmap -sS --script http-enum,http-vuln* -p 80,443 <target_ip>

# Enumeración de usuarios SMTP
nmap -sS --script smtp-enum-users -p 25 <target_ip>

# Algoritmos y hostkey SSH
nmap -sS --script ssh2-enum-algos -p 22 <target_ip>
```

Guarda cada resultado para referencia posterior:
```bash
nmap -sS --script ftp-anon -p 21 -oN ftp_enum.txt <target_ip>
nmap -sS --script smb-enum-shares -p 445 -oN smb_enum.txt <target_ip>
```

---

### Paso 3: Exportar Resultados a XML

Ya usaste `-oX` en el Paso 1, pero también puedes exportar escaneos adicionales:

```bash
nmap -sS --script smb-enum-shares -oX smb_scan.xml <target_ip>
nmap -sS --script http-enum -oX web_scan.xml <target_ip>
```

El formato XML es crítico porque **Metasploit puede importarlo** y almacenar todos los hosts, puertos y datos de servicios en su base de datos.

---

### Paso 4: Lanzar Metasploit y Crear un Workspace

Abre la consola de Metasploit:
```bash
msfconsole
```

Crea un nuevo workspace para este objetivo (para mantener todo organizado):
```bash
workspace -a enumeration_target_<target_ip>
```

Opciones de workspace:
- `-a`: Agregar nuevo workspace
- `-w`: Cambiar a workspace existente
- `-d`: Eliminar workspace
- `-l`: Listar workspaces

Esto asegura que todos tus datos de escaneo, notas y módulos futuros permanezcan separados de otras evaluaciones.

---

### Paso 5: Importar XML de Nmap a la Base de Datos de Metasploit

Importa tu archivo XML de escaneo:
```bash
db_import initial_scan.xml
```

Este comando analiza el XML y almacena **hosts, puertos, servicios y scripts** en la base de datos de Metasploit automáticamente.

Puedes importar múltiples archivos a la vez:
```bash
db_import *.xml
```

---

### Paso 6: Listar Hosts y Servicios

Ahora verifica qué se importó:

```bash
# Listar todos los hosts en la base de datos
hosts

# Listar todos los servicios en la base de datos
services

# Filtrar por puerto
services -p 22

# Filtrar por protocolo
services -p 445 -p 3389
```

También puedes usar `db_nmap` para ejecutar Nmap directamente desde Metasploit con almacenamiento automático en base de datos:

```bash
db_nmap -sS -sV -oX db_scan.xml <target_ip>
db_nmap -sS -Pn -p 80,443 <target_ip>
```

El flag `-Pn` es útil cuando el objetivo bloquea ping pero aún tiene puertos abiertos.

---

### Paso 7: Listar Vulnerabilidades

Metasploit puede mostrar vulnerabilidades detectadas desde escaneos importados:

```bash
vulns
```

Esto listará cualquier vulnerabilidad encontrada por scripts NSE (como scripts de categoría `vuln`) que fueron almacenados en la base de datos.

---

### Paso 8: Usar Módulos Auxiliares de Metasploit para Enumeración

Metasploit tiene **módulos auxiliares** diseñados específicamente para **reconocimiento y enumeración** (no explotación). Estos son perfectos para la fase de footprinting:

```bash
# Buscar módulos auxiliares
search auxiliary

# Chequeo de login anónimo FTP
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run

# Enumeración SMB
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run

# Enumeración de usuarios SMB
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run

# Chequeo de versión SSH
use auxiliary/scanner/ssh/ssh_version
set RHOSTS <target_ip>
run

# Enumeración de usuarios SMTP
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <target_ip>
run

# Enumeración de directorios HTTP
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set THREADS 10
run
```

Categorías clave de escáneres auxiliares:
- `auxiliary/scanner/ftp/`
- `auxiliary/scanner/smb/`
- `auxiliary/scanner/ssh/`
- `auxiliary/scanner/smtp/`
- `auxiliary/scanner/http/`
- `auxiliary/scanner/mysql/`

Estos módulos realizan **enumeración pasiva o activa** sin explotar vulnerabilidades, manteniéndote en la **fase de footprinting/reconocimiento**.

---

### Paso 9: Organizar y Documentar

Después de la enumeración, documenta tus hallazgos:

```bash
# Exportar datos del workspace
db_export -f xml workspace_export.xml

# Listar todos los hosts descubiertos
hosts -d

# Listar todos los servicios con detalles
services -v
```

Guarda tus resultados de escaneo, salida NSE y exportaciones de base de datos de Metasploit para tu informe final.

---

## NSE vs Módulos Auxiliares de Metasploit

| Característica | Scripts NSE | Auxiliares Metasploit |
|----------------|-------------|----------------------|
| **Ubicación** | Dentro de Nmap | Dentro de Metasploit |
| **Velocidad** | Muy rápido (herramienta única) | Más lento (más opciones) |
| **Integración** | Exportación manual necesaria | Almacenamiento automático en base de datos |
| **Scripting** | Basado en Lua | Basado en Ruby |
| **Mejor para** | Escaneos rápidos, enumeración rápida | Evaluaciones organizadas, flujo de trabajo con base de datos |
| **Caso de uso** | Footprinting inicial | Enumeración detallada, reportes |

---

## Puntos Clave

- **NSE** usa scripts en Lua para automatizar enumeración de servicios, detección de vulnerabilidades y recolección de información.
- `-sC` ejecuta scripts por defecto; `--script` ejecuta scripts o categorías específicas.
- Scripts comunes: `ftp-anon`, `smb-enum-shares`, `http-enum`, `smtp-enum-users`, `ssh2-enum-algos`.
- Exporta resultados de escaneo con **`-oX`** a XML para importación en Metasploit.
- Crea un **workspace de Metasploit** para mantener evaluaciones organizadas.
- Usa **`db_import`** para cargar XML de Nmap en la base de datos de Metasploit.
- Lista hosts con **`hosts`**, servicios con **`services`**, y vulnerabilidades con **`vulns`**.
- Usa **`db_nmap`** para ejecutar Nmap directamente desde Metasploit con almacenamiento automático en base de datos.
- Los **módulos auxiliares de Metasploit** (`auxiliary/scanner/*`) realizan enumeración sin explotación.
