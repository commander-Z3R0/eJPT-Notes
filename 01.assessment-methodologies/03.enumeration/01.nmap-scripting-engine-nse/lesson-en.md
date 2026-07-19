# Nmap Scripting Engine (NSE)

The **Nmap Scripting Engine (NSE)** is one of the most powerful features in Nmap. It allows you to run **scripts** against target hosts to perform advanced **service enumeration**, **vulnerability detection**, and **information gathering** in an automated and customizable way.

## What is NSE?

NSE uses **Lua scripts** to extend Nmap's capabilities beyond simple port scanning. Each script is designed to perform a specific task, such as:

- Enumerating **users** (FTP, SMTP, SMB, SSH).
- Detecting **service versions** and configurations.
- Finding **vulnerabilities** (CVEs, misconfigurations).
- Gathering **web directory** structures.
- Extracting **DNS records** and network information.

NSE scripts are stored in `/usr/share/nmap/scripts/` on most Linux systems and are categorized by function (default, discovery, vuln, brute, auth, etc.).

---

## Running NSE Scripts

### Default Scripts (`-sC`)
The `-sC` flag runs the **default NSE script category**, which includes safe and commonly useful scripts for basic enumeration:

```bash
nmap -sS -sC <target_ip>
```

### Specific Scripts (`--script`)
You can run specific scripts or script categories by name:

```bash
nmap -sS --script http-enum <target_ip>
nmap -sS --script vuln <target_ip>
nmap -sS --script ftp-anon,ftp-bounce <target_ip>
nmap -sS --script smb-enum-shares,smb-enum-users <target_ip>
```

### Script Categories
NSE scripts are organized into categories:

| Category | Purpose |
|----------|---------|
| **default** | Safe scripts enabled with `-sC` |
| **discovery** | Service and host discovery |
| **vuln** | Vulnerability detection |
| **auth** | Authentication testing |
| **brute** | Brute-force attacks |
| **http** | Web server enumeration |
| **smb** | SMB enumeration |
| **ftp** | FTP enumeration |
| **ssh** | SSH enumeration |
| **smtp** | SMTP enumeration |

Example:
```bash
nmap -sS --script default <target_ip>
nmap -sS --script discovery <target_ip>
nmap -sS --script vuln <target_ip>
```

---

## Common NSE Scripts by Service

### FTP Enumeration
```bash
nmap -sS --script ftp-anon <target_ip>
nmap -sS --script ftp-bounce,ftp-libopie <target_ip>
```

### SMB Enumeration
```bash
nmap -sS --script smb-enum-shares,smb-enum-users <target_ip>
nmap -sS --script smb-security-mode,smb-os-discovery <target_ip>
```

### SSH Enumeration
```bash
nmap -sS --script ssh2-enum-algos,ssh-hostkey <target_ip>
```

### SMTP Enumeration
```bash
nmap -sS --script smtp-enum-users,smtp-commands <target_ip>
```

### Web Server Enumeration
```bash
nmap -sS --script http-enum,http-headers <target_ip>
nmap -sS --script http-sql-injection,xss <target_ip>
```

### MySQL Enumeration
```bash
nmap -sS --script mysql-enum,mysql-info <target_ip>
```

---

## Full Enumeration Methodology (IP → Services → Metasploit)

This methodology shows how to go from **raw IP enumeration** to a **organized Metasploit workspace** with imported scan data, all for **footprinting and reconnaissance** purposes only (no exploitation).

---

### Step 1: Initial IP Scan with NSE

Start with a basic scan to discover open ports and services:

```bash
nmap -sS -sV -O -p- -sC -oX initial_scan.xml <target_ip>
```

What this does:
- `-sS`: TCP SYN scan (stealthy, default)
- `-sV`: Detect service versions
- `-O`: Attempt OS detection
- `-p-`: Scan all 65535 ports
- `-sC`: Run default NSE scripts
- `-oX`: Export results to XML for Metasploit import

This gives you ports, services, versions, OS fingerprinting, and basic enumeration from NSE scripts.

---

### Step 2: Advanced NSE Enumeration by Service

Based on what you found, run targeted NSE scripts:

```bash
# FTP anonymous access check
nmap -sS --script ftp-anon -p 21 <target_ip>

# SMB shares and users
nmap -sS --script smb-enum-shares,smb-enum-users -p 139,445 <target_ip>

# Web directories and vulnerabilities
nmap -sS --script http-enum,http-vuln* -p 80,443 <target_ip>

# SMTP user enumeration
nmap -sS --script smtp-enum-users -p 25 <target_ip>

# SSH algorithms and hostkey
nmap -sS --script ssh2-enum-algos -p 22 <target_ip>
```

