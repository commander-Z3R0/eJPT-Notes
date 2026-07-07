# Governance, Risk & Compliance (GRC)

**GRC** son las siglas de **Governance, Risk, and Compliance**. Es un marco integral utilizado por las organizaciones para gestionar y alinear sus prácticas de gobernanza, estrategias de gestión de riesgos y cumplimiento de requisitos regulatorios — ayudando a mantener la **transparencia**, la **responsabilidad** y la **resiliencia**.

Esta lección cubre los **tres pilares del GRC**, por qué es relevante para los pentesters y los **frameworks, estándares y directrices** clave que necesitas conocer para la certificación eJPT.

---

## Los Tres Pilares del GRC

| Pilar | Definición |
|-------|-----------|
| **Governance** | El marco de políticas, procedimientos y prácticas que garantiza que una organización alcance sus objetivos, gestione sus riesgos y cumpla con los requisitos legales. Incluye el desarrollo de políticas, la definición de roles y responsabilidades, y el establecimiento de mecanismos de rendición de cuentas. |
| **Risk** | El proceso de identificar, evaluar y mitigar los riesgos que podrían impactar negativamente en los activos y operaciones de una organización. |
| **Compliance** | Garantizar que una organización cumpla con las leyes, regulaciones y estándares del sector relevantes — tanto externos (GDPR, HIPAA, PCI DSS) como políticas internas de seguridad. Incluye la realización de revisiones periódicas para verificar el cumplimiento continuo. |

---

## Por Qué el GRC Importa para los Pentesters

El GRC es directamente relevante para los pentesters porque ayuda a:

- Realizar **evaluaciones más exhaustivas y relevantes** al entender el contexto de gobernanza de la organización.
- **Enmarcar los hallazgos** dentro del contexto de las políticas organizacionales, la gestión de riesgos y los requisitos de cumplimiento.
- Proporcionar **recomendaciones más estratégicas** que se alineen con el framework GRC de la organización.
- Ayudar a fortalecer la **postura de seguridad general** más allá de las vulnerabilidades técnicas.

Cuando entiendes el framework GRC de una empresa, tu informe de pentesting se vuelve mucho más accionable y alineado con lo que realmente les importa a los interesados.

---

## Frameworks vs. Estándares vs. Directrices

Antes de profundizar en los frameworks específicos, es importante entender la distinción:

| Tipo | Descripción | ¿Obligatorio? |
|------|-------------|--------------|
| **Framework** | Enfoque estructurado para implementar prácticas de seguridad; flexible y adaptable a diversas organizaciones e industrias | Generalmente No |
| **Estándar** | Requisitos y criterios específicos que deben cumplirse para lograr la conformidad | A menudo Sí (industrias reguladas) |
| **Directriz** | Prácticas recomendadas y consejos; no vinculantes | No |

---

## Frameworks, Estándares y Directrices Clave

### NIST CSF (Cybersecurity Framework)

Desarrollado por el **National Institute of Standards and Technology**, el NIST CSF proporciona directrices y mejores prácticas para ayudar a las organizaciones a gestionar y reducir los riesgos de ciberseguridad.

**Funciones Principales**:

| Función | Propósito |
|---------|-----------|
| **Identify** | Comprender activos, riesgos y vulnerabilidades |
| **Protect** | Implementar salvaguardas para garantizar la prestación de servicios |
| **Detect** | Identificar la ocurrencia de eventos de ciberseguridad |
| **Respond** | Actuar ante incidentes de ciberseguridad detectados |
| **Recover** | Restaurar capacidades tras un incidente de ciberseguridad |

---

### COBIT (Control Objectives for Information & Related Technologies)

Un framework para **desarrollar, implementar, monitorizar y mejorar** las prácticas de gobernanza y gestión de TI.

**Enfoques clave**:
- Alinear los objetivos de TI con los **objetivos de negocio**.
- Gestionar los **riesgos de TI** de manera efectiva.
- Garantizar el **cumplimiento** de las regulaciones.

---

### ISO/IEC 27001

Un **estándar internacional** para los Sistemas de Gestión de Seguridad de la Información (SGSI/ISMS). Establece las mejores prácticas para gestionar y proteger la información sensible.

**Enfoques clave**:
- Establecer, implementar, mantener y **mejorar continuamente** un ISMS.
- Aplicable a **cualquier organización**, independientemente de su tamaño o sector.
- La certificación demuestra un compromiso formal con la seguridad de la información.

---

### PCI DSS (Payment Card Industry Data Security Standard)

Un conjunto de estándares de seguridad diseñados para **proteger la información de tarjetas de pago** y garantizar el procesamiento seguro de transacciones con tarjeta de crédito.

**Enfoques clave**:
- Proteger los **datos del titular de la tarjeta**.
- Mantener una **red segura**.
- Implementar robustas medidas de **control de acceso**.

