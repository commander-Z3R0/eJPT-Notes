# Introduction to Security Auditing

**Security Auditing** is a systematic process of evaluating and verifying the security measures and controls within an organization to ensure they are **effective**, **appropriate**, and **compliant** with relevant standards, policies, and regulations.

This lesson covers the **fundamentals of security auditing**, including its importance, key terminology, the auditing process, types of audits, and how auditing relates to penetration testing — all essential knowledge for the eJPT certification.

---

## Why Security Auditing Matters

Security auditing helps organizations:

- **Identify vulnerabilities and weaknesses** in systems, processes, and infrastructure before attackers do.
- **Ensure compliance** with regulatory standards (GDPR, HIPAA, PCI DSS, ISO 27001).
- **Enhance risk management** by prioritizing threats based on potential impact.
- **Improve security policies** by identifying gaps and areas for improvement.
- **Support business objectives** by protecting operations and building customer trust.
- **Enable continuous improvement** by keeping security measures up to date against evolving threats.

Unlike penetration testing (which focuses on exploiting technical vulnerabilities), security auditing starts at the **foundational level** — assessing organizational processes, employee adherence to policies, and operational procedures.

---

## Essential Terminology

| Term | Definition |
|------|-----------|
| **Security Policy** | Formal document defining an organization's security objectives, guidelines, and procedures to protect information assets |
| **Compliance** | Adherence to regulatory requirements, industry standards, and internal policies related to security and data protection |
| **Vulnerability** | A weakness in a system or process that can be exploited to gain unauthorized access or cause harm |
| **Control** | A safeguard or countermeasure implemented to mitigate risks and protect information assets |
| **Risk Assessment** | Process of identifying, analyzing, and evaluating risks to an organization's information assets |
| **Audit Trail** | A chronological record of events providing evidence of actions taken within a system |
| **Compliance Audit** | Examination of an organization's adherence to regulatory requirements and industry standards |
| **Access Control** | Measures that regulate who can access specific systems/data and what actions they can perform |
| **Audit Report** | Formal document presenting findings, conclusions, and recommendations from a security audit |

---

## Compliance Standards You Must Know

| Standard | Full Name | Who It Applies To |
|----------|-----------|------------------|
| **GDPR** | General Data Protection Regulation | Any org handling EU citizen data |
| **HIPAA** | Health Insurance Portability and Accountability Act | Healthcare organizations (US) |
| **PCI DSS** | Payment Card Industry Data Security Standard | Orgs processing card payments |
| **ISO 27001** | Information Security Management System | General information security management |

Non-compliance can result in **serious legal and financial penalties**, especially following data breaches.

---

## The Auditing Process

### 1. Planning & Preparation
- Define the **scope and objectives** of the audit.
- Gather relevant documentation: policies, network diagrams, previous audit reports.
- Assemble the audit team and set a timeline.

### 2. Information Gathering
- Review existing security **policies, procedures, and standards**.
- Conduct **interviews** with key personnel to identify potential gaps.
- Collect technical data on system configurations and network architecture.

### 3. Risk Assessment
- **Identify critical assets** and potential threats against them.
- **Evaluate vulnerabilities** in existing systems and processes.
- **Assign risk levels** based on likelihood and potential impact.

### 4. Audit Execution
- Perform **technical testing**: vulnerability scans, configuration reviews, penetration tests.
- **Verify compliance** with relevant regulations and standards.
- **Evaluate controls** — assess the effectiveness of existing security measures.

### 5. Analysis & Evaluation
- **Analyze findings** to identify security weaknesses and improvement areas.
- **Compare against standards** and industry best practices.
- **Prioritize issues** by severity and potential business impact.

### 6. Reporting
- **Document findings**: vulnerabilities, non-compliance issues, ineffective controls.
- **Provide actionable recommendations** to address each identified issue.
- **Present results** to stakeholders and discuss key findings.

### 7. Remediation & Follow-up
- **Develop a remediation plan** based on audit findings.
- **Implement changes** and improvements.
- **Schedule follow-up audits** to verify remediation was effective.
- **Monitor continuously** and update security measures as threats evolve.

---

## Types of Audits

| Audit Type | Who Performs It | Focus | Pentest Relevance |
|------------|----------------|-------|-------------------|
| **Internal** | Internal audit team | Internal controls, policy adherence | Self-assessment baseline; highlights areas needing deeper testing |
| **External** | Third-party auditors | Unbiased security evaluation, compliance benchmarks | Findings guide scope of pentest efforts |
| **Compliance** | Internal or external | Regulatory adherence (GDPR, HIPAA, PCI DSS) | Identifies regulatory gaps for targeted testing |
| **Technical** | Security specialists | IT infrastructure, hardware, software, configs | Reveals technical controls where pentesting can uncover vulns |
| **Network** | Network security team | Routers, switches, firewalls, network devices | Uncovers network design flaws for exploitation testing |
| **Application** | AppSec team | Code quality, input validation, authentication, data handling | Highlights app flaws (SQLi, XSS) for pentest scenarios |

---

## Security Audit vs. Penetration Test

These are two **different types** of security assessments. Understanding the distinction is critical for the eJPT.

| Aspect | Security Audit | Penetration Test |
|--------|---------------|-----------------|
| **Purpose** | Evaluate overall security posture and compliance | Simulate real-world attacks, exploit vulnerabilities |
| **Scope** | Comprehensive: policies, procedures, controls, compliance | Specific: defined systems, networks, or applications |
| **Methodology** | Documentation review, interviews, technical assessment | Active exploitation using tools and techniques |
| **Outcome** | Gaps in policies and controls; compliance status | Detailed vulnerability report with attack vectors |
| **Frequency** | Regular schedule (annually/biannually) or as required | As needed: after system changes, on a schedule, or post-audit |

### Sequential Approach (Most Common)

The typical flow is:

```
Security Audit → Findings → Penetration Test → Remediation → Follow-up Audit
```

1. Organization performs a **security audit** to evaluate overall posture and compliance.
2. Audit findings define the **scope and focus** of the penetration test.
3. Penetration test **validates and confirms** technical vulnerabilities found in the audit.
4. Remediation is applied and a **follow-up audit** verifies the fixes.

**Advantages**:
- Comprehensive view from both policy and technical perspectives.
- Addresses gaps in both procedural and technical controls.
- Helps prioritize remediation efforts based on real risk.

### Combined Approach

Some organizations run both simultaneously in a **holistic security assessment**.

**Advantages**:
- Streamlines the process by combining policy, procedural, and technical evaluations.
- Delivers a more complete picture of security posture in a single engagement.
- More efficient and cost-effective.

---

## Key Takeaways

- **Security auditing** evaluates an organization's overall security posture — policies, procedures, controls, and compliance.
- It is **not the same as a penetration test**: auditing focuses on processes and compliance; pentesting focuses on technical exploitation.
- The **7-step auditing process**: Planning → Information Gathering → Risk Assessment → Execution → Analysis → Reporting → Remediation.
- **Compliance standards** to know: GDPR, HIPAA, PCI DSS, ISO 27001.
- **Audit types**: Internal, External, Compliance, Technical, Network, Application.
- Audits typically come **before** penetration tests — audit findings define pentest scope.
- Security auditing is an **ongoing, continuous process** — not a one-time event.
- A strong security posture built through regular auditing **supports both compliance and business objectives**.
