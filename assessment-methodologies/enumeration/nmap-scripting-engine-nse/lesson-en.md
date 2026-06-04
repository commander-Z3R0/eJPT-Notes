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
nmap -sS -sC 192.168.1.10
```

### Specific Scripts (`--script`)
You can run specific scripts or script categories by name:

```bash
nmap -sS --script http-enum 192.168.1.10
nmap -sS --script vuln 192.168.1.10
nmap -sS --script ftp-anon,ftp-bounce 192.168.1.10
nmap -sS --script smb-enum-shares,smb-enum-users 192.168.1.10
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
nmap -sS --script default 192.168.1.10
nmap -sS --script discovery 192.168.1.10
nmap -sS --script vuln 192.168.1.10
```

---

## Common NSE Scripts by Service

### FTP Enumeration
```bash
nmap -sS --script ftp-anon 192.168.1.10
nmap -sS --script ftp-bounce,ftp-libopie 192.168.1.10
```

### SMB Enumeration
```bash
nmap -sS --script smb-enum-shares,smb-enum-users 192.168.1.10
nmap -sS --script smb-security-mode,smb-os-discovery 192.168.1.10
```

### SSH Enumeration
```bash
nmap -sS --script ssh2-enum-algos,ssh-hostkey 192.168.1.10
```

### SMTP Enumeration
```bash
nmap -sS --script smtp-enum-users,smtp-commands 192.168.1.10
```

### Web Server Enumeration
```bash
nmap -sS --script http-enum,http-headers 192.168.1.10
nmap -sS --script http-sql-injection,xss 192.168.1.10
```

### MySQL Enumeration
```bash
nmap -sS --script mysql-enum,mysql-info 192.168.1.10
```

---

## Full Enumeration Methodology (IP → Services → Metasploit)

This methodology shows how to go from **raw IP enumeration** to a **organized Metasploit workspace** with imported scan data, all for **footprinting and reconnaissance** purposes only (no exploitation).

---

### Step 1: Initial IP Scan with NSE

Start with a basic scan to discover open ports and services:

```bash
nmap -sS -sV -O -p- -sC -oX initial_scan.xml 192.168.1.10
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
nmap -sS --script ftp-anon -p 21 192.168.1.10

# SMB shares and users
nmap -sS --script smb-enum-shares,smb-enum-users -p 139,445 192.168.1.10

# Web directories and vulnerabilities
nmap -sS --script http-enum,http-vuln* -p 80,443 192.168.1.10

# SMTP user enumeration
nmap -sS --script smtp-enum-users -p 25 192.168.1.10

# SSH algorithms and hostkey
nmap -sS --script ssh2-enum-algos -p 22 192.168.1.10
```

Save each result for later reference:
```bash
nmap -sS --script ftp-anon -p 21 -oN ftp_enum.txt 192.168.1.10
nmap -sS --script smb-enum-shares -p 445 -oN smb_enum.txt 192.168.1.10
```

---

### Step 3: Export Results to XML

You already used `-oX` in Step 1, but you can also export additional scans:

```bash
nmap -sS --script smb-enum-shares -oX smb_scan.xml 192.168.1.10
nmap -sS --script http-enum -oX web_scan.xml 192.168.1.10
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
workspace -a enumeration_target_192.168.1.10
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
db_nmap -sS -sV -oX db_scan.xml 192.168.1.10
db_nmap -sS -Pn -p 80,443 192.168.1.10
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
set RHOSTS 192.168.1.10
run

# SMB enum
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 192.168.1.10
run

# SMB user enumeration
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS 192.168.1.10
run

# SSH version check
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.1.10
run

# SMTP user enumeration
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 192.168.1.10
run

# HTTP directory enumeration
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.1.10
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
