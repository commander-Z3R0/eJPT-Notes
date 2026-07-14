# Windows File System Vulnerabilities

Esta lección cubre los **Alternate Data Streams (ADS)**, una función del sistema de archivos NTFS que los atacantes abusan para ocultar payloads maliciosos dentro de archivos legítimos — evadiendo la detección básica de antivirus.

> ⚠️ **Nota**: Todos los comandos de esta lección se ejecutan en el **sistema Windows objetivo** a través de una sesión de shell o símbolo del sistema. Se requieren privilegios de administrador para el paso del symlink.

---

## Alternate Data Streams (ADS)

### ¿Qué son los ADS?

**Alternate Data Streams (ADS)** es un atributo del **sistema de archivos NTFS** (usado por Windows). Fue diseñado originalmente para proporcionar compatibilidad con el **Apple HFS (Hierarchical File System)** al transferir archivos entre macOS y Windows.

Cada archivo creado en una unidad con formato NTFS contiene **dos streams**:

| Stream | Propósito | ¿Visible para el Usuario? |
|--------|-----------|--------------------------|
| **Data Stream** | Stream por defecto — contiene el contenido real del archivo | Sí |
| **Resource Stream** | Contiene los metadatos del archivo (autor, marcas de tiempo, etc.) | No |

El **Resource Stream** es la clave — puede almacenar datos arbitrarios, incluyendo código ejecutable, y permanece **completamente oculto** en los listados de directorio estándar y en el Explorador de Windows.

### Por Qué los Atacantes Usan ADS

ADS es una técnica de evasión potente porque:

- El archivo anfitrión parece **normal** — mismo nombre, mismo tamaño visible.
- El stream oculto **no aparece** en los listados `dir` ni en el Explorador.
- El **antivirus básico basado en firmas** y las **herramientas de escaneo estático** no inspeccionan los ADS.
- Los datos ocultos persisten mientras exista el archivo anfitrión.

---

## ADS en la Práctica

### Paso 1: Ocultar Datos Dentro de un Stream de Archivo

El siguiente comando crea un stream oculto llamado `secret.txt` dentro de `test.txt`. Aunque `test.txt` no tenga contenido visible, el stream secreto almacena datos de forma independiente:

```cmd
notepad test.txt:secret.txt
```

Cuando se abra el Bloc de notas, escribe el contenido oculto y guárdalo. El resultado:
- `test.txt` muestra **0 bytes** en el Explorador y en `dir` — parece vacío.
- El contenido se almacena de forma invisible en el stream `secret.txt` de `test.txt`.

### Paso 2: Ocultar un Payload Malicioso en un Archivo Legítimo

Copia tu payload en el resource stream de un archivo de log de apariencia inocente:

```cmd
type payload.exe > C:\Temp\windowslog.txt:payload.exe
```

Lo que ocurre:
- `windowslog.txt` aparece como un archivo de log normal y vacío (0 bytes).
- `payload.exe` está ahora oculto dentro de su resource stream.
- Puedes **eliminar el `payload.exe` original** — la copia persiste dentro de `windowslog.txt`.

```cmd
del payload.exe
```

### Paso 3: Crear un Symlink para Ejecutar el Payload Oculto

Para ejecutar el payload oculto sin extraerlo, crea un **enlace simbólico** que mapea un nombre de comando directamente al stream oculto. Esto debe hacerse con **privilegios de administrador**:

```cmd
cd C:\Windows\System32
mklink winsvclog.exe C:\Temp\windowslog.txt:payload.exe
```

Lo que hace:
- Crea un symlink llamado `winsvclog.exe` en `System32`.
- Cuando se escribe `winsvclog.exe` en cualquier símbolo del sistema, Windows ejecuta el `payload.exe` oculto dentro de `windowslog.txt`.
- El payload se ejecuta sin ningún archivo visible en disco — solo existe un archivo de log vacío y un symlink.

### Paso 4: Ejecutar el Payload Oculto

```cmd
winsvclog.exe
```

El payload se ejecuta silenciosamente desde dentro del stream ADS.

---

## Verificar ADS (Detección)

El comando `dir` estándar **no** revela los streams ADS. Para listar todos los streams de un archivo:

```cmd
dir /r C:\Temp\windowslog.txt
```

El flag `/r` revela los resource streams. Ejemplo de salida:

```
windowslog.txt          0 bytes
windowslog.txt:payload.exe:$DATA
```

También puedes usar la herramienta **Sysinternals Streams**:

```cmd
streams.exe C:\Temp\windowslog.txt
```

---

## Resumen del Ataque ADS

| Paso | Comando | Propósito |
|------|---------|-----------|
| **1. Ocultar datos** | `notepad file.txt:hidden.txt` | Almacenar datos de forma invisible en un stream |
| **2. Ocultar payload** | `type payload.exe > log.txt:payload.exe` | Copiar ejecutable en el resource stream del archivo |
| **3. Eliminar original** | `del payload.exe` | Eliminar evidencia visible |
| **4. Crear symlink** | `mklink winsvclog.exe C:\Temp\log.txt:payload.exe` | Mapear un comando al stream oculto |
| **5. Ejecutar** | `winsvclog.exe` | Ejecutar payload oculto sin archivo en disco |
| **6. Detectar** | `dir /r file.txt` | Revelar streams ocultos |

---

## Puntos Clave

- **ADS (Alternate Data Streams)** es una función NTFS que permite que los archivos contengan múltiples streams de datos — solo el stream por defecto es visible.
- Diseñado originalmente para la **compatibilidad con macOS HFS**, los ADS son abusados por los atacantes para ocultar payloads maliciosos dentro de archivos legítimos.
- Cada archivo NTFS tiene un **Data Stream** (contenido visible) y un **Resource Stream** (metadatos ocultos — donde los atacantes esconden payloads).
- Un archivo con un stream ADS oculto reporta **0 bytes** en el Explorador y en el `dir` estándar — parece completamente vacío.
- Usa `type payload.exe > file.txt:payload.exe` para copiar un ejecutable en el stream oculto de un archivo.
- Usa `mklink` en `System32` para crear un symlink que ejecute el payload oculto por nombre — requiere **privilegios de administrador**.
- Usa `dir /r` o **Sysinternals Streams** para detectar y revelar los streams ADS ocultos.
- Los ADS evaden el **AV básico basado en firmas** y los **escáneres estáticos** — pero las soluciones EDR modernas y los AV avanzados pueden detectarlo.
