# Governance, Risk & Compliance (GRC)

**GRC** stands for **Governance, Risk, and Compliance**. It is a comprehensive framework used by organizations to manage and align their governance practices, risk management strategies, and compliance with regulatory requirements — helping maintain **transparency**, **accountability**, and **resilience**.

This lesson covers the **three pillars of GRC**, why it matters for penetration testers, and the key **frameworks, standards, and guidelines** you need to know for the eJPT certification.

---

## The Three Pillars of GRC

| Pillar | Definition |
|--------|-----------|
| **Governance** | The framework of policies, procedures, and practices that ensures an organization achieves its objectives, manages risks, and complies with legal requirements. Includes policy development, defining roles/responsibilities, and establishing accountability mechanisms. |
| **Risk** | The process of identifying, assessing, and mitigating risks that could negatively impact an organization's assets and operations. |
| **Compliance** | Ensuring an organization adheres to relevant laws, regulations, and industry standards — both external (GDPR, HIPAA, PCI DSS) and internal security policies. Includes conducting regular reviews to verify ongoing compliance. |

---

## Why GRC Matters for Penetration Testers

GRC is directly relevant to pentesters because it helps to:

- Conduct **more thorough and relevant assessments** by understanding the organization's governance context.
- **Frame findings** within the context of organizational policies, risk management, and compliance requirements.
- Provide **more strategic recommendations** that align with the organization's GRC framework.
- Help strengthen the overall **security posture** beyond just technical vulnerabilities.

When you understand a company's GRC framework, your pentest report becomes far more actionable and aligned with what stakeholders actually care about.

---

## Frameworks vs. Standards vs. Guidelines

Before diving into the specific frameworks, it's important to understand the distinction:

| Type | Description | Mandatory? |
|------|-------------|-----------|
| **Framework** | Structured approach to implementing security practices; flexible and adaptable to various organizations and industries | Usually No |
| **Standard** | Specific requirements and criteria that must be met to achieve compliance | Often Yes (regulated industries) |
| **Guideline** | Recommended practices and advice; not binding | No |

---

## Key Frameworks, Standards & Guidelines

### NIST CSF (Cybersecurity Framework)

Developed by the **National Institute of Standards and Technology**, the NIST CSF provides guidelines and best practices to help organizations manage and reduce cybersecurity risks.

**Core Functions**:

| Function | Purpose |
|----------|---------|
| **Identify** | Understand assets, risks, and vulnerabilities |
| **Protect** | Implement safeguards to ensure service delivery |
| **Detect** | Identify the occurrence of cybersecurity events |
| **Respond** | Take action on detected cybersecurity incidents |
| **Recover** | Restore capabilities after a cybersecurity incident |

---

### COBIT (Control Objectives for Information & Related Technologies)

A framework for **developing, implementing, monitoring, and improving** IT governance and management practices.

**Key focuses**:
- Aligning IT goals with **business objectives**.
- Managing **IT risks** effectively.
- Ensuring **compliance** with regulations.

---

### ISO/IEC 27001

An **international standard** for Information Security Management Systems (ISMS). It outlines best practices for managing and protecting sensitive information.

**Key focuses**:
- Establishing, implementing, maintaining, and **continually improving** an ISMS.
- Applicable to **any organization**, regardless of size or sector.
- Certification demonstrates a formal commitment to information security.

---

### PCI DSS (Payment Card Industry Data Security Standard)

A set of security standards designed to **protect payment card information** and ensure the secure processing of credit card transactions.

**Key focuses**:
- Protecting **cardholder data**.
- Maintaining a **secure network**.
- Implementing robust **access control** measures.

**Mandatory for**: Any organization that handles credit card transactions.

---

### HIPAA (Health Insurance Portability and Accountability Act)

A **US law** that sets standards for protecting sensitive patient information and ensuring the privacy and security of health data.

**Key focuses**:
- Protecting **Protected Health Information (PHI)**.
- Security and privacy standards for health data handling.

**Mandatory for**: US healthcare providers, health plans, and any entity handling PHI.

---

### GDPR (General Data Protection Regulation)

A **European regulation** governing data protection and privacy for individuals within the EU and EEA (European Economic Area).

**Key focuses**:
- **Data protection principles** and rights of data subjects.
- Obligations for **data controllers** and **processors**.
- User consent, right to access, right to erasure ("right to be forgotten").

**Mandatory for**: Any organization processing personal data of EU/EEA individuals — regardless of where the organization is located.

---

### CIS Controls (Center for Internet Security Controls)

A set of **prioritized, actionable best practices** to help organizations improve their cybersecurity posture. Organized into Implementation Groups (IG1, IG2, IG3) based on organizational maturity and resources.

**Key focuses**:
- Practical, step-by-step security improvements.
- Widely used as a **baseline for security hardening**.

---

### NIST SP 800-53

A NIST publication providing a **comprehensive catalogue of security and privacy controls** for federal information systems and organizations.

**Key focuses**:
- Security controls for **federal information systems**.
- Covers risk management, access control, incident response, and more.

**Mandatory for**: US federal agencies and organizations handling federal data.

---

## Frameworks & Standards Comparison

| Framework/Standard | Type | Mandatory? | Key Focus | Who It Applies To |
|-------------------|------|-----------|-----------|------------------|
| **NIST CSF** | Framework | No | Identify, Protect, Detect, Respond, Recover | Any organization |
| **COBIT** | Framework | No | IT governance aligned with business objectives | IT-driven organizations |
| **ISO/IEC 27001** | Standard | No (certification voluntary) | Information Security Management System (ISMS) | Any organization |
| **PCI DSS** | Standard | Yes | Protect payment card data | Orgs processing card payments |
| **HIPAA** | Standard (Law) | Yes | Protect patient health information | US healthcare entities |
| **GDPR** | Standard (Law) | Yes | Data privacy and protection | Orgs handling EU/EEA personal data |
| **CIS Controls** | Guidelines | No | Actionable security best practices | Any organization |
| **NIST SP 800-53** | Standard | Yes (federal) | Security controls catalogue | US federal agencies |

---

## Key Takeaways

- **GRC** stands for Governance, Risk, and Compliance — a framework to align security practices with organizational objectives and regulatory requirements.
- **Governance** sets policies and accountability; **Risk** identifies and mitigates threats; **Compliance** ensures adherence to laws and standards.
- For pentesters, understanding GRC helps frame findings strategically and provide recommendations that resonate with stakeholders.
- **Frameworks** (NIST CSF, COBIT) are flexible and adaptable — not mandatory.
- **Standards** (PCI DSS, HIPAA, GDPR, ISO 27001) set specific requirements — often legally mandatory.
- **NIST CSF** core functions: Identify → Protect → Detect → Respond → Recover.
- **PCI DSS** — mandatory for payment card processing organizations.
- **HIPAA** — mandatory for US healthcare entities handling PHI.
- **GDPR** — mandatory for any org handling EU/EEA personal data, regardless of location.
- **ISO 27001** — international standard for ISMS; voluntary but widely recognized.
- **CIS Controls** — practical, prioritized steps for security hardening; great starting point for any org.
- **NIST SP 800-53** — mandatory for US federal agencies; comprehensive security controls catalogue.