**Obligatorio para**: Cualquier organización que gestione transacciones con tarjeta de crédito.

---

### HIPAA (Health Insurance Portability and Accountability Act)

Una **ley estadounidense** que establece estándares para proteger la información sensible de los pacientes y garantizar la privacidad y seguridad de los datos de salud.

**Enfoques clave**:
- Proteger la **Información de Salud Protegida (PHI)**.
- Estándares de seguridad y privacidad para el manejo de datos de salud.

**Obligatorio para**: Proveedores de salud estadounidenses, planes de salud y cualquier entidad que gestione PHI.

---

### GDPR (General Data Protection Regulation)

Un **reglamento europeo** que rige la protección de datos y la privacidad de las personas dentro de la UE y el EEE (Espacio Económico Europeo).

**Enfoques clave**:
- **Principios de protección de datos** y derechos de los interesados.
- Obligaciones para los **controladores** y **procesadores** de datos.
- Consentimiento del usuario, derecho de acceso, derecho al olvido ("right to be forgotten").

**Obligatorio para**: Cualquier organización que procese datos personales de personas en la UE/EEE — independientemente de dónde esté ubicada la organización.

---

### CIS Controls (Center for Internet Security Controls)

Un conjunto de **mejores prácticas priorizadas y accionables** para ayudar a las organizaciones a mejorar su postura de ciberseguridad. Organizadas en Grupos de Implementación (IG1, IG2, IG3) según la madurez y recursos de la organización.

**Enfoques clave**:
- Mejoras de seguridad prácticas y paso a paso.
- Ampliamente utilizadas como **línea base para el hardening de seguridad**.

---

### NIST SP 800-53

Una publicación de NIST que proporciona un **catálogo exhaustivo de controles de seguridad y privacidad** para sistemas de información federales y organizaciones.

**Enfoques clave**:
- Controles de seguridad para **sistemas de información federales**.
- Cubre gestión de riesgos, control de acceso, respuesta a incidentes y más.

**Obligatorio para**: Agencias federales de EE.UU. y organizaciones que manejen datos federales.

---

## Comparativa de Frameworks y Estándares

| Framework/Estándar | Tipo | ¿Obligatorio? | Enfoque Principal | A Quién Aplica |
|-------------------|------|--------------|-------------------|----------------|
| **NIST CSF** | Framework | No | Identify, Protect, Detect, Respond, Recover | Cualquier organización |
| **COBIT** | Framework | No | Gobernanza de TI alineada con objetivos de negocio | Organizaciones orientadas a TI |
| **ISO/IEC 27001** | Estándar | No (certificación voluntaria) | Sistema de Gestión de Seguridad de la Información (ISMS) | Cualquier organización |
| **PCI DSS** | Estándar | Sí | Proteger datos de tarjetas de pago | Orgs que procesan pagos con tarjeta |
| **HIPAA** | Estándar (Ley) | Sí | Proteger información de salud del paciente | Entidades sanitarias de EE.UU. |
| **GDPR** | Estándar (Ley) | Sí | Privacidad y protección de datos | Orgs que manejan datos personales de la UE/EEE |
| **CIS Controls** | Directrices | No | Mejores prácticas de seguridad accionables | Cualquier organización |
| **NIST SP 800-53** | Estándar | Sí (federal) | Catálogo de controles de seguridad | Agencias federales de EE.UU. |

---

## Puntos Clave

- **GRC** son las siglas de Governance, Risk, and Compliance — un marco para alinear las prácticas de seguridad con los objetivos organizacionales y los requisitos regulatorios.
- **Governance** establece políticas y rendición de cuentas; **Risk** identifica y mitiga amenazas; **Compliance** garantiza la adherencia a leyes y estándares.
- Para los pentesters, entender el GRC ayuda a enmarcar los hallazgos estratégicamente y proporcionar recomendaciones que resuenen con los interesados.
- Los **frameworks** (NIST CSF, COBIT) son flexibles y adaptables — no obligatorios.
- Los **estándares** (PCI DSS, HIPAA, GDPR, ISO 27001) establecen requisitos específicos — a menudo legalmente obligatorios.
- Las funciones principales del **NIST CSF**: Identify → Protect → Detect → Respond → Recover.
- **PCI DSS** — obligatorio para organizaciones que procesan pagos con tarjeta.
- **HIPAA** — obligatorio para entidades sanitarias de EE.UU. que manejan PHI.
- **GDPR** — obligatorio para cualquier organización que gestione datos personales de la UE/EEE, independientemente de su ubicación.
- **ISO 27001** — estándar internacional para ISMS; voluntario pero ampliamente reconocido.
- **CIS Controls** — pasos prácticos y priorizados para el hardening de seguridad; excelente punto de partida para cualquier organización.
- **NIST SP 800-53** — obligatorio para agencias federales de EE.UU.; catálogo exhaustivo de controles de seguridad.
