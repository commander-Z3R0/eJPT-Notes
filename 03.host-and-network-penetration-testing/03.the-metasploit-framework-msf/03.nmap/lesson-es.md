# Nmap

## Visión general

**Nmap** es un escáner de red libre y de código abierto usado para descubrir hosts en una red, analizar objetivos en busca de puertos abiertos e identificar los servicios y sistemas operativos que ejecutan esos objetivos.

También puede exportar los resultados del escaneo en un formato que se puede importar en Metasploit para la detección de vulnerabilidades y la explotación.

## Escaneo de puertos y enumeración

Nmap se usa comúnmente para:

- Descubrir hosts activos en una red.
- Escanear puertos abiertos.
- Identificar servicios que se ejecutan en los puertos abiertos.
- Detectar el sistema operativo del objetivo.

### Sintaxis básica de Nmap

```bash
nmap <target_ip>
```

### Ejemplos comunes

Escanear un host único:

```bash
nmap <target_ip>
```

Escanear un rango de red:

```bash
nmap <target_network>/24
```

Escanear puertos específicos:

```bash
nmap -p 22,80,443 <target_ip>
```

Escanear versiones de servicios:

```bash
nmap -sV <target_ip>
```

Detectar el sistema operativo:

```bash
nmap -O <target_ip>
```

## Importar resultados de Nmap en MSF

Metasploit puede importar resultados de escaneos de Nmap y usarlos para poblar su base de datos con hosts y servicios.

### Exportar resultados de Nmap

Para importar los resultados en Metasploit, la salida de Nmap debe guardarse en formato XML:

```bash
nmap -sV -O -oX scan.xml <target_ip>
```

o

```bash
nmap -sV -O -oX scan.xml <target_network>/24
```

### Importar en Metasploit

Dentro de `msfconsole`, usa:

```bash
db_import scan.xml
```

Después de importar, puedes ver los hosts y servicios descubiertos con:

```bash
hosts
services
```

## Por qué importa

Importar los resultados de Nmap en Metasploit ahorra tiempo y ayuda a organizar los datos de la evaluación. Permite reutilizar los resultados del escaneo para etapas posteriores de explotación y mantener la información de hosts y servicios en un solo lugar.

## Idea clave

Nmap es una de las primeras herramientas que se usan en una prueba de penetración porque ofrece una vista rápida de hosts activos, puertos abiertos, servicios y sistemas operativos. Su salida en XML puede importarse directamente en Metasploit para continuar la evaluación de forma eficiente.
