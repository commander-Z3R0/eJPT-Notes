#  MSF

## Overview

La detección de vulnerabilidades consiste en comprobar si un objetivo tiene vulnerabilidades conocidas y verificar si pueden ser explotadas. En esta fase, Metasploit se usa con módulos auxiliares y de explotación para identificar debilidades en servicios, sistemas operativos y aplicaciones web.

Esta fase es importante porque ayuda a reducir qué vulnerabilidades merece la pena probar durante la explotación. También permite organizar los resultados dentro de Metasploit para reutilizarlos más adelante.

## Entorno de laboratorio

Para esta sección, el laboratorio usa **Metasploitable3**, una máquina virtual intencionalmente vulnerable basada en Windows Server 2008. Fue creada por Rapid7 para demostrar cómo se puede usar Metasploit contra un sistema Windows.

## Gestión de workspaces

Los workspaces son útiles para separar los datos de escaneo por objetivo o por red.

### Comandos útiles

Listar workspaces:

```bash
workspace
```

Crear un workspace:

```bash
workspace -a lab1
```

Cambiar a un workspace:

```bash
workspace lab1
```

Eliminar un workspace:

```bash
workspace -d lab1
```

## Escaneo y análisis

Antes de escanear, asegúrate de que la base de datos esté activa y de haber seleccionado el workspace correcto.

### Verificar el estado de la base de datos

```bash
db_status
```

### Ver hosts

```bash
hosts
```

### Ver servicios

```bash
services
```

### Ver vulnerabilidades

```bash
vulns
```

## Comandos para escanear vulnerabilidades

Metasploit puede escanear un host único o toda una subred.

Escanear un host:

```bash
set RHOSTS <target_ip>
```

Escanear una subred:

```bash
set RHOSTS <target_network>/24
```

### Buscar módulos relevantes

Buscar por servicio o por nombre de vulnerabilidad:

```bash
search <keyword>
```

Ejemplos:

```bash
search smb
search http
search mysql
search ftp
```

### Comprobar un módulo

Algunos módulos de explotación permiten usar una comprobación segura para verificar si el objetivo probablemente es vulnerable:

```bash
check
```

### Ejecutar un módulo

```bash
run
```

o

```bash
exploit
```

## Importar resultados de escaneos

Metasploit también puede importar resultados de herramientas externas como Nmap y Nessus.

Importar un escaneo XML de Nmap:

```bash
db_import scan.xml
```

Después de importar, revisa los resultados:

```bash
hosts
services
vulns
```

## Searchsploit

`searchsploit` es útil para buscar exploits públicos después de identificar un servicio o versión vulnerable.

Buscar por nombre de software:

```bash
searchsploit <software_name>
```

Buscar por versión:

```bash
searchsploit <software_name> <version>
```

Copiar un exploit localmente si hace falta:

```bash
searchsploit -m <exploit_id>
```

## Metasploit Autopwn

`metasploit-autopwn` fue un método antiguo de automatización usado para relacionar servicios con posibles exploits. Ya no se considera parte del flujo estándar, pero sigue apareciendo en algunos materiales de formación como referencia histórica.

## Integración con Nessus

Los resultados de Nessus también se pueden importar en Metasploit si se tiene una exportación compatible. Esto ayuda a centralizar los datos de vulnerabilidades dentro de la base de datos de Metasploit.

## Flujo de trabajo útil

1. Seleccionar o crear un workspace.
2. Verificar la base de datos con `db_status`.
3. Importar resultados o escanear el objetivo directamente.
4. Revisar `hosts`, `services` y `vulns`.
5. Buscar módulos con `search`.
6. Confirmar la vulnerabilidad con `check`.
7. Usar `searchsploit` para comparar los hallazgos con exploits públicos.

## Idea clave

La detección de vulnerabilidades con Metasploit consiste en identificar posibles debilidades y organizar bien los resultados. Los workspaces, la base de datos, los escaneos importados y las comprobaciones de módulos facilitan el paso del descubrimiento a la explotación.
