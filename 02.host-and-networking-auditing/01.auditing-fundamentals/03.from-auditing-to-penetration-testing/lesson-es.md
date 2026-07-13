# From Auditing to Penetration Testing

Esta lección demuestra cómo las **auditorías de seguridad y los tests de penetración trabajan juntos** en la práctica. Usando un escenario del mundo real, recorremos el desarrollo de una política de seguridad, la realización de una auditoría del sistema con Lynis y la validación de las remediaciones mediante un pentest — mostrando cómo cada fase informa y se construye sobre la anterior.

---

## Escenario Práctico: SecureTech Solutions

**Empresa**: SecureTech Solutions
**Especialización**: Securizar la infraestructura IT para diversos clientes.

**Objetivo**: Desarrollar una política de seguridad para servidores Linux, realizar una evaluación de riesgos usando el framework **NIST SP 800-53**, llevar a cabo una auditoría de seguridad y validar las remediaciones con un test de penetración.

---

> **Nota**: Todos los comandos de esta lección asumen que estás operando como **root** directamente en el servidor Linux objetivo. Si ejecutas como usuario no-root, antepón `sudo` a cada comando. En entornos de laboratorio (INE/eLabs), el acceso root está disponible por defecto.

---

## El Flujo de Trabajo en Tres Fases

```
Phase 1: Develop Security Policy → Phase 2: Audit with Lynis → Phase 3: Penetration Test
```

Cada fase alimenta directamente a la siguiente — la política define qué auditar, los hallazgos de la auditoría definen qué remediar, y el pentest valida que las remediaciones realmente funcionan.

---

## Phase 1 — Develop a Security Policy

### Objetivo

Establecer una **política de seguridad base para servidores Linux** alineada con las directrices del NIST SP 800-53, garantizando que los servidores estén configurados y gestionados de forma segura — protegidos frente a accesos no autorizados, vulnerabilidades y otras amenazas de seguridad.

### Estructura de la Política (Alineada con NIST SP 800-53)

| Sección | Propósito |
|---------|-----------|
| **1. Purpose & Scope** | Definir los objetivos de la política y qué sistemas/activos cubre |
| **2. Access Control** | Reglas sobre quién puede acceder al servidor y bajo qué condiciones |
| **3. Audit & Accountability** | Requisitos de logging, gestión de pistas de auditoría y procedimientos de revisión |
| **4. Configuration Management** | Configuraciones base, control de cambios y estándares de hardening |
| **5. Identification & Authentication** | Políticas de contraseñas, requisitos de MFA y gestión de cuentas de usuario |
| **6. System & Information Integrity** | Gestión de parches, protección contra malware y monitorización de integridad |
| **7. Maintenance** | Procedimientos de mantenimiento programado y personal de mantenimiento autorizado |

> Documentación completa del NIST SP 800-53: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final

### Por Qué Importa Esta Fase

Sin una política de seguridad definida, **no hay línea base contra la que auditar**. La política establece qué significa "seguro" para esta organización — todo lo que se hace en las Fases 2 y 3 se mide contra ella.

---

## Phase 2 — Auditing with Lynis

### Objetivo

Realizar una auditoría de seguridad en el servidor Linux usando **Lynis**, identificar vulnerabilidades y remediarlas — como actualizar software y reforzar las políticas de contraseñas — para poner el sistema en conformidad con la política de la Fase 1.

### ¿Qué es Lynis?

**Lynis** es una herramienta de auditoría de seguridad open-source para sistemas Unix/Linux. Realiza un **escaneo de salud** del sistema para apoyar:
- **Hardening del sistema**
- **Pruebas de cumplimiento**
- **Identificación de vulnerabilidades**

> Documentación de Lynis: https://cisofy.com/lynis/

**Importante**: Ejecuta todos los comandos de Lynis directamente en el **servidor Linux que se está auditando** — no desde tu máquina Kali ni desde una VM.

### Instalación

```bash
# Descargar Lynis
wget https://downloads.cisofy.com/lynis/lynis-3.1.4.tar.gz

# Descomprimir
gzip -d lynis-3.1.4.tar.gz

# Extraer
tar -xf lynis-3.1.4.tar

# Entrar al directorio y establecer permisos
cd lynis/
chmod +x lynis
```

### Ejecutar la Auditoría

```bash
./lynis audit system
```

Lynis escaneará el sistema y producirá un informe detallado que cubre:
- **Sugerencias de hardening del sistema**
- **Vulnerabilidades y malas configuraciones detectadas**
- **Estado de cumplimiento frente a benchmarks de seguridad**
- **Advertencias y recomendaciones** priorizadas por severidad

### Interpretación de la Salida de la Auditoría

Tras el escaneo, contextualiza los resultados frente a la política de seguridad de la organización:

| Salida de Lynis | Acción |
|----------------|--------|
| **Warnings** | Problemas de alta prioridad — remediar inmediatamente |
| **Suggestions** | Mejoras de menor prioridad — programar para remediación |
| **Hardening Index** | Puntuación general; cuanto más baja, más hardening es necesario |

### Ejemplos de Remediación

Basados en hallazgos típicos de Lynis frente a una política NIST SP 800-53:

```bash
# Actualizar todos los paquetes (System & Information Integrity)
apt update && apt upgrade -y

# Reforzar políticas de contraseñas (Identification & Authentication)
# Editar /etc/login.defs para establecer:
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14

# Deshabilitar servicios no utilizados (Configuration Management)
systemctl disable <servicio_no_usado>
systemctl stop <servicio_no_usado>

# Habilitar y configurar auditd (Audit & Accountability)
apt install auditd -y
systemctl enable auditd
systemctl start auditd
```

### Por Qué Importa Esta Fase

La auditoría establece **qué está realmente mal** en el sistema comparado con la línea base de la política. Las remediaciones se aplican aquí — pero la auditoría por sí sola no puede confirmar que funcionan. Para eso está la Fase 3.

---

## Phase 3 — Penetration Test

### Objetivo

**Validar la efectividad de las remediaciones** de la Fase 2 probando activamente el servidor Linux — confirmando que las vulnerabilidades han sido resueltas e identificando cualquier debilidad residual o previamente desconocida.

### Qué Hace el Penetration Test

| Tarea | Propósito |
|-------|-----------|
| **Comparar hallazgos iniciales de auditoría vs. resultados del pentest** | Verificar que las vulnerabilidades remediadas ya no son explotables |
| **Probar vulnerabilidades residuales** | Comprobar si algún problema persiste tras la remediación |
| **Descubrir nuevas vulnerabilidades** | Identificar debilidades no detectadas por la auditoría de Lynis |
| **Validar el cumplimiento** | Confirmar que el servidor cumple ahora con los requisitos de la política de seguridad |

### Flujo de Trabajo del Pentest para Este Escenario

```
Reconnaissance → Scanning → Enumeration → Exploitation Attempts → Post-Exploitation → Reporting
```

1. **Reconnaissance**: Recopilar información sobre el servidor Linux (versión del SO, puertos abiertos, servicios en ejecución).
2. **Scanning**: Usar Nmap para identificar puertos abiertos y versiones de servicios.

```bash
nmap -sV -sC -p- <target_ip>
```

3. **Enumeration**: Enumerar servicios en busca de malas configuraciones, credenciales débiles y CVEs conocidos.
4. **Exploitation Attempts**: Intentar explotar las vulnerabilidades que Lynis señaló — confirmando si las remediaciones aguantaron.
5. **Post-Exploitation**: Si se obtiene acceso, evaluar qué podría hacer un atacante (escalada de privilegios, movimiento lateral, acceso a datos).
6. **Reporting**: Documentar todos los hallazgos con clasificaciones de severidad y recomendaciones de remediación.

### El Informe del Pentest

El informe final es el entregable que une las tres fases:

| Sección del Informe | Contenido |
|--------------------|-----------|
| **Executive Summary** | Visión general de alto nivel de los hallazgos para la dirección |
| **Audit vs. Pentest Comparison** | Comparativa de hallazgos de Lynis vs. resultados del pentest |
| **Confirmed Remediations** | Vulnerabilidades de la Fase 2 que ahora están corregidas |
| **Residual Vulnerabilities** | Problemas que no fueron completamente remediados |
| **New Findings** | Vulnerabilidades descubiertas únicamente durante el pentesting |
| **Severity Ratings** | Critical / High / Medium / Low por hallazgo |
| **Recommendations** | Pasos accionables para abordar cada problema restante |

### Por Qué Importa Esta Fase

La remediación sin validación es **seguridad incompleta**. Un test de penetración proporciona confirmación activa y adversarial de que las correcciones realmente funcionan — y saca a la luz todo lo que se escapó durante la fase de auditoría.

---

## Resumen del Flujo de Trabajo Completo

| Fase | Herramienta/Método | Resultado |
|------|--------------------|-----------|
| **1. Security Policy** | Framework NIST SP 800-53 | Política base documentada para servidores Linux |
| **2. Security Audit** | Lynis | Lista de vulnerabilidades y malas configuraciones; remediaciones aplicadas |
| **3. Penetration Test** | Nmap, Metasploit, pruebas manuales | Informe de validación que confirma la efectividad de las remediaciones y nuevos hallazgos |

---

## Puntos Clave

- La auditoría de seguridad y el test de penetración son **complementarios, no intercambiables** — cada uno tiene un propósito distinto.
- La **política de seguridad** (Fase 1) es la base — sin ella, no hay línea base contra la que auditar o probar.
- **Lynis** es una potente herramienta open-source para auditoría y hardening de sistemas Linux — ejecútala siempre directamente en el servidor objetivo, no desde Kali.
- El comando `./lynis audit system` realiza un escaneo completo de salud produciendo warnings, suggestions y un hardening index.
- Las **remediaciones** deben aplicarse tras la auditoría antes de que comience el test de penetración.
- El **penetration test** (Fase 3) valida que las remediaciones realmente funcionan y descubre vulnerabilidades residuales o nuevas.
- El **informe final** compara los hallazgos de la auditoría con los resultados del pentest — mostrando claramente qué se arregló, qué queda pendiente y qué se descubrió de nuevo.
- Los hallazgos del informe se **clasifican por severidad** (Critical / High / Medium / Low) para ayudar a priorizar los esfuerzos de remediación.
- Este flujo de trabajo de tres fases se corresponde directamente con los compromisos del mundo real y es la base del **penetration testing orientado al cumplimiento normativo**.
