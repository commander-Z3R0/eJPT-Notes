# Evasión, Rendimiento de Escaneo y Salida

Nmap no solo sirve para escanear puertos. También ofrece opciones para **evadir filtros básicos**, **controlar la velocidad del escaneo** y **guardar resultados en distintos formatos** para analizarlos después.

---

## Evasión

La **evasión** se refiere a técnicas usadas para hacer que los escaneos sean más difíciles de detectar o bloquear por firewalls, IDS o IPS. En pentesting, el objetivo no es “hackear más rápido”, sino entender cómo reaccionan las defensas ante paquetes y patrones de escaneo poco comunes.

Nmap incluye varias opciones relacionadas con evasión que pueden ayudar cuando una red objetivo está filtrando o inspeccionando tráfico:

- **La fragmentación** divide los paquetes en fragmentos IP más pequeños.
- **Los decoys** ocultan la fuente real mezclando IPs falsas con el escaneo.
- **El tamaño de datos personalizado** cambia cómo se ve un paquete en la red.
- **La suplantación de origen** puede hacer que el tráfico parezca venir de otra dirección, aunque normalmente las respuestas no volverán correctamente.

Estas técnicas son útiles en pruebas controladas porque ayudan a evaluar si una red puede detectar o bloquear tráfico de escaneo no estándar.

---

## Fragmentación de paquetes

La **fragmentación de paquetes** divide los probes de Nmap en fragmentos IP más pequeños para que algunos filtros de paquetes o herramientas IDS tengan más dificultad inspeccionando todo el encabezado TCP/UDP de una sola vez.

### `-f`
El flag `-f` hace que Nmap fragmente los paquetes en **fragmentos pequeños**. Nmap divide el paquete en fragmentos de **8 bytes o menos después del encabezado IP**. Si usas `-f` dos veces, Nmap aumenta el tamaño del fragmento a **16 bytes**.

Ejemplo:
```bash
nmap -sS -f 192.168.1.10
nmap -sS -ff 192.168.1.10
```

### `--mtu`
La opción `--mtu` permite definir manualmente el tamaño del fragmento. El valor debe ser un **múltiplo de 8**, y no debes usarla junto con `-f`.

Ejemplo:
```bash
nmap -sS --mtu 24 192.168.1.10
```

### `--data-length`
La opción `--data-length` añade bytes aleatorios a los paquetes enviados. Esto hace que los probes sean menos mínimos y puede cambiar ligeramente cómo aparecen ante herramientas de monitoreo.

Ejemplo:
```bash
nmap -sS --data-length 16 192.168.1.10
```

### `-D`
La opción `-D` usa **decoys**. Nmap hace que el objetivo vea varias IPs de origen escaneando al mismo tiempo, así que es más difícil identificar cuál es el escáner real.

Ejemplo:
```bash
nmap -sS -D 10.10.10.10,10.10.10.11,ME 192.168.1.10
nmap -sS -D RND:5 192.168.1.10
```

### Cuándo usar estas opciones
Estas funciones son útiles cuando quieres probar si un firewall o IDS detecta tráfico de escaneo no estándar, o cuando necesitas entender cómo se comportan los controles defensivos ante probes poco comunes. También pueden hacer el escaneo más lento y menos fiable, así que no siempre son la mejor opción para un escaneo base normal.

---

## Rendimiento del escaneo

El **rendimiento del escaneo** es el equilibrio entre velocidad, precisión e impacto en la red. En pentesting, el rendimiento importa porque un escaneo demasiado lento desperdicia tiempo, pero uno demasiado agresivo puede activar alertas, perder paquetes o generar resultados inexactos.

Una buena estrategia de escaneo depende del entorno:

- En redes internas estables, los escaneos rápidos suelen ser aceptables.
- En redes ruidosas, frágiles o muy monitorizadas, los escaneos más lentos pueden ser más fiables.
- En evaluaciones reales, las decisiones de timing afectan tanto al **riesgo de detección** como a la **calidad del informe**.

Nmap ofrece varias opciones para ajustar este comportamiento.

### `-T`
La plantilla de tiempo `-T` controla cuán agresivo debe ser Nmap. Nmap define seis perfiles: **paranoid (`-T0`)**, **sneaky (`-T1`)**, **polite (`-T2`)**, **normal (`-T3`)**, **aggressive (`-T4`)** e **insane (`-T5`)**.

Ejemplos:
```bash
nmap -T3 192.168.1.10
nmap -T4 192.168.1.10
```

- `-T0` y `-T1` son más lentos y se usan cuando la discreción es más importante que la velocidad.
- `-T3` es el valor por defecto y suele ser un buen punto de partida.
- `-T4` y `-T5` son más rápidos, pero también aumentan el ruido y la posibilidad de perder respuestas.

