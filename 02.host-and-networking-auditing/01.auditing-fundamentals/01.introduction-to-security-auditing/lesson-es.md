# Introduction to Security Auditing

La **Auditoría de Seguridad** es un proceso sistemático de evaluación y verificación de las medidas y controles de seguridad dentro de una organización para garantizar que sean **efectivos**, **apropiados** y **conformes** con los estándares, políticas y regulaciones relevantes.

Esta lección cubre los **fundamentos de la auditoría de seguridad**, incluyendo su importancia, terminología clave, el proceso de auditoría, los tipos de auditorías y cómo se relaciona con el pentesting — todo conocimiento esencial para la certificación eJPT.

---

## Por Qué Importa la Auditoría de Seguridad

La auditoría de seguridad ayuda a las organizaciones a:

- **Identificar vulnerabilidades y debilidades** en sistemas, procesos e infraestructura antes de que lo hagan los atacantes.
- **Garantizar el cumplimiento** de estándares regulatorios (GDPR, HIPAA, PCI DSS, ISO 27001).
- **Mejorar la gestión de riesgos** priorizando amenazas según su impacto potencial.
- **Mejorar las políticas de seguridad** identificando brechas y áreas de mejora.
- **Apoyar los objetivos de negocio** protegiendo operaciones y generando confianza en los clientes.
- **Habilitar la mejora continua** manteniendo las medidas de seguridad actualizadas frente a amenazas emergentes.

A diferencia del pentesting (que se centra en explotar vulnerabilidades técnicas), la auditoría de seguridad comienza en el **nivel fundacional** — evaluando procesos organizacionales, el cumplimiento de políticas por parte de los empleados y los procedimientos operativos.

---

## Terminología Esencial

| Término | Definición |
|---------|-----------|
| **Política de Seguridad** | Documento formal que define los objetivos, directrices y procedimientos de seguridad de una organización para proteger sus activos de información |
| **Cumplimiento (Compliance)** | Adherencia a requisitos regulatorios, estándares del sector y políticas internas relacionadas con la seguridad y la protección de datos |
| **Vulnerabilidad** | Debilidad en un sistema o proceso que puede ser explotada para obtener acceso no autorizado o causar daño |
| **Control** | Salvaguarda o contramedida implementada para mitigar riesgos y proteger activos de información |
| **Evaluación de Riesgos** | Proceso de identificar, analizar y evaluar riesgos para los activos de información de una organización |
| **Pista de Auditoría** | Registro cronológico de eventos que evidencia las acciones realizadas dentro de un sistema |
| **Auditoría de Cumplimiento** | Examen de la adherencia de una organización a requisitos regulatorios y estándares del sector |
| **Control de Acceso** | Medidas que regulan quién puede acceder a sistemas o datos específicos y qué acciones puede realizar |
| **Informe de Auditoría** | Documento formal que presenta los hallazgos, conclusiones y recomendaciones resultantes de una auditoría de seguridad |

---

## Estándares de Cumplimiento que Debes Conocer

| Estándar | Nombre Completo | A Quién Aplica |
|----------|----------------|----------------|
| **GDPR** | General Data Protection Regulation | Cualquier organización que gestione datos de ciudadanos de la UE |
| **HIPAA** | Health Insurance Portability and Accountability Act | Organizaciones sanitarias (EE.UU.) |
| **PCI DSS** | Payment Card Industry Data Security Standard | Organizaciones que procesan pagos con tarjeta |
| **ISO 27001** | Information Security Management System | Gestión general de seguridad de la información |

El incumplimiento puede resultar en **graves sanciones legales y económicas**, especialmente tras una brecha de datos.

---

## El Proceso de Auditoría

### 1. Planificación y Preparación
- Definir el **alcance y los objetivos** de la auditoría.
- Recopilar documentación relevante: políticas, diagramas de red, informes de auditorías anteriores.
- Formar el equipo auditor y establecer un calendario.

### 2. Recopilación de Información
- Revisar las **políticas, procedimientos y estándares** de seguridad existentes.
- Realizar **entrevistas** con personal clave para identificar posibles brechas.
- Recopilar datos técnicos sobre configuraciones de sistemas y arquitectura de red.

### 3. Evaluación de Riesgos
- **Identificar activos críticos** y amenazas potenciales contra ellos.
- **Evaluar vulnerabilidades** en los sistemas y procesos existentes.
- **Asignar niveles de riesgo** basándose en la probabilidad e impacto potencial.

### 4. Ejecución de la Auditoría
- Realizar **pruebas técnicas**: escaneos de vulnerabilidades, revisiones de configuración, pruebas de penetración.
- **Verificar el cumplimiento** de regulaciones y estándares relevantes.
- **Evaluar controles** — valorar la efectividad de las medidas de seguridad existentes.

### 5. Análisis y Evaluación
- **Analizar los hallazgos** para identificar debilidades de seguridad y áreas de mejora.
- **Comparar con estándares** y mejores prácticas del sector.
- **Priorizar los problemas** por severidad e impacto potencial en el negocio.

