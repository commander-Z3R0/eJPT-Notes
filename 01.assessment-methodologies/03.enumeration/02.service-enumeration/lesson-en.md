# Service Enumeration

**Service enumeration** is the process of identifying and analyzing network services running on a target system. In penetration testing, this phase comes after **footprinting and scanning**, and it focuses on gathering detailed information about services like **FTP, SMB, Web, MySQL, SSH, and SMTP** to find misconfigurations, weak credentials, and potential vulnerabilities.

This lesson covers **service enumeration using Metasploit auxiliary modules**, including port scanning, module selection, RHOST configuration, and credential testing with wordlists—all within the **reconnaissance/footprinting phase** (no exploitation).

---

## Why Service Enumeration Matters

Service enumeration helps you:

- Identify **running services** and their versions.
- Discover **open shares** (SMB), **anonymous FTP access**, and **web directories**.
- Find **weak or default credentials** (SSH, MySQL, FTP).
- Detect **misconfigurations** that could lead to unauthorized access.
- Gather intelligence for **subsequent penetration testing phases**.

Metasploit provides **auxiliary scanner modules** specifically designed for service enumeration without exploitation.

---

## Metasploit Workflow for Service Enumeration

### General Workflow

1. **Launch Metasploit**: `msfconsole`
2. **Create a workspace**: `workspace -a service_enum_target`
3. **Search for auxiliary modules**: `search auxiliary/scanner`
4. **Select module**: `use auxiliary/scanner/...`
5. **Set target**: `set RHOSTS <target_ip>`
6. **Configure options** (ports, threads, wordlists): `set ...`
7. **Run**: `run` or `exploit`

---

## 1. FTP Enumeration

FTP is one of the most common services to enumerate. You check for **anonymous login**, **open shares**, and **weak credentials**.

### Step 1: Port Scan to Find FTP
```bash
db_nmap -sS -sV -p 21 <target_ip>
```

### Step 2: Check Anonymous FTP Access
```bash
msf6 > use auxiliary/scanner/ftp/anonymous
msf6 auxiliary(scanner/ftp/anonymous) > set RHOSTS <target_ip>
RHOSTS => <target_ip>
msf6 auxiliary(scanner/ftp/anonymous) > run
```

If anonymous login succeeds, you've found an open FTP share.

### Step 3: FTP Login Brute-force (Credential Testing)
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

