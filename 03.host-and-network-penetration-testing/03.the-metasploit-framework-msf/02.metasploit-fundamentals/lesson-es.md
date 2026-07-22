# The Metasploit Framework

## Instalación y configuración

Antes de usar Metasploit, primero hay que instalarlo y asegurarse de que la base de datos funcione correctamente. Metasploit es distribuido por Rapid7 y puede instalarse en Windows y Linux, pero en este curso usamos Kali Linux porque Metasploit y sus dependencias ya vienen incluidos.

Metasploit utiliza una base de datos para guardar hosts, escaneos, servicios y otros datos de la evaluación. PostgreSQL es la base de datos principal, así que el servicio de PostgreSQL debe estar iniciado antes de inicializar la base de datos de Metasploit.

### Pasos de instalación

```bash
sudo apt update && sudo apt upgrade -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
msfdb init
msfconsole
```

Para verificar la conexión a la base de datos dentro de `msfconsole`, usa:

```bash
db_status
```

Si todo está bien configurado, Metasploit mostrará que PostgreSQL está conectado. La base de datos también permite importar resultados de herramientas como Nmap y Nessus.

## Fundamentos de MSFconsole

`msfconsole` es la interfaz principal de Metasploit, todo en uno. Será la herramienta que más usarás durante el curso porque te da acceso a toda la funcionalidad del framework desde una sola consola.

### Lo que necesitas saber

- Cómo buscar módulos.
- Cómo seleccionar módulos.
- Cómo configurar opciones y variables de los módulos.
- Cómo buscar payloads.
- Cómo administrar sesiones.
- Cómo usar funciones adicionales.
- Cómo guardar la configuración.

### Comandos útiles

Buscar un módulo:

```bash
search <keyword>
```

Seleccionar un módulo:

```bash
use <module_path>
```

Ver las opciones del módulo:

```bash
show options
```

Ver los payloads disponibles:

```bash
show payloads
```

Ejecutar un módulo:

```bash
run
```

o

```bash
exploit
```

### Variables de los módulos

Muchos módulos requieren valores como la IP del objetivo, puertos y direcciones de callback. Estos valores se configuran mediante variables dentro de `msfconsole`.

| Variable | Propósito |
|---|---|
| `LHOST` | Dirección IP del atacante. |
| `LPORT` | Puerto en la máquina atacante usado para recibir la conexión inversa. |
| `RHOST` | Dirección IP del objetivo. |
| `RHOSTS` | Varias IPs objetivo o un rango de red. |
| `RPORT` | Puerto del objetivo. |

Ejemplo:

```bash
set RHOSTS 10.10.10.10
set LHOST 10.10.14.2
set LPORT 4444
```

Puedes definir valores de forma local para un solo módulo o de forma global para toda la sesión.

## Crear y administrar workspaces

Los workspaces te ayudan a organizar hosts, escaneos y actividades durante una evaluación. Son útiles para separar datos por objetivo o por cliente, y así mantener el trabajo limpio y ordenado.

### Comandos de workspaces

Listar workspaces:

```bash
workspace
```

Crear un nuevo workspace:

```bash
workspace -a client1
```

Cambiar a un workspace:

```bash
workspace client1
```

Eliminar un workspace:

```bash
workspace -d client1
```

### Por qué son importantes

- Mantienen las evaluaciones separadas.
- Hacen más fácil seguir hosts y resultados de escaneo.
- Ayudan a organizar el trabajo en compromisos más largos.

## Idea clave

El primer paso en Metasploit es preparar el entorno: PostgreSQL debe estar ejecutándose, `msfdb` debe estar inicializado y `msfconsole` es donde gestionas módulos, variables, sesiones y workspaces. Una vez eso está listo, puedes usar Metasploit de forma eficiente durante el resto del curso.