### `--host-timeout`
`--host-timeout` establece el tiempo máximo que Nmap dedicará a un host antes de abandonarlo. Es útil cuando un objetivo es lento, está filtrado o hace que el escaneo se quede bloqueado.

Ejemplo:
```bash
nmap --host-timeout 2m 192.168.1.10
```

### `--min-rate`
`--min-rate` obliga a Nmap a enviar probes a una velocidad mínima de paquetes por segundo. Puede acelerar escaneos grandes, pero también aumentar la pérdida de paquetes o el riesgo de detección.

Ejemplo:
```bash
nmap --min-rate 100 192.168.1.10
```

### `--scan-delay`
`--scan-delay` inserta una pausa entre probes enviados al mismo host. Es útil cuando quieres reducir el ritmo del tráfico y minimizar la posibilidad de disparar alarmas.

Ejemplo:
```bash
nmap --scan-delay 200ms 192.168.1.10
```

### Visión práctica del rendimiento
En footprinting, el rendimiento no solo trata de velocidad. También trata de **control**: saber cuándo escanear rápido, cuándo bajar el ritmo y cuándo aceptar un tiempo mayor a cambio de resultados más limpios. Los controles de tiempo de Nmap te permiten adaptar el escaneo al entorno en lugar de usar la misma velocidad en todas partes.

---

## Formatos de salida

Nmap puede guardar resultados en distintos formatos para revisarlos, filtrarlos o importarlos después. Esto es especialmente útil en footprinting porque la salida del escaneo suele convertirse en la base de notas, informes y análisis posteriores.

### `-oN`
`-oN` guarda el escaneo en formato **normal de texto**. Se parece mucho a la salida que ves en terminal, pero escrita en un archivo.

Ejemplo:
```bash
nmap -sS -oN scan.txt 192.168.1.10
```

### `-oX`
`-oX` guarda el escaneo en formato **XML**. Es el formato más útil cuando quieres reutilizar los resultados en otras herramientas o automatizar su procesamiento.

Ejemplo:
```bash
nmap -sS -oX scan.xml 192.168.1.10
```

### `-oG`
`-oG` guarda el escaneo en formato **grepable**. Este formato antiguo está pensado para ser fácil de analizar con herramientas de shell y scripts simples.

Ejemplo:
```bash
nmap -sS -oG scan.gnmap 192.168.1.10
```

### `-oA`
`-oA` guarda la salida en **todos los formatos principales a la vez**: normal, XML y grepable. Es práctico cuando quieres que un solo escaneo produzca varios tipos de reporte.

Ejemplo:
```bash
nmap -sS -oA scan_results 192.168.1.10
```

### Por qué importa XML
XML es especialmente importante porque muchas herramientas pueden importarlo directamente. Conserva datos estructurados del escaneo como hosts, puertos, servicios y metadatos.

---

## Importar en Metasploit

En footprinting, puedes usar las funciones de base de datos de **Metasploit** para importar resultados XML de Nmap y organizar los hosts y servicios descubiertos para revisarlos después. Lo importante aquí es que esto sigue siendo parte de la fase de reconocimiento y reporte, no de explotación.

Flujo típico:
```bash
nmap -sS -sV -oX scan.xml 192.168.1.10
msfconsole
db_import scan.xml
hosts
services
```

Qué ocurre aquí:

- `nmap -oX` genera un reporte XML.
- `msfconsole` abre Metasploit.
- `db_import` carga el XML en la base de datos de Metasploit.
- `hosts` y `services` permiten revisar los datos de footprinting importados.

Esto es útil porque centraliza los resultados de reconocimiento y hace más fácil el análisis posterior, especialmente en evaluaciones grandes.

---

## Uso recomendado

Un flujo sencillo de footprinting podría verse así:

```bash
nmap -sS -T3 -oN scan.txt 192.168.1.10
nmap -sS -T4 --scan-delay 100ms -oX scan.xml 192.168.1.10
nmap -sS -f -D RND:5 -oG scan.gnmap 192.168.1.10
```

Usa configuraciones rápidas cuando la red sea estable, y opciones más lentas o cuidadosas cuando quieras reducir el ruido o probar controles defensivos.

---

## Puntos clave

- **Evasión** significa cambiar cómo se ven los probes para que firewalls e IDS tengan más dificultad detectándolos.
- `-f` fragmenta paquetes, `--mtu` define el tamaño del fragmento, `--data-length` añade payload aleatorio y `-D` usa decoys.
- **Rendimiento** trata del equilibrio entre velocidad, precisión y discreción durante el escaneo.
- `-T`, `--host-timeout`, `--min-rate` y `--scan-delay` son los principales controles de timing.
- `-oN`, `-oX` y `-oG` guardan resultados en distintos formatos para análisis y reporte.
- La salida XML de Nmap es especialmente útil porque puede importarse en `msfconsole` con `db_import` durante footprinting.
