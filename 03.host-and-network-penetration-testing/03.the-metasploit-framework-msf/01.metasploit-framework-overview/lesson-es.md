# The Metasploit Framework

## Overview

El **Metasploit Framework (MSF)** es un framework de pruebas de penetración y explotación de código abierto usado por pentesters e investigadores de seguridad. Proporciona la estructura necesaria para automatizar muchas fases del ciclo de una prueba de penetración, desde el reconocimiento hasta la postexplotación.

También se utiliza para desarrollar y probar exploits, y cuenta con una gran base de datos de exploits públicos y probados. Su diseño modular facilita añadir nueva funcionalidad.

## Historia

- Desarrollado por **HD Moore** en **2003**.
- Escrito originalmente en **Perl**.
- Reescrito en **Ruby** en **2007**.
- Adquirido por **Rapid7** en **2009**.
- **Metasploit 5.0** fue lanzado en **2019**.
- **Metasploit 6.0** fue lanzado en **2020**.

## Ediciones

- **Metasploit Pro** — comercial.
- **Metasploit Express** — comercial.
- **Metasploit Framework** — edición comunitaria.

## Terminología esencial

| Término | Significado |
|---|---|
| **Interfaz** | Forma de interactuar con Metasploit. |
| **Módulo** | Fragmento de código que realiza una tarea concreta, como un exploit. |
| **Vulnerabilidad** | Debilidad o fallo en un sistema o red que puede ser explotado. |
| **Exploit** | Código o módulo usado para aprovechar una vulnerabilidad. |
| **Payload** | Código entregado al sistema objetivo por un exploit, normalmente para ejecutar comandos o dar acceso remoto. |
| **Listener** | Utilidad que espera una conexión entrante desde el objetivo. |

## Interfaces principales

### MSFconsole

La **Metasploit Framework Console (MSFconsole)** es una interfaz todo en uno fácil de usar que da acceso a toda la funcionalidad de Metasploit.

### MSFcli

La **Metasploit Framework Command Line Interface (MSFcli)** era una utilidad de línea de comandos usada para facilitar la creación de scripts de automatización con módulos de Metasploit. Permitía redirigir salida entre otras herramientas y Metasploit. Esta interfaz fue descontinuada en 2015, aunque una funcionalidad similar sigue disponible a través de MSFconsole.

### Metasploit Community Edition

**Metasploit Community Edition** es una interfaz gráfica web que simplifica el descubrimiento de red y la identificación de vulnerabilidades.

### Armitage

**Armitage** es una interfaz gráfica gratuita basada en Java para Metasploit que simplifica el descubrimiento de red, la explotación y la postexplotación.

## Arquitectura

Metasploit es modular, y cada módulo realiza una tarea específica. Las bibliotecas del framework permiten ejecutar módulos sin tener que escribir manualmente el código necesario para lanzarlos.

## Tipos de módulos

- **Exploit** — se usa para aprovechar una vulnerabilidad, normalmente junto con un payload.
- **Payload** — código entregado y ejecutado en el objetivo después de una explotación exitosa. Un ejemplo común es una reverse shell que conecta de vuelta con el atacante.
- **Encoder** — se usa para codificar payloads y ayudar a evitar la detección por antivirus.
- **NOPs** — se usan para mantener tamaños consistentes de payload y mejorar la estabilidad de ejecución.
- **Auxiliary** — se usa para tareas como escaneo de puertos y enumeración.
- **Post** — se usa después del acceso inicial para tareas como escalada de privilegios, obtención de credenciales y persistencia.

## Tipos de payloads

Metasploit soporta dos tipos principales de payloads:

- **Non-staged payloads**: se envían al objetivo como una sola unidad junto con el exploit.
- **Staged payloads**: se entregan en dos partes:
  - El **stager** establece la conexión inversa y descarga la segunda parte.
  - La **stage** es el componente del payload que se descarga y ejecuta.

## Meterpreter

**Meterpreter** es un payload avanzado y multifunción que se ejecuta en memoria en el sistema objetivo, lo que dificulta su detección. Se comunica a través de un socket de stager y proporciona un intérprete de comandos interactivo para tareas como ejecución de comandos, navegación por el sistema de archivos, keylogging y más.

## Estructura del sistema de archivos

El sistema de archivos de Metasploit está organizado en una estructura simple de directorios.

En Linux, los módulos de Metasploit se almacenan en:

```bash
/usr/share/metasploit-framework/modules
```

Los módulos definidos por el usuario se almacenan en:

```bash
~/.ms4/modules
```

## Flujo de trabajo de pentesting

Metasploit puede usarse para realizar y automatizar tareas en todo el ciclo de una prueba de penetración. Una forma útil de entender su papel es mediante la metodología PTES.

Las fases principales son:

- Recopilación de información y enumeración.
- Escaneo de vulnerabilidades.
- Explotación.
- Postexplotación.
- Escalada de privilegios.
- Mantenimiento de acceso persistente.

## Idea clave

Metasploit es un framework modular y flexible que ayuda a automatizar pruebas de penetración de principio a fin. Su fuerza está en su estructura, su biblioteca de módulos y su capacidad para combinar explotación y postexplotación en una sola plataforma.
