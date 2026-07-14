# Linux Vulnerabilities

Linux es un sistema operativo libre y de código abierto compuesto por el kernel de Linux y el conjunto de herramientas GNU. Se utiliza ampliamente en muchos entornos, pero es más comúnmente desplegado como sistema operativo de servidores.

Debido a esto, los servidores Linux suelen ejecutar servicios y protocolos específicos. Estos servicios pueden exponer superficies de ataque que los atacantes pueden aprovechar para obtener acceso a un sistema objetivo.

## Por qué Linux es un objetivo

Los sistemas Linux son objetivo de los atacantes por varias razones:

1. **Dominio en servidores:** Una gran parte de los servidores web y sistemas de infraestructura utilizan Linux.
2. **Exposición de servicios:** Los servidores Linux suelen exponer servicios de red como SSH, FTP y servidores web.
3. **Mala configuración:** Permisos incorrectos, configuraciones débiles o servicios expuestos pueden introducir vulnerabilidades.
4. **Software desactualizado:** Paquetes sin actualizar o kernels antiguos pueden contener exploits conocidos.
5. **Naturaleza de código abierto:** Aunque es transparente, las vulnerabilidades también son estudiadas públicamente y pueden ser explotadas rápidamente.
6. **Debilidad en credenciales:** Contraseñas débiles o reutilización de claves SSH pueden permitir accesos no autorizados.

## Tipos de vulnerabilidades en Linux

Las vulnerabilidades en Linux pueden originarse en el sistema, los servicios instalados o problemas de configuración. Ejemplos comunes incluyen:

- **Escalada de privilegios:** Explotación del kernel o binarios SUID para obtener privilegios elevados.
- **Ejecución remota de código (RCE):** Vulnerabilidades en servicios que permiten ejecutar comandos de forma remota.
- **Permisos mal configurados:** Permisos incorrectos en archivos o directorios que 