### 6. Informe
- **Documentar hallazgos**: vulnerabilidades, incumplimientos, controles ineficaces.
- **Proporcionar recomendaciones accionables** para abordar cada problema identificado.
- **Presentar los resultados** a los interesados y discutir los hallazgos clave.

### 7. Remediación y Seguimiento
- **Desarrollar un plan de remediación** basado en los hallazgos de la auditoría.
- **Implementar cambios** y mejoras.
- **Programar auditorías de seguimiento** para verificar que la remediación fue efectiva.
- **Monitorizar continuamente** y actualizar las medidas de seguridad conforme evolucionan las amenazas.

---

## Tipos de Auditorías

| Tipo de Auditoría | Quién la Realiza | Enfoque | Relevancia para Pentesting |
|-------------------|-----------------|---------|---------------------------|
| **Interna** | Equipo interno de auditoría | Controles internos, adherencia a políticas | Línea base de autoevaluación; señala áreas que necesitan pruebas más profundas |
| **Externa** | Auditores externos independientes | Evaluación imparcial de seguridad, benchmarks de cumplimiento | Los hallazgos guían el alcance del pentesting |
| **Cumplimiento** | Interna o externa | Adherencia regulatoria (GDPR, HIPAA, PCI DSS) | Identifica brechas regulatorias para pruebas específicas |
| **Técnica** | Especialistas en seguridad | Infraestructura TI, hardware, software, configuraciones | Revela controles técnicos donde el pentesting puede descubrir vulns |
| **Red** | Equipo de seguridad de red | Routers, switches, firewalls, dispositivos de red | Descubre fallos de diseño de red para pruebas de explotación |
| **Aplicación** | Equipo AppSec | Calidad de código, validación de entradas, autenticación, manejo de datos | Señala fallos en apps (SQLi, XSS) para escenarios de pentesting |

---

## Auditoría de Seguridad vs. Prueba de Penetración

Son dos **tipos diferentes** de evaluaciones de seguridad. Entender la distinción es crítico para el eJPT.

| Aspecto | Auditoría de Seguridad | Prueba de Penetración |
|---------|----------------------|----------------------|
| **Propósito** | Evaluar la postura de seguridad general y el cumplimiento | Simular ataques reales y explotar vulnerabilidades |
| **Alcance** | Integral: políticas, procedimientos, controles, cumplimiento | Específico: sistemas, redes o aplicaciones definidas |
| **Metodología** | Revisión de documentación, entrevistas, evaluación técnica | Explotación activa usando herramientas y técnicas |
| **Resultado** | Brechas en políticas y controles; estado de cumplimiento | Informe detallado de vulnerabilidades con vectores de ataque |
| **Frecuencia** | Programación regular (anual/semestral) o según requisitos | Según necesidad: tras cambios en sistemas, en un calendario, o tras auditoría |

### Enfoque Secuencial (Más Común)

El flujo típico es:

```
Auditoría de Seguridad → Hallazgos → Prueba de Penetración → Remediación → Auditoría de Seguimiento
```

1. La organización realiza una **auditoría de seguridad** para evaluar la postura general y el cumplimiento.
2. Los hallazgos de la auditoría definen el **alcance y el enfoque** del pentesting.
3. El pentesting **valida y confirma** las vulnerabilidades técnicas encontradas en la auditoría.
4. Se aplica la remediación y una **auditoría de seguimiento** verifica las correcciones.

**Ventajas**:
- Visión integral desde perspectivas de política y técnica.
- Aborda brechas en controles tanto procedimentales como técnicos.
- Ayuda a priorizar los esfuerzos de remediación basándose en el riesgo real.

### Enfoque Combinado

Algunas organizaciones ejecutan ambas simultáneamente en una **evaluación de seguridad holística**.

**Ventajas**:
- Agiliza el proceso combinando evaluaciones de política, procedimiento y técnica.
- Ofrece una visión más completa de la postura de seguridad en un único compromiso.
- Más eficiente y rentable.

---

## Puntos Clave

- La **auditoría de seguridad** evalúa la postura de seguridad general de una organización — políticas, procedimientos, controles y cumplimiento.
- **No es lo mismo que un pentesting**: la auditoría se centra en procesos y cumplimiento; el pentesting en la explotación técnica.
- El **proceso de auditoría en 7 pasos**: Planificación → Recopilación de Información → Evaluación de Riesgos → Ejecución → Análisis → Informe → Remediación.
- **Estándares de cumplimiento** a conocer: GDPR, HIPAA, PCI DSS, ISO 27001.
- **Tipos de auditoría**: Interna, Externa, Cumplimiento, Técnica, Red, Aplicación.
- Las auditorías normalmente vienen **antes** del pentesting — los hallazgos de la auditoría definen el alcance del pentest.
- La auditoría de seguridad es un **proceso continuo y permanente** — no un evento puntual.
- Una postura de seguridad sólida construida mediante auditorías regulares **apoya tanto el cumplimiento como los objetivos de negocio**.