Save each result for later reference:
```bash
nmap -sS --script ftp-anon -p 21 -oN ftp_enum.txt <target_ip>
nmap -sS --script smb-enum-shares -p 445 -oN smb_enum.txt <target_ip>
```

---

### Step 3: Export Results to XML

You already used `-oX` in Step 1, but you can also export additional scans:

```bash
nmap -sS --script smb-enum-shares -oX smb_scan.xml <target_ip>
nmap -sS --script http-enum -oX web_scan.xml <target_ip>
```

XML format is critical because **Metasploit can import it** and store all hosts, ports, and service data in its database.

---

### Step 4: Launch Metasploit and Create a Workspace

Open Metasploit Console:
```bash
msfconsole
```

Create a new workspace for this target (to keep everything organized):
```bash
workspace -a enumeration_target_<target_ip>
```

Workspace options:
- `-a`: Add new workspace
- `-w`: Switch to existing workspace
- `-d`: Delete workspace
- `-l`: List workspaces

This ensures all your scan data, notes, and future modules stay separated from other assessments.

---

### Step 5: Import Nmap XML into Metasploit Database

Import your XML scan file:
```bash
db_import initial_scan.xml
```

This command parses the XML and stores **hosts, ports, services, and scripts** in the Metasploit database automatically.

You can import multiple files at once:
```bash
db_import *.xml
```

---

### Step 6: List Hosts and Services

Now verify what was imported:

```bash
# List all hosts in the database
hosts

# List all services in the database
services

# Filter by port
services -p 22

# Filter by protocol
services -p 445 -p 3389
```

You can also use `db_nmap` to run Nmap directly from Metasploit with automatic database storage:

```bash
db_nmap -sS -sV -oX db_scan.xml <target_ip>
db_nmap -sS -Pn -p 80,443 <target_ip>
```

The `-Pn` flag is useful when the target blocks ping but still has open ports.

---

### Step 7: List Vulnerabilities

Metasploit can show detected vulnerabilities from imported scans:

```bash
vulns
```

This will list any vulnerabilities found by NSE scripts (like `vuln` category scripts) that were stored in the database.

---

### Step 8: Use Metasploit Auxiliary Modules for Enumeration

Metasploit has **auxiliary modules** specifically designed for **reconnaissance and enumeration** (not exploitation). These are perfect for the footprinting phase:

```bash
# Search for auxiliary modules
search auxiliary

# FTP anonymous login check
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run

# SMB enum
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run

# SMB user enumeration
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run

# SSH version check
use auxiliary/scanner/ssh/ssh_version
set RHOSTS <target_ip>
run

# SMTP user enumeration
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <target_ip>
run

# HTTP directory enumeration
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set THREADS 10
run
```

Key auxiliary scanner categories:
- `auxiliary/scanner/ftp/`
- `auxiliary/scanner/smb/`
- `auxiliary/scanner/ssh/`
- `auxiliary/scanner/smtp/`
- `auxiliary/scanner/http/`
- `auxiliary/scanner/mysql/`

These modules perform **passive or active enumeration** without exploiting vulnerabilities, keeping you in the **footprinting/reconnaissance phase**.

---

### Step 9: Organize and Document

After enumeration, document your findings:

```bash
# Export workspace data
db_export -f xml workspace_export.xml

# List all discovered hosts
hosts -d

# List all services with details
services -v
```

Save your scan results, NSE output, and Metasploit database exports for your final report.

---

## NSE vs Metasploit Auxiliary Modules

| Feature | NSE Scripts | Metasploit Auxiliary |
|---------|-------------|---------------------|
| **Location** | Inside Nmap | Inside Metasploit |
| **Speed** | Very fast (single tool) | Slower (more options) |
| **Integration** | Manual export needed | Automatic database storage |
| **Scripting** | Lua-based | Ruby-based |
| **Best for** | Quick scans, quick enumeration | Organized assessments, database workflow |
| **Use case** | Initial footprinting | Detailed enumeration, reporting |

---

## Key Takeaways

- **NSE** uses Lua scripts to automate service enumeration, vulnerability detection, and information gathering.
- `-sC` runs default scripts; `--script` runs specific scripts or categories.
- Common scripts: `ftp-anon`, `smb-enum-shares`, `http-enum`, `smtp-enum-users`, `ssh2-enum-algos`.
- Export scan results with **`-oX`** to XML for Metasploit import.
- Create a **Metasploit workspace** to keep assessments organized.
- Use **`db_import`** to load Nmap XML into the Metasploit database.
- List hosts with **`hosts`**, services with **`services`**, and vulnerabilities with **`vulns`**.
- Use **`db_nmap`** to run Nmap directly from Metasploit with automatic database storage.
- Metasploit **auxiliary modules** (`auxiliary/scanner/*`) perform enumeration without exploitation.
