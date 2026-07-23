# Automating

## Overview

Los **resource scripts** de Metasploit son una función útil que permite automatizar tareas y comandos repetitivos dentro de `msfconsole`. Funcionan como scripts por lotes, ya que te permiten definir una secuencia de comandos y ejecutarlos automáticamente en orden.

Esto es especialmente útil cuando quieres repetir la misma configuración varias veces, como iniciar un handler, configurar opciones de payload o ejecutar varios comandos de Metasploit sin tener que escribirlos manualmente cada vez.

## Qué hacen los resource scripts

Los resource scripts te permiten:

- Automatizar comandos repetitivos de `msfconsole`.
- Ejecutar comandos de forma secuencial.
- Ahorrar tiempo durante pruebas y explotación.
- Estandarizar tareas comunes.
- Reducir errores manuales.

Se usan normalmente para:

- Configurar un `multi/handler`.
- Cargar y ejecutar payloads.
- Configurar opciones automáticamente.
- Ejecutar un flujo de trabajo predefinido de Metasploit.

## Idea básica

Un resource script es solo un archivo de texto que contiene comandos de `msfconsole`. Cuando se carga el script, Metasploit ejecuta cada comando en orden.

### Ejemplo de estructura

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

## Cómo funciona

En lugar de escribir los mismos comandos manualmente, los colocas en un archivo y luego cargas ese archivo dentro de `msfconsole`. Metasploit ejecuta los comandos uno tras otro.

### Ejemplo de archivo de recurso

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Cargar el script

Dentro de `msfconsole`, usa:

```bash
resource script.rc
```

Si el archivo es válido, Metasploit ejecutará automáticamente todos los comandos que contiene.

## Usos comunes

### Configuración de multi/handler

Un resource script es útil cuando necesitas un listener listo antes de lanzar un payload.

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Flujos de trabajo repetidos

También puedes usar resource scripts para automatizar otras tareas repetitivas, como:

- Seleccionar un workspace.
- Comprobar el estado de la base de datos.
- Cargar un plugin.
- Ejecutar un módulo.
- Guardar la salida.

## Por qué importa

Los resource scripts son valiosos porque hacen que Metasploit sea más rápido y eficiente de usar. Son especialmente útiles cuando necesitas repetir los mismos comandos durante laboratorios, demostraciones o evaluaciones reales.

## Idea clave

Los resource scripts de Metasploit automatizan `msfconsole` ejecutando una lista de comandos en secuencia. Son una forma sencilla pero potente de ahorrar tiempo y reducir la repetición.