Or use separate username and password wordlists:
```bash
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ftp/ftp_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

**Key outputs**:
- `+` indicates successful login.
- `-` indicates failed login.

---

## 2. SMB Enumeration

SMB (Server Message Block) is used for file sharing in Windows and Linux (Samba). Enumeration focuses on **shares**, **users**, and **security mode**.

### Step 1: Port Scan to Find SMB
```bash
db_nmap -sS -sV -p 139,445 <target_ip>
```

### Step 2: Enumerate SMB Shares
```bash
msf6 > use auxiliary/scanner/smb/smb_enumshares
msf6 auxiliary(scanner/smb/smb_enumshares) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumshares) > run
```

This lists all shared folders accessible on the target.

### Step 3: Enumerate SMB Users
```bash
msf6 > use auxiliary/scanner/smb/smb_enumusers
msf6 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_enumusers) > run
```

This lists user accounts on the SMB server.

### Step 4: SMB Security Mode
```bash
msf6 > use auxiliary/scanner/smb/smb_security_mode
msf6 auxiliary(scanner/smb/smb_security_mode) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_security_mode) > run
```

This reveals the SMB security configuration (e.g., NTLMv1 vs NTLMv2).

### Step 5: SMB Login Brute-force
```bash
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smb/smb_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/smb/smb_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/smb/smb_login) > run
```

---

## 3. Web Server Enumeration (Apache)

Web server enumeration focuses on **directories**, **headers**, **versions**, and **vulnerabilities**.

### Step 1: Port Scan to Find Web
```bash
db_nmap -sS -sV -p 80,443 <target_ip>
```

### Step 2: HTTP Header Enumeration
```bash
msf6 > use auxiliary/scanner/http/http_header
msf6 auxiliary(scanner/http/http_header) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_header) > set RPORT 80
msf6 auxiliary(scanner/http/http_header) > run
```

This shows server headers (e.g., Apache version, security headers).

### Step 3: Directory Brute-force (Web Enumeration)
```bash
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(scanner/http/dir_scanner) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/dir_scanner) > set RPORT 80
msf6 auxiliary(scanner/http/dir_scanner) > set WORDLIST /usr/share/metasploit-framework/data/wordlists/dirbuster.txt
msf6 auxiliary(scanner/http/dir_scanner) > set THREADS 10
msf6 auxiliary(scanner/http/dir_scanner) > run
```

This discovers hidden directories like `/admin`, `/backup`, `/config`.

### Step 4: Apache Version Detection
```bash
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(scanner/http/http_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/http/http_version) > run
```

---

## 4. MySQL Enumeration

MySQL enumeration focuses on **version detection**, **database listing**, and **authentication testing**.

### Step 1: Port Scan to Find MySQL
```bash
db_nmap -sS -sV -p 3306 <target_ip>
```

### Step 2: MySQL Information Enumeration
```bash
msf6 > use auxiliary/scanner/mysql/mysql_info
msf6 auxiliary(scanner/mysql/mysql_info) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_info) > run
```

This shows MySQL version and configuration details.

### Step 3: MySQL Login Brute-force
```bash
msf6 > use auxiliary/scanner/mysql/mysql_login
msf6 auxiliary(scanner/mysql/mysql_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/mysql/mysql_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/mysql/mysql_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/mysql/mysql_login) > run
```

Common default credentials: `root:`, `root:root`, `root:password`.

---

## 5. SSH Enumeration

SSH enumeration focuses on **algorithm detection**, **hostkey verification**, and **login testing**.

### Step 1: Port Scan to Find SSH
```bash
db_nmap -sS -sV -p 22 <target_ip>
```

### Step 2: SSH Version and Algorithm Detection
```bash
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_version) > run
```

This shows the SSH version (e.g., SSH-2.0-OpenSSH_8.4).

### Step 3: SSH Hostkey Enumeration
```bash
msf6 > use auxiliary/scanner/ssh/ssh_hostkey
msf6 auxiliary(scanner/ssh/ssh_hostkey) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_hostkey) > run
```

This retrieves the SSH hostkey fingerprint.

### Step 4: SSH Login Brute-force (Credential Testing)
```bash
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/ssh/ssh_login) > set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > THREADS 10
msf6 auxiliary(scanner/ssh/ssh_login) > run
```

This tests username/password combinations against SSH. Successful logins show `+`.

**Wordlist paths**:
- `/usr/share/metasploit-framework/data/wordlists/default_users.txt`
- `/usr/share/metasploit-framework/data/wordlists/default_passwords.txt`
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`

---

## 6. SMTP Enumeration (Bonus)

SMTP enumeration focuses on **user enumeration** and **command testing**.

### Step 1: Port Scan to Find SMTP
```bash
db_nmap -sS -sV -p 25 <target_ip>
```

### Step 2: SMTP User Enumeration
```bash
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS <target_ip>
msf6 auxiliary(scanner/smtp/smtp_enum) > run
```

This lists email users on the SMTP server using VRFY/EXPN commands.

---

## Complete Service Enumeration Workflow Example

Here's a full workflow from scan to service enumeration for a single target:

```bash
# 1. Launch Metasploit
msfconsole

# 2. Create workspace
workspace -a service_enum_<target_ip>

# 3. Initial port scan with database storage
db_nmap -sS -sV -p- -oX full_scan.xml <target_ip>

# 4. Import scan results
db_import full_scan.xml

# 5. List discovered services
services

# 6. FTP enumeration
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run

# 7. SMB enumeration
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run

use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run

# 8. Web enumeration
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set RPORT 80
set THREADS 10
run

# 9. MySQL enumeration
use auxiliary/scanner/mysql/mysql_login
set RHOSTS <target_ip>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
run

# 10. SSH enumeration
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <target_ip>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/default_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/default_passwords.txt
set THREADS 10
run

# 11. Export results
db_export -f xml enumeration_results.xml
```

