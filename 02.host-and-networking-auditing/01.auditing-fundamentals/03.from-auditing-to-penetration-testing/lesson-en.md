# From Auditing to Penetration Testing

This lesson demonstrates how **security audits and penetration tests work together** in practice. Using a real-world scenario, we walk through developing a security policy, performing a system audit with Lynis, and validating remediations through a penetration test — showing how each phase informs and builds on the next.

---

## Practical Scenario: SecureTech Solutions

**Company**: SecureTech Solutions
**Specialization**: Securing IT infrastructure for various clients.

**Objective**: Develop a security policy for Linux servers, perform a risk assessment using the **NIST SP 800-53** framework, conduct a security audit, and validate remediations with a penetration test.

---

> **Note**: All commands in this lesson assume you are operating as **root** directly on the target Linux server. If running as a non-root user, prepend `sudo` to each command. In lab environments (INE/eLabs), root access is provided by default.

---

## The Three-Phase Workflow

```
Phase 1: Develop Security Policy → Phase 2: Audit with Lynis → Phase 3: Penetration Test
```

Each phase feeds directly into the next — the policy defines what to audit, the audit findings define what to remediate, and the penetration test validates that remediations actually work.

---

## Phase 1 — Develop a Security Policy

### Goal

Establish a **baseline security policy for Linux servers** aligned with NIST SP 800-53 guidelines, ensuring servers are configured and managed securely — protected from unauthorized access, vulnerabilities, and other security threats.

### Policy Structure (NIST SP 800-53 Aligned)

| Section | Purpose |
|---------|---------|
| **1. Purpose & Scope** | Define the goals of the policy and which systems/assets it covers |
| **2. Access Control** | Rules for who can access the server and under what conditions |
| **3. Audit & Accountability** | Logging requirements, audit trail management, and review procedures |
| **4. Configuration Management** | Baseline configurations, change control, and hardening standards |
| **5. Identification & Authentication** | Password policies, MFA requirements, and user account management |
| **6. System & Information Integrity** | Patch management, malware protection, and integrity monitoring |
| **7. Maintenance** | Scheduled maintenance procedures and authorized maintenance personnel |

> Full NIST SP 800-53 documentation: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final

### Why This Phase Matters

Without a defined security policy, there is **no baseline to audit against**. The policy establishes what "secure" means for this organization — everything in Phases 2 and 3 is measured against it.

---

## Phase 2 — Auditing with Lynis

### Goal

Perform a security audit on the Linux server using **Lynis**, identify vulnerabilities, and remediate them — such as updating software and enforcing password policies — to bring the system into compliance with the Phase 1 policy.

### What is Lynis?

**Lynis** is an open-source security auditing tool for Unix/Linux systems. It performs a **health scan** of the system to support:
- **System hardening**
- **Compliance testing**
- **Vulnerability identification**

> Lynis documentation: https://cisofy.com/lynis/

**Important**: Run all Lynis commands directly on the **Linux server being audited** — not from your Kali machine or VM.

### Installation

```bash
# Download Lynis
wget https://downloads.cisofy.com/lynis/lynis-3.1.4.tar.gz

# Decompress
gzip -d lynis-3.1.4.tar.gz

# Extract
tar -xf lynis-3.1.4.tar

# Enter directory and set permissions
cd lynis/
chmod +x lynis
```

### Running the Audit

```bash
./lynis audit system
```

Lynis will scan the system and produce a detailed report covering:
- **System hardening suggestions**
- **Detected vulnerabilities and misconfigurations**
- **Compliance status against security benchmarks**
- **Warnings and recommendations** prioritized by severity

### Audit Output Interpretation

After the scan, contextualize the results against the organization's security policy:

| Lynis Output | Action |
|-------------|--------|
| **Warnings** | High-priority issues — remediate immediately |
| **Suggestions** | Lower-priority improvements — schedule for remediation |
| **Hardening Index** | Overall score; lower = more hardening needed |

### Remediation Examples

Based on typical Lynis findings against a NIST SP 800-53 policy:

