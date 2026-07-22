# Nessus

## Overview

**Nessus** es un escáner de vulnerabilidades propietario desarrollado por Tenable. Se utiliza para analizar un sistema objetivo en busca de vulnerabilidades y puede mostrar datos útiles como el identificador CVE, la gravedad y otra información relacionada con cada hallazgo.

Un escaneo con Nessus es útil por sí solo, pero se vuelve mucho más valioso cuando importas los resultados en Metasploit Framework para analizarlos, validarlos y, si procede, explotarlos.

## Entorno de laboratorio

Para esta parte, el laboratorio usa **Metasploitable3**, una máquina virtual intencionalmente vulnerable basada en Windows Server 2008. Fue creada por Rapid7 para demostrar cómo se puede usar Metasploit contra un sistema Windows.

## Instalación de Nessus

La edición gratuita es **Nessus Essentials**, que permite escanear hasta 16 IPs. Es suficiente para laboratorios y prácticas.

### Flujo general de instalación

1. Descargar el instalador de Nessus desde Tenable.
2. Instalar el paquete en el sistema.
3. Iniciar el servicio de Nessus.
4. Abrir la interfaz web en el puerto `8834`.
5. Activar Nessus Essentials y crear una cuenta de usuario.
6. Esperar a que terminen de descargarse y actualizarse los plugins.

### Ejemplo en Debian o Kali

```bash
sudo apt install ./Nessus-*.deb
sudo systemctl start nessusd
sudo systemctl enable nessusd
```

Después abre el navegador y entra en:

```bash
https://127.0.0.1:8834
```

Si accedes desde otra máquina:

```bash
https://<your_ip>:8834
```

## Crear un escaneo

Una vez dentro, Nessus permite crear un nuevo escaneo y elegir una política.

### Flujo de trabajo típico

1. Crear un nuevo escaneo.
2. Seleccionar el tipo o plantilla de escaneo.
3. Introducir la IP objetivo o el rango de red.
4. Lanzar el escaneo.
5. Analizar los resultados.

### Ejemplos de objetivo

Escanear un host:

```bash
<target_ip>
```

Escanear un rango de red:

```bash
<target_network>/24
```

## Qué reporta Nessus

Un informe de Nessus puede incluir:

- Puertos abiertos.
- Servicios detectados.
- Versiones de servicios.
- Vulnerabilidades.
- Referencias CVE.
- Niveles de gravedad.
- Recomendaciones de remediación.

Esto lo hace muy útil para pasar del reconocimiento a la explotación.

## Importar resultados de Nessus en Metasploit

Metasploit puede importar la salida de Nessus para integrar los datos del escaneo en su base de datos. Esto ayuda a centralizar la información de la evaluación y a revisar más fácilmente hosts, servicios y vulnerabilidades.

### Comando de importación

Si exportas el informe de Nessus en un formato compatible, impórtalo dentro de `msfconsole` con:

```bash
db_import scan.nessus
```

Después de importar, puedes revisar los datos con:

```bash
hosts
services
vulns
```

## Comandos útiles en Metasploit

Comprobar el estado de la base de datos:

```bash
db_status
```

Ver los hosts importados:

```bash
hosts
```

Ver los servicios importados:

```bash
services
```

Ver las vulnerabilidades:

```bash
vulns
```

Buscar un módulo de explotación relacionado con una falla:

```bash
search <keyword>
```

Obtener información detallada de un módulo:

```bash
info <module_name>
```

Usar un módulo:

```bash
use <module_path>
```

Comprobar si un objetivo es vulnerable:

```bash
check
```

Ejecutar la explotación:

```bash
exploit
```

## Ejemplo de flujo de trabajo

Un flujo simple sería así:

1. Escanear `<target_ip>` o `<target_network>/24` con Nessus.
2. Exportar el informe.
3. Importarlo en Metasploit con `db_import`.
4. Revisar `hosts`, `services` y `vulns`.
5. Buscar un exploit correspondiente.
6. Confirmar con `check`.
7. Explotar si es apropiado.

## Por qué es importante

Nessus automatiza la detección de vulnerabilidades y proporciona resultados estructurados que pueden reutilizarse en Metasploit. Esto ahorra tiempo, ayuda a organizar los resultados y hace mucho más fluido el paso entre el escaneo y la explotación.

## Idea clave

Nessus sirve para identificar vulnerabilidades de forma rápida y clara, mientras que Metasploit sirve para validarlas y explotarlas. Importar los resultados de Nessus en Metasploit da un flujo de trabajo más limpio y eficiente para una prueba de intrusión.
