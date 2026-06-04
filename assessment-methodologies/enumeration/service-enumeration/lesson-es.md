# Service Enumeration

La **enumeración de servicios** es el proceso de identificar y analizar servicios de red que se ejecutan en un sistema objetivo. En pentesting, esta fase viene después del **footprinting y escaneo**, y se enfoca en recopilar información detallada sobre servicios como **FTP, SMB, Web (Apache), MySQL, SSH y SMTP** para encontrar malconfiguraciones, credenciales débiles y vulnerabilidades potenciales.

Esta lección cubre la **enumeración de servicios usando módulos auxiliares de Metasploit**, incluyendo escaneo de puertos, selección de módulo, configuración de RHOST y pruebas de credenciales con wordlists, todo dentro de la **fase de reconocimiento/footprinting** (sin explotación).

---

## Por Qué Importa la Enumeración de Servicios

La enumeración de servicios te ayuda a:

- Identificar **servicios en ejecución** y sus versiones.
- Descubrir **shares abiertos** (SMB), **acceso anónimo FTP** y **directorios web**.
- Encontrar **credenciales débiles o por defecto** (SSH, MySQL, FTP).
- Detectar **malconfiguraciones** que podrían llevar a acceso no autorizado.
- Recopilar inteligencia para **fases subsiguientes de pentesting**.

Metasploit proporciona **módulos auxiliares de escáner** diseñados específicamente para enumeración de servicios sin explotación.

---

## Flujo de Trabajo de Metasploit para Enumeración de Servicios

### Flujo General

1. **Lanzar Metasploit**: `msfconsole`
2. **Crear un workspace**: `workspace -a service_enum_target`
3. **Buscar módulos auxiliares**: `search auxiliary/scanner`
4. **Seleccionar módulo**: `use auxiliary/scanner/...`
5. **Establecer objetivo**: `set RHOSTS <target_ip>`
6. **Configurar opciones** (puertos, threads, wordlists): `set ...`
7. **Ejecutar**: `run` o `exploit`

---

## 1. Enumeración FTP

FTP es uno de los servicios más comunes de enumerar. Verificas **login anónimo**, **shares abiertos** y **credenciales débiles**.

### Paso 1: Escaneo de Puertos para Encontrar FTP
```bash
db_nmap -sS -sV -p 21 <target_ip>
```

### Paso 2: Verificar Acceso FTP Anónimo
```bash
msf6 > use auxiliary/scanner/ftp/anonymous
msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS <target_ip>
RHOSTS => <target_ip>
msf6 auxiliary(scanner/ftp/anonymous) > run
```

Si el login anónimo tiene éxito, has encontrado un share FTP abierto.

### Paso 3: Brute-force de Login FTP (Prueba de Credenciales)
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

O usa wordlists separadas de usuario y contraseña:
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

**Salidas clave**:
- `+` indica login exitoso.
- `-` indica login fallido.

---

## 2. Enumeración SMB

SMB (Server Message Block) se usa para compartir archivos en Windows y Linux (Samba). La enumeración se enfoca en **shares**, **usuarios** y **modo de seguridad**.

### Paso 1: Escaneo de Puertos para Encontrar SMB
```bash
db_nmap -sS -sV -p 139,445 <target_ip>
```

### Paso 2: Enumerar Shares SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(scanner/smb/smb_enumshares) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumshares) > run
```

Esto lista todas las carpetas compartidas accesibles en el objetivo.

### Paso 3: Enumerar Usuarios SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumusers) > run
```

Esto lista las cuentas de usuario en el servidor SMB.

