# Nessus

## Overview

**Nessus** is a proprietary vulnerability scanner developed by Tenable. It is used to scan a target system for vulnerabilities and can provide useful details such as the CVE identifier, severity, and other findings related to each issue.

A Nessus scan is useful on its own, but it becomes even more valuable when you import the results into Metasploit Framework for analysis, validation, and possible exploitation.

## Lab Environment

For this section, the lab uses **Metasploitable3**, an intentionally vulnerable virtual machine based on Windows Server 2008. It was created by Rapid7 to demonstrate how Metasploit can be used against a Windows system.

## Installing Nessus

The free edition is **Nessus Essentials**, which allows scanning up to 16 IPs. It is enough for labs and practice environments.

### General Installation Flow

1. Download the Nessus installer from Tenable.
2. Install the package on your system.
3. Start the Nessus service.
4. Open the web interface on port `8834`.
5. Activate Nessus Essentials and create your user account.
6. Wait for the plugins to finish downloading and updating.

### Debian or Kali Example

```bash
sudo apt install ./Nessus-*.deb
sudo systemctl start nessusd
sudo systemctl enable nessusd
```

Then open the browser and go to:

```bash
https://127.0.0.1:8834
```

If you are accessing it from another machine:

```bash
https://<your_ip>:8834
```

## Creating a Scan

Once logged in, Nessus lets you create a new scan and choose a policy.

### Typical Workflow

1. Create a new scan.
2. Select the target type or scan template.
3. Enter the target IP or network range.
4. Launch the scan.
5. Review the results.

### Target Examples

Scan one host:

```bash
<target_ip>
```

Scan a network range:

```bash
<target_network>/24
```

## What Nessus Reports

A Nessus report can include:

- Open ports.
- Detected services.
- Service versions.
- Vulnerabilities.
- CVE references.
- Severity levels.
- Suggested remediation.

This makes it very useful for moving from discovery to exploitation.

## Importing Nessus Results Into Metasploit

Metasploit can import Nessus output so the scan data becomes part of the Metasploit database. This helps centralize your assessment data and makes it easier to review hosts, services, and vulnerabilities.

### Import Command

If you export the Nessus report in a compatible format, import it inside `msfconsole` with:

```bash
db_import scan.nessus
```

After importing, you can review the data with:

```bash
hosts
services
vulns
```

## Useful Metasploit Commands

Check database status:

```bash
db_status
```

View imported hosts:

```bash
hosts
```

View imported services:

```bash
services
```

View vulnerabilities:

```bash
vulns
```

Search for an exploit module related to a finding:

```bash
search <keyword>
```

Get detailed module information:

```bash
info <module_name>
```

Use a module:

```bash
use <module_path>
```

Check if a target is vulnerable:

```bash
check
```

Run the exploit:

```bash
exploit
```

## Example Workflow

A simple workflow would look like this:

1. Scan `<target_ip>` or `<target_network>/24` with Nessus.
2. Export the report.
3. Import it into Metasploit using `db_import`.
4. Review the imported `hosts`, `services`, and `vulns`.
5. Search for a matching exploit.
6. Confirm with `check`.
7. Exploit if appropriate.

## Why This Matters

Nessus automates vulnerability detection and provides structured results that can be reused in Metasploit. This saves time, helps you organize findings, and makes the transition from scanning to exploitation much smoother.

## Key Idea

Nessus is used to identify vulnerabilities quickly and clearly, while Metasploit is used to validate and exploit them. Importing Nessus results into Metasploit gives you a cleaner and more efficient workflow for penetration testing.
