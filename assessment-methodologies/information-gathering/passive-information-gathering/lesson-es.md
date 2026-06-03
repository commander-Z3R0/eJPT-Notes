# Reconocimiento

## Reconocimiento Pasivo

La recopilación pasiva de información consiste en obtener la mayor cantidad de datos posible sin interactuar activamente con el sistema objetivo.

### Qué recopilar
- Direcciones IP e información DNS.
- Nombres de dominio e información de propiedad.
- Direcciones de correo electrónico y perfiles en redes sociales.
- Tecnologías web utilizadas en los sitios objetivo.
- Subdominios.

### Comando Host
El comando `host` es una utilidad simple de consulta DNS en sistemas Unix/Linux. Puedes usarlo para encontrar la dirección IP asociada a un dominio, identificar servidores de correo, y más.

```bash
host -a <domain>
```

Si se devuelve más de una dirección IP, el objetivo puede estar usando un proxy o una CDN como Cloudflare.

### robots.txt
Una vez accedas a un sitio web, revisa el archivo `/robots.txt`. Indica a los rastreadores qué partes del sitio no deben indexarse y puede revelar rutas ocultas o sensibles.

### sitemap.xml
Revisa el archivo `/sitemap.xml` para ver todas las páginas indexadas del sitio. Puede ayudarte a entender la estructura más rápidamente.

### Tecnologías Web
Usa herramientas como:
- `BuiltWith`
- `Wappalyzer`
- `whatweb`

Estas herramientas ayudan a identificar las tecnologías utilizadas por un sitio objetivo.

```bash
whatweb <url>
```

### HTTrack
Usa `httrack` para descargar un sitio web completo e inspeccionar sus directorios y archivos localmente.

```bash
httrack <url>
```

### Enumeración Whois
`whois` es un protocolo utilizado para consultar bases de datos que almacenan información de registro sobre nombres de dominio, bloques de direcciones IP y sistemas autónomos.

```bash
whois <domain>
whois <ip>
```

También puedes usar:
- https://who.is/
- https://www.whois.com/

### Netcraft
Netcraft (https://sitereport.netcraft.com/) es útil para reconocimiento pasivo de sitios web. Puede revelar tecnologías, certificados e información del servidor.

### DNS Recon
`dnsrecon` es una herramienta de enumeración DNS que puede descubrir:
- Registros `A`, `AAAA`, `NS`, `MX`, `SOA`, `TXT` y `SPF`.
- Transferencias de zona.
- Búsquedas inversas.
- Subdominios.

```bash
dnsrecon -d <domain>
```

Otra herramienta útil es `dnsdumpster`, que proporciona un mapa visual del DNS.

### Wafw00f
Un WAF (Web Application Firewall) protege aplicaciones web filtrando tráfico malicioso. `wafw00f` identifica qué WAF utiliza un sitio web.

```bash
wafw00f <url>
```

### Sublist3r
`Sublist3r` enumera subdominios utilizando fuentes OSINT. No uses la opción de fuerza bruta si deseas mantenerte en reconocimiento pasivo.

```bash
sublist3r -d <domain>
```

### Google Dorks
Los Google dorks usan operadores de búsqueda específicos para encontrar información concreta.

Ejemplos:
- `site:<domain>`
- `filetype:pdf`
- `inurl:index.php`
- `intitle:index of`
- `cache:<url>`
- `related:<url>`

Recursos útiles:
- https://www.exploit-db.com/google-hacking-database
- https://archive.org/

### TheHarvester
`theHarvester` está diseñado para OSINT. Recopila correos electrónicos, nombres, subdominios, IPs, hosts y URLs de fuentes públicas.

```bash
theHarvester -d <domain> -b all
```

Herramientas y servicios útiles:
- https://phonebook.cz/
- https://search.censys.io/
- https://www.shodan.io/

### Bases de datos filtradas
Usa https://haveibeenpwned.com/ para verificar si una dirección de correo electrónico ha sido expuesta en una filtración de datos.