```bash
# Update all packages (System & Information Integrity)
apt update && apt upgrade -y

# Enforce password policies (Identification & Authentication)
# Edit /etc/login.defs to set:
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14

# Disable unused services (Configuration Management)
systemctl disable <unused_service>
systemctl stop <unused_service>

# Enable and configure auditd (Audit & Accountability)
apt install auditd -y
systemctl enable auditd
systemctl start auditd
```

### Why This Phase Matters

The audit establishes **what is actually wrong** on the system compared to the policy baseline. Remediations are applied here — but the audit alone cannot confirm they work. That's what Phase 3 is for.

---

## Phase 3 — Penetration Test

### Goal

**Validate the effectiveness of the remediations** from Phase 2 by actively testing the Linux server — confirming that vulnerabilities have been addressed and identifying any remaining or previously unknown weaknesses.

### What the Penetration Test Does

| Task | Purpose |
|------|---------|
| **Compare initial audit findings vs. pentest results** | Verify that remediated vulnerabilities are no longer exploitable |
| **Test for residual vulnerabilities** | Check if any issues remain after remediation |
| **Discover new vulnerabilities** | Identify weaknesses not caught by the Lynis audit |
| **Validate compliance** | Confirm the server now meets the security policy requirements |

### Penetration Test Workflow for This Scenario

```
Reconnaissance → Scanning → Enumeration → Exploitation Attempts → Post-Exploitation → Reporting
```

1. **Reconnaissance**: Gather information about the Linux server (OS version, open ports, running services).
2. **Scanning**: Use Nmap to identify open ports and service versions.

```bash
nmap -sV -sC -p- <target_ip>
```

3. **Enumeration**: Enumerate services for misconfigurations, weak credentials, and known CVEs.
4. **Exploitation Attempts**: Attempt to exploit vulnerabilities that Lynis flagged — confirming whether remediations held.
5. **Post-Exploitation**: If access is gained, assess what an attacker could do (privilege escalation, lateral movement, data access).
6. **Reporting**: Document all findings with severity ratings and remediation recommendations.

### The Pentest Report

The final report is the deliverable that ties all three phases together:

| Report Section | Content |
|----------------|---------|
| **Executive Summary** | High-level overview of findings for management |
| **Audit vs. Pentest Comparison** | Side-by-side of Lynis findings vs. pentest results |
| **Confirmed Remediations** | Vulnerabilities from Phase 2 that are now fixed |
| **Residual Vulnerabilities** | Issues that were not fully remediated |
| **New Findings** | Vulnerabilities discovered only during pentesting |
| **Severity Ratings** | Critical / High / Medium / Low per finding |
| **Recommendations** | Actionable steps to address each remaining issue |

### Why This Phase Matters

Remediation without validation is **incomplete security**. A penetration test provides active, adversarial confirmation that the fixes actually work — and surfaces anything that slipped through the audit phase.

---

## Full Workflow Summary

| Phase | Tool/Method | Output |
|-------|------------|--------|
| **1. Security Policy** | NIST SP 800-53 framework | Documented baseline policy for Linux servers |
| **2. Security Audit** | Lynis | List of vulnerabilities and misconfigurations; remediations applied |
| **3. Penetration Test** | Nmap, Metasploit, manual testing | Validation report confirming remediation effectiveness and new findings |

---

## Key Takeaways

- Security auditing and penetration testing are **complementary, not interchangeable** — each serves a distinct purpose.
- The **security policy** (Phase 1) is the foundation — without it, there is no baseline to audit or test against.
- **Lynis** is a powerful open-source tool for Linux system auditing and hardening — always run it directly on the target server, not from Kali.
- The `./lynis audit system` command performs a full health scan producing warnings, suggestions, and a hardening index.
- **Remediations** must be applied after the audit before the penetration test begins.
- The **penetration test** (Phase 3) validates that remediations actually work and discovers any remaining or new vulnerabilities.
- The **final report** compares audit findings to pentest results — clearly showing what was fixed, what remains, and what was newly discovered.
- Findings in the report are **rated by severity** (Critical / High / Medium / Low) to help prioritize remediation efforts.
- This three-phase workflow maps directly to real-world engagements and is the foundation of **compliance-driven penetration testing**.