---

## Metasploit Auxiliary Scanner Modules Summary

| Service | Port | Auxiliary Module | Purpose |
|---------|------|------------------|---------|
| **FTP** | 21 | `auxiliary/scanner/ftp/anonymous` | Check anonymous login |
| **FTP** | 21 | `auxiliary/scanner/ftp/ftp_login` | Login brute-force |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumshares` | Enumerate shares |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_enumusers` | Enumerate users |
| **SMB** | 139,445 | `auxiliary/scanner/smb/smb_login` | Login brute-force |
| **Web** | 80,443 | `auxiliary/scanner/http/dir_scanner` | Directory brute-force |
| **Web** | 80,443 | `auxiliary/scanner/http/http_header` | Header enumeration |
| **Web** | 80,443 | `auxiliary/scanner/http/http_version` | Version detection |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_info` | Service info |
| **MySQL** | 3306 | `auxiliary/scanner/mysql/mysql_login` | Login brute-force |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_version` | Version detection |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_hostkey` | Hostkey enumeration |
| **SSH** | 22 | `auxiliary/scanner/ssh/ssh_login` | Login brute-force |
| **SMTP** | 25 | `auxiliary/scanner/smtp/smtp_enum` | User enumeration |

---

## Key Metasploit Commands for Service Enumeration

| Command | Purpose |
|---------|---------|
| `msfconsole` | Launch Metasploit console |
| `workspace -a <name>` | Create new workspace |
| `db_nmap <options> <target>` | Run Nmap with database storage |
| `db_import <file>` | Import scan results |
| `hosts` | List discovered hosts |
| `services` | List discovered services |
| `search auxiliary/scanner` | Search scanner modules |
| `use auxiliary/scanner/...` | Select module |
| `set RHOSTS <ip>` | Set target IP |
| `set RPORT <port>` | Set target port |
| `set USER_FILE <path>` | Set username wordlist |
| `set PASS_FILE <path>` | Set password wordlist |
| `set THREADS <num>` | Set number of threads |
| `run` or `exploit` | Execute module |
| `db_export -f xml <file>` | Export database to XML |

---

## Wordlist Paths in Metasploit

Metasploit includes built-in wordlists for credential testing:

```bash
/usr/share/metasploit-framework/data/wordlists/default_users.txt
/usr/share/metasploit-framework/data/wordlists/default_passwords.txt
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
/usr/share/metasploit-framework/data/wordlists/dirbuster.txt
```

Use these with `USER_FILE`, `PASS_FILE`, and `WORDLIST` options.

---

## Key Takeaways

- **Service enumeration** identifies and analyzes running services (FTP, SMB, Web, MySQL, SSH, SMTP).
- Metasploit **auxiliary scanner modules** are designed for reconnaissance without exploitation.
- Always start with **port scanning** (`db_nmap`) to identify open ports and services.
- Use **`set RHOSTS`** to specify the target IP for each module.
- **FTP**: Check anonymous login with `auxiliary/scanner/ftp/anonymous`.
- **SMB**: Enumerate shares and users with `smb_enumshares` and `smb_enumusers`.
- **Web (Apache)**: Use `dir_scanner` for directory brute-force and `http_header` for version info.
- **MySQL**: Use `mysql_info` for service details and `mysql_login` for credential testing.
- **SSH**: Use `ssh_version`, `ssh_hostkey`, and `ssh_login` for version, hostkey, and credential testing.
- **SSH login brute-force** uses `auxiliary/scanner/ssh/ssh_login` with username and password wordlists.
- **SMTP**: Use `smtp_enum` for user enumeration.
- Wordlists are located in `/usr/share/metasploit-framework/data/wordlists/`.
- Use **workspaces** to keep assessments organized.