### Paso 4: Modo de Seguridad SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_security_mode
msf6 auxiliary(scanner/smb/smb_security_mode) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_security_mode) > run
```

Esto revela la configuración de seguridad SMB (ej. NTLMv1 vs NTLMv2).

### Paso 5: Brute-force de Login SMB
```bash
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/smb/smb_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/smb/smb_login) > run
```

---

## 3. Enumeración de Servidor Web (Apache)

La enumeración de servidor web se enfoca en **directorios**, **encabezados**, **versiones** y **vulnerabilidades**.

### Paso 1: Escaneo de Puertos para Encontrar Web
```bash
db_nmap -sS -sV -p 80,443 <target_ip>
```

### Paso 2: Enumeración de Encabezados HTTP
```bash
msf6 > use auxiliary/scanner/http/http_header
msf6 auxiliary(scanner/http/http_header) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_header) > set RPORT 80
msf6 auxiliary(scanner/http/http_header) > run
```

Esto muestra los encabezados del servidor (ej. versión Apache, encabezados de seguridad).

### Paso 3: Brute-force de Directorios (Enumeración Web)
```bash
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(scanner/http/dir_scanner) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/dir_scanner) > set RPORT 80
msf6 auxiliary(scanner/http/dir_scanner) > set WORDLIST /usr/share/metasploit-framework/data/wordlists/dirbuster.txt
msf6 auxiliary(scanner/http/dir_scanner) > set THREADS 10
msf6 auxiliary(scanner/http/dir_scanner) > run
```

Esto descubre directorios ocultos como `/admin`, `/backup`, `/config`.

### Paso 4: Detección de Versión Apache
```bash
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(scanner/http/http_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_version) > run
```

---

## 4. Enumeración MySQL

La enumeración MySQL se enfoca en **detección de versión**, **lista de bases de datos** y **pruebas de autenticación**.

### Paso 1: Escaneo de Puertos para Encontrar MySQL
```bash
db_nmap -sS -sV -p 3306 <target_ip>
```

### Paso 2: Enumeración de Información MySQL
```bash
msf6 > use auxiliary/scanner/mysql/mysql_info
msf6 auxiliary(scanner/mysql/mysql_info) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_info) > run
```

Esto muestra la versión de MySQL y detalles de configuración.

### Paso 3: Brute-force de Login MySQL
```bash
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 auxiliary(scanner/mysql/mysql_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/mysql/mysql_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/mysql/mysql_login) > run
```

Credenciales por defecto comunes: `root:`, `root:root`, `root:password`.

---

## 5. Enumeración SSH

La enumeración SSH se enfoca en **detección de algoritmos**, **verificación de hostkey** y **pruebas de login**.

### Paso 1: Escaneo de Puertos para Encontrar SSH
```bash
db_nmap -sS -sV -p 22 <target_ip>
```

### Paso 2: Detección de Versión y Algoritmo SSH
```bash
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_version) > run
```

Esto muestra la versión de SSH (ej. SSH-2.0-OpenSSH_8.4).

### Paso 3: Enumeración de Hostkey SSH
```bash
msf6 > use auxiliary/scanner/ssh/ssh_hostkey
msf6 auxiliary(scanner/ssh/ssh_hostkey) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_hostkey) > run
```

Esto recupera la huella digital (fingerprint) del hostkey SSH.

### Paso 4: Brute-force de Login SSH (Prueba de Credenciales)
```bash
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > THREADS 10
msf6 auxiliary(scanner/ssh/ssh_login) > run
```

Esto prueba combinaciones de usuario/contraseña contra SSH. Los logins exitosos muestran `+`.

**Rutas de wordlists**:
- `/usr/share/metasploit-framework/data/wordlists/default_users.txt`
- `/usr/share/metasploit-framework/data/wordlists/default_passwords.txt`
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`

---

## 6. Enumeración SMTP (Bonus)

La enumeración SMTP se enfoca en **enumeración de usuarios** y **pruebas de comandos**.

### Paso 1: Escaneo de Puertos para Encontrar SMTP
```bash
db_nmap -sS -sV -p 25 <target_ip>
```

### Paso 2: Enumeración de Usuarios SMTP
```bash
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smtp/smtp_enum) > run
```

Esto lista usuarios de correo en el servidor SMTP usando comandos VRFY/EXPN.

---

## Ejemplo de Flujo Completo de Enumeración de Servicios

Aquí tienes un flujo completo desde escaneo hasta enumeración de servicios para un solo objetivo:

```bash
# 1. Lanzar Metasploit
msfconsole

# 2. Crear workspace
workspace -a service_enum_192.168.1.10

# 3. Escaneo inicial con almacenamiento en base de datos
db_nmap -sS -sV -p- -oX full_scan.xml 192.168.1.10

# 4. Importar resultados de escaneo
db_import full_scan.xml

# 5. Listar servicios descubiertos
services

# 6. Enumeración FTP
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.1.10
run

# 7. Enumeración SMB
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 192.168.1.10
run

use auxiliary/scanner/smb/smb_enumusers
set RHOSTS 192.168.1.10
run

# 8. Enumeración Web
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.1.10
set RPORT 80
set THREADS 10
run

# 9. Enumeración MySQL
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 192.168.1.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
run

# 10. Enumeración SSH
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.10
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
set THREADS 10
run

# 11. Exportar resultados
db_export -f xml enumeration_results.xml
```

