# Enumeration

## Overview

La enumeración es el proceso de recopilar información detallada sobre un servicio objetivo después de identificarlo. En esta fase, el objetivo es extraer datos útiles como versiones de servicios, usuarios, recursos compartidos, directorios, detalles de configuración y otra información que pueda ayudar durante la explotación o en una fase posterior de postexplotación.

Los módulos auxiliares son especialmente útiles para la enumeración porque permiten realizar escaneos, descubrimiento, fuzzing y comprobaciones específicas de cada servicio. Se pueden usar durante la fase de recopilación de información, después del acceso inicial e incluso después de hacer pivoting hacia otra subred.

## Escaneo de puertos con módulos auxiliares

Los módulos auxiliares pueden usarse para escanear puertos TCP y UDP, descubrir hosts y enumerar servicios en un objetivo o en una red interna.

### Comandos útiles

Escanear un host único:

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_ip>
run
```

Escanear una subred:

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_network>/24
run
```

Ejemplo de escaneo UDP:

```bash
use auxiliary/scanner/discovery/udp_probe
set RHOSTS <target_ip>
run
```

## Enumeración FTP

FTP usa el puerto TCP 21 y se utiliza para transferir archivos entre un cliente y un servidor. También se usa a veces para mover archivos hacia y desde el directorio de un servidor web.

La autenticación FTP normalmente requiere un nombre de usuario y una contraseña, pero algunos servidores permiten acceso anónimo si están mal configurados.

### Comandos útiles

Comprobar la versión de FTP:

```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target_ip>
run
```

Comprobar si permite acceso anónimo:

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run
```

Fuerza bruta de credenciales FTP:

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Enumeración SMB

SMB es un protocolo de compartición de archivos usado en redes locales. Normalmente funciona sobre el puerto TCP 445, aunque originalmente también utilizaba el puerto 139 sobre NetBIOS. SAMBA es la implementación de SMB en Linux.

Con los módulos auxiliares de SMB puedes enumerar la versión de SMB, los recursos compartidos, los usuarios y también realizar ataques de fuerza bruta.

### Comandos útiles

Comprobar la versión de SMB:

```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS <target_ip>
run
```

Enumerar recursos compartidos:

```bash
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run
```

Enumerar usuarios:

```bash
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run
```

Fuerza bruta de credenciales SMB:

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS <target_ip>
set USER_FILE users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Enumeración de servidores web

Un servidor web es software que sirve contenido de sitios web mediante HTTP. HTTP usa el puerto TCP 80, y entre los servidores web más conocidos están Apache, Nginx y Microsoft IIS.

Con módulos auxiliares, puedes identificar la versión del servidor web, inspeccionar las cabeceras HTTP y hacer fuerza bruta de directorios.

### Comandos útiles

Comprobar el título HTTP:

```bash
use auxiliary/scanner/http/title
set RHOSTS <target_ip>
run
```

Comprobar cabeceras HTTP:

```bash
use auxiliary/scanner/http/http_header
set RHOSTS <target_ip>
run
```

Fuerza bruta de directorios:

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set PATH /usr/share/wordlists/dirb/common.txt
run
```

## Enumeración MySQL

MySQL es un sistema de gestión de bases de datos relacional de código abierto basado en SQL. Se usa habitualmente para almacenar datos de aplicaciones y de sitios web, y normalmente funciona sobre el puerto TCP 3306.

Los módulos auxiliares pueden usarse para identificar la versión de MySQL, realizar fuerza bruta de contraseñas y ejecutar consultas SQL.

### Comandos útiles

Comprobar la versión de MySQL:

```bash
use auxiliary/scanner/mysql/mysql_version
set RHOSTS <target_ip>
run
```

Fuerza bruta de credenciales MySQL:

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS <target_ip>
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Enumeración SSH

SSH es un protocolo de administración remota que proporciona acceso cifrado y es el sucesor de Telnet. Normalmente usa el puerto TCP 22.

Los módulos auxiliares pueden usarse para identificar la versión de SSH y realizar fuerza bruta de credenciales.

### Comandos útiles

Comprobar la versión de SSH:

```bash
use auxiliary/scanner/ssh/ssh_version
set RHOSTS <target_ip>
run
```

Fuerza bruta de credenciales SSH:

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Enumeración SMTP

SMTP es el protocolo usado para enviar correo electrónico. Normalmente usa el puerto TCP 25, aunque también puede funcionar en los puertos 465 y 587.

Los módulos auxiliares pueden usarse para identificar la versión de SMTP y enumerar usuarios.

### Comandos útiles

Comprobar la versión de SMTP:

```bash
use auxiliary/scanner/smtp/smtp_version
set RHOSTS <target_ip>
run
```

Enumerar usuarios:

```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <target_ip>
run
```

## Por qué importa

La enumeración te ayuda a recopilar los detalles necesarios para decidir qué servicio atacar y cómo hacerlo. Los módulos auxiliares hacen este proceso más rápido porque automatizan los escaneos y las comprobaciones específicas de cada servicio.

## Idea clave

La enumeración consiste en convertir un servicio descubierto en información útil. Una vez identificas la versión del servicio, usuarios, recursos compartidos, cabeceras o credenciales, puedes pasar a la explotación con mucho más contexto.
