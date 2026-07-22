# Web Apps

## Overview

**WMAP** es un escáner de vulnerabilidades para aplicaciones web integrado en Metasploit. Sirve para automatizar la enumeración de servidores web y escanear aplicaciones web en busca de vulnerabilidades desde `msfconsole`.

WMAP es útil porque mantiene todo el flujo de trabajo dentro de Metasploit, lo que permite guardar sitios, rutas y vulnerabilidades en la base de datos del framework.

## Entorno de laboratorio

Para practicar, el curso usa un objetivo intencionalmente vulnerable como **Metasploitable3**. En el laboratorio, normalmente escanearás un servicio web en `<target_ip>` o una aplicación web accesible en `<target_network>/24`.

## Preparación

Antes de usar WMAP, asegúrate de que Metasploit y la base de datos estén funcionando.

```bash
sudo systemctl start postgresql
msfdb init
msfconsole
```

Comprueba el estado de la base de datos:

```bash
db_status
```

Crea o cambia a un workspace:

```bash
workspace -a webapp
workspace webapp
```

## Cargar WMAP

Dentro de `msfconsole`, carga el plugin con:

```bash
load wmap
```

Si se carga correctamente, los comandos de WMAP quedarán disponibles en la consola.

## Añadir un sitio

Antes de escanear, añade la aplicación web objetivo como sitio.

Añadir un objetivo único:

```bash
wmap_sites -a <target_ip>
```

O añadir un objetivo mediante URL:

```bash
wmap_targets -t http://<target_ip>
```

Para una aplicación web alojada en una red, usa:

```bash
wmap_targets -t http://<target_network>/24
```

Listar los sitios configurados:

```bash
wmap_sites -l
```

Listar los objetivos configurados:

```bash
wmap_targets -l
```

## Ejecutar escaneos con WMAP

WMAP puede mapear el sitio y luego ejecutar los escáneres habilitados contra él.

Listar los módulos de escaneo disponibles:

```bash
wmap_run -t
```

Ejecutar todos los módulos habilitados:

```bash
wmap_run -e
```

Dependiendo del tamaño de la aplicación, esto puede tardar un poco.

## Revisar resultados

Cuando termine el escaneo, revisa las vulnerabilidades y rutas descubiertas.

```bash
wmap_vulns -l
```

También puedes revisar los datos generales de la base de Metasploit con:

```bash
hosts
services
vulns
```

## Flujo de trabajo útil

1. Iniciar PostgreSQL y Metasploit.
2. Comprobar `db_status`.
3. Crear un workspace.
4. Cargar WMAP con `load wmap`.
5. Añadir el sitio con `wmap_sites -a <target_ip>`.
6. Definir el objetivo con `wmap_targets -t http://<target_ip>`.
7. Listar objetivos y sitios para confirmar.
8. Ejecutar `wmap_run -t` y luego `wmap_run -e`.
9. Revisar los problemas descubiertos con `wmap_vulns -l`.
10. Consultar `hosts`, `services` y `vulns` en la base de datos.

## Por qué importa

WMAP es útil porque automatiza el reconocimiento y el escaneo de vulnerabilidades web desde Metasploit. Esto facilita conectar el descubrimiento con la explotación posterior, manteniendo todo organizado en un solo lugar.

## Idea clave

WMAP es el escáner de aplicaciones web integrado en Metasploit. Ayuda a mapear un sitio, escanearlo en busca de vulnerabilidades y guardar los resultados en la base de datos de Metasploit para usarlos después.