---

## Resumen de Módulos Auxiliares de Escáner de Metasploit

| Servicio | Puerto | Módulo Auxiliar | Propósito |
|----------|--------|-----------------|-----------|
| **FTP** | 21 | `auxiliary/scanner/ftp/anonymous` | Verificar login anónimo |
| **FTP** | 21 | `auxiliary/scanner/ftp/ftp_login` | Brute-force de login |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumshares` | Enumerar shares |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumusers` | Enumerar usuarios |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_login` | Brute-force de login |
| **Web** | 80,443 | `auxiliary/scanner/http/dir_scanner` | Brute-force de directorios |
| **Web** | 80,443 | `auxiliary/scanner/http/http_header` | Enumeración de encabezados |
| **Web** | 80,443 | `auxiliary/scanner/http/http_version` | Detección de versión |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_info` | Información de servicio |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_login` | Brute-force de login |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_version` | Detección de versión |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_hostkey` | Enumeración de hostkey |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_login` | Brute-force de login |
| **SMTP** | 25 | `auxiliary/scanner/smtp/smtp_enum` | Enumeración de usuarios |

---

## Comandos Clave de Metasploit para Enumeración de Servicios

| Comando | Propósito |
|---------|-----------|
| `msfconsole` | Lanzar consola de Metasploit |
| `workspace -a <nombre>` | Crear nuevo workspace |
| `db_nmap <opciones> <target>` | Ejecutar Nmap con almacenamiento en BD |
| `db_import <archivo>` | Importar resultados de escaneo |
| `hosts` | Listar hosts descubiertos |
| `services` | Listar servicios descubiertos |
| `search auxiliary/scanner` | Buscar módulos de escáner |
| `use auxiliary/scanner/...` | Seleccionar módulo |
| `set RHOSTS <ip>` | Establecer IP del objetivo |
| `set RPORT <puerto>` | Establecer puerto del objetivo |
| `set USER_FILE <ruta>` | Establecer wordlist de usuarios |
| `set PASS_FILE <ruta>` | Establecer wordlist de contraseñas |
| `set THREADS <número>` | Establecer número de threads |
| `run` o `exploit` | Ejecutar módulo |
| `db_export -f xml <archivo>` | Exportar base de datos a XML |

---

## Rutas de Wordlists en Metasploit

Metasploit incluye wordlists integradas para pruebas de credenciales:

```bash
/usr/share/metasploit-framework/data/wordlists/default_users.txt
/usr/share/metasploit-framework/data/wordlists/default_passwords.txt
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
/usr/share/metasploit-framework/data/wordlists/dirbuster.txt
```

Usa estas con las opciones `USER_FILE`, `PASS_FILE` y `WORDLIST`.

---

## Puntos Clave

- La **enumeración de servicios** identifica y analiza servicios en ejecución (FTP, SMB, Web, MySQL, SSH, SMTP).
- Los **módulos auxiliares de escáner** de Metasploit están diseñados para reconocimiento sin explotación.
- Siempre comienza con **escaneo de puertos** (`db_nmap`) para identificar puertos abiertos y servicios.
- Usa **`set RHOSTS`** para especificar la IP del objetivo para cada módulo.
- **FTP**: Verifica login anónimo con `auxiliary/scanner/ftp/anonymous`.
- **SMB**: Enumera shares y usuarios con `smb_enumshares` y `smb_enumusers`.
- **Web (Apache)**: Usa `dir_scanner` para brute-force de directorios y `http_header` para información de versión.
- **MySQL**: Usa `mysql_info` para detalles de servicio y `mysql_login` para pruebas de credenciales.
- **SSH**: Usa `ssh_version`, `ssh_hostkey` y `ssh_login` para versión, hostkey y pruebas de credenciales.
- El **brute-force de login SSH** usa `auxiliary/scanner/ssh/ssh_login` con wordlists de usuario y contraseña.
- **SMTP**: Usa `smtp_enum` para enumeración de usuarios.
- Las wordlists están en `/usr/share/metasploit-framework/data/wordlists/`.
- Usa **workspaces** para mantener las evaluaciones organizadas.
