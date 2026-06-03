# Reconocimient

## Reconocimiento Activo

La recopilación activa de información implica interactuar con el sistema objetivo, lo que requiere autorización.

### Transferencias de zona DNS
DNS (Domain Name System) resuelve nombres de dominio y hosts en direcciones IP. Los servidores DNS almacenan registros para la mayoría de los dominios en internet.

Tipos comunes de registros DNS:
- `A` — Nombre de host o dominio a una dirección IPv4.
- `AAAA` — Nombre de host o dominio a una dirección IPv6.
- `NS` — Servidor de nombres del dominio.
- `MX` — Dominio a un servidor de correo.
- `CNAME` — Alias de dominio.
- `TXT` — Registro de texto.
- `HINFO` — Información del host.
- `SOA` — Autoridad del dominio.
- `SRV` — Registros de servicio.
- `PTR` — Dirección IP a nombre de host.

La interrogación DNS es el proceso de enumerar registros DNS para un dominio específico. Las transferencias de zona pueden ser explotadas si los servidores DNS están mal configurados.

Puedes realizar una transferencia de zona con:

```bash
dig axfr @<dns-server> <domain>
dnsenum <domain>
```

`Fierce` es otra herramienta útil que a menudo se usa antes de Nmap.

### Nmap
Para encontrar tu propia dirección IP:

```bash
ip a
```

Para descubrir hosts:

```bash
nmap -sn <subnet>
netdiscover
```

Recuerda usar `sudo` cuando sea necesario.

## Opciones de Nmap

| Opción    | Uso                                                             |
| --------- | --------------------------------------------------------------- |
| -v        | Modo detallado, muestra lo que ocurre en tiempo real.           |
| -sV       | Detecta la versión de un servicio en un puerto.                 |
| -O        | Detecta el sistema operativo en ejecución.                      |
| -A        | Activa todas las detecciones (SO, versión, scripts, etc.).      |
| -T0 a -T5 | Ajusta la intensidad del escaneo de baja a alta velocidad.      |
| -sC       | Escanea con scripts NSE por defecto.                            |
| -F        | Escaneo más rápido con un conjunto reducido de puertos comunes. |
| -p        | Escanea todos o puertos específicos.                            |
| -Pn       | Desactiva el descubrimiento de hosts y solo escanea puertos.    |
| -sn       | Desactiva el escaneo de puertos y solo descubre hosts.          |
| -sU       | Escaneo de puertos UDP.                                         |
| -sS       | Escaneo TCP SYN.                                                |
