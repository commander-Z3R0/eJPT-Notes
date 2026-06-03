# Port Scanning

Port scanning is the process of identifying **open ports** on a target host. Once you know a host is alive (from host discovery), port scanning tells you **what services are running** and potentially vulnerable.

## What is Port Scanning?

In penetration testing and reconnaissance, port scanning helps you:

- Discover **open ports** (services accepting connections).
- Identify **running services** (HTTP, SSH, SMTP, SMB, etc.).
- Determine **service versions** (for exploitation).
- Map the **attack surface** of the target.

Nmap supports many scanning techniques, each with different advantages depending on the network and firewall environment:

- Some scans are **stealthy** (harder to detect).
- Some scans work better **behind firewalls**.
- Some require **root privileges**, others don't.

---

## Port Scanning Techniques

### 1. TCP SYN Scan (Stealth Scan)

**TCP SYN scan** is the **default and most popular** scan in Nmap. It sends TCP SYN packets and analyzes responses without completing the TCP handshake. This is why it's called a **half-open scan**.

- **Nmap option**: `-sS`
- **Requires**: root/administrator privileges (raw packet access)
- **How it works**:
  1. Send TCP SYN packet to target port.
  2. If port is **open**: receive **SYN/ACK** → port is open.
  3. If port is **closed**: receive **RST** → port is closed.
  4. If **filtered**: no response or ICMP error → port is filtered.
  5. Nmap sends **RST** to tear down (never sends ACK).

- **Pros**:
  - **Fast** and **stealthy** (doesn't complete connections).
  - Works in most environments.
  - Standard for penetration testing.
- **Cons**:
  - Requires **root** on Unix/Linux.
  - Can still be logged by IDS/IPS.

Example:
```bash
nmap -sS 192.168.1.5
nmap -sS -p 22,80,443 192.168.1.5
nmap -sS -p- 192.168.1.5        # scan all 65535 ports
```

---

### 2. TCP Connect Scan

**TCP connect scan** uses the system's **connect()** function to complete the full TCP handshake. This is used when you **don't have root privileges**.

- **Nmap option**: `-sT`
- **Requires**: no root privileges (user-level)
- **How it works**:
  1. Send TCP SYN.
  2. If port is **open**: receive SYN/ACK → send ACK → connection established.
  3. If port is **closed**: receive RST.
  4. Nmap closes the connection immediately after.

- **Pros**:
  - Works **without root** privileges.
  - Reliable and well-supported.
- **Cons**:
  - **More noisy** (completes full connections, easier to log).
  - **Slower** than SYN scan.

Example:
```bash
nmap -sT 192.168.1.5
nmap -sT -p 80,443 192.168.1.5
```

---

### 3. UDP Scan

**UDP scan** probes UDP ports to find open UDP services. UDP is **connectionless**, so scanning is slower and less reliable than TCP .

- **Nmap option**: `-sU`
- **Requires**: root privileges (usually)
- **How it works**:
  1. Send empty UDP packet (or protocol-specific) to port.
  2. If receive **ICMP port unreachable** → port is **closed**.
  3. If receive **UDP response** → port is **open**.
  4. If **no response** → port is **open|filtered** (uncertain).

- **Pros**:
  - Discovers **UDP services** (DNS, DHCP, SNMP, NTP, etc.).
  - Important for complete reconnaissance.
- **Cons**:
  - **Very slow** (many timeouts).
  - Less reliable (many hosts don't send ICMP errors).
  - Often filtered by firewalls.

Example:
```bash
nmap -sU 192.168.1.5
nmap -sU -p 53,161,123 192.168.1.5      # DNS, SNMP, NTP
nmap -sU --top-ports 100 192.168.1.5    # top 100 UDP ports
```

---

### 4. TCP ACK Scan

**TCP ACK scan** sends TCP packets with the **ACK flag** set. It's primarily used to **map firewall rulesets**, not to find open ports.

- **Nmap option**: `-sA`
- **Requires**: root privileges
- **How it works**:
  1. Send TCP ACK packet to port.
  2. If receive **RST** → port is **unfiltered**.
  3. If **no response** or ICMP error → port is **filtered**.

- **Pros**:
  - Good for **firewall mapping**.
  - Bypasses some **stateless firewalls**.
- **Cons**:
  - **Cannot determine open vs closed** (only unfiltered vs filtered).
  - Not useful for finding services.

Example:
```bash
nmap -sA 192.168.1.5
nmap -sA -p 80,443 192.168.1.5
```

---

### 5. TCP Window Scan

**TCP window scan** is similar to ACK scan but checks the **TCP window size** in the RST response to determine open vs closed ports.

- **Nmap option**: `-sW`
- **Requires**: root privileges
- **How it works**:
  1. Send TCP ACK packet.
  2. If RST has **non-zero window** → port is **open**.
  3. If RST has **zero window** → port is **closed**.

- **Pros**:
  - Can detect open ports (unlike ACK scan).
  - Sometimes bypasses firewalls.
- **Cons**:
  - Rarely more useful than SYN scan.
  - Not reliable on all systems.

Example:
```bash
nmap -sW 192.168.1.5
```

---

### 6. TCP NULL, FIN, and Xmas Scans

These are **stealth scans** that send packets with **no flags** (NULL), **only FIN**, or **FIN+URG+PSH** (Xmas). They exploit how systems handle malformed packets.

- **Nmap options**:
  - `-sN` (NULL scan)
  - `-sF` (FIN scan)
  - `-sX` (Xmas scan)
- **Requires**: root privileges
- **How it works**:
  - Send packet with **no SYN** (bypasses some stateful firewalls).
  - If receive **RST** → port is **closed**.
  - If **no response** → port is **open|filtered**.

- **Pros**:
  - **Stealthy** (no SYN flag, harder to detect).
  - Can bypass some firewalls.
- **Cons**:
  - Only works on **Unix/Linux** (Windows responds differently).
  - Not reliable; many modern systems respond with RST anyway.

Example:
```bash
nmap -sN 192.168.1.5
nmap -sF 192.168.1.5
nmap -sX 192.168.1.5
```

---

## Port Selection

You can specify which ports to scan using `-p` :

| Option | Description |
|--------|-------------|
| `-p 80,443,22` | Scan specific ports |
| `-p 1-1000` | Scan ports 1 through 1000 |
| `-p-` | Scan **all 65535 ports** |
| `--top-ports 100` | Scan **top 100 most common ports** |
| `-p U:53,T:80,443` | Scan UDP 53 + TCP 80,443 |

Example:
```bash
nmap -sS -p 22,80,443 192.168.1.5
nmap -sS -p- 192.168.1.5
nmap -sS --top-ports 1000 192.168.1.5
```

---

## Service Version Detection

**-sV** probes open ports to determine **service version** (e.g., Apache 2.4.49, OpenSSH 8.4) :

- **Nmap option**: `-sV`
- **Use case**: when you need to know exact versions for exploitation.

Example:
```bash
nmap -sS -sV 192.168.1.5
nmap -sS -sV --version-intensity 5 192.168.1.5   # max intensity
```

---

## OS Detection

**-O** attempts to detect the **operating system** of the target:

- **Nmap option**: `-O`
- **Requires**: root privileges
- **How it works**: Analyzes TCP/IP stack fingerprints.

Example:
```bash
nmap -sS -O 192.168.1.5
```

---

## Aggressive Scan (A)

**-A** enables **OS detection, version detection, script scanning, and traceroute** all at once:

- **Nmap option**: `-A`
- **Equivalent to**: `-O -sV -sC --traceroute`
- **Use case**: when you want maximum information.

Example:
```bash
nmap -A 192.168.1.5
```

---

## Nmap Scripting Engine (NSE)

**-sC** runs **default NSE scripts** for additional reconnaissance (vulnerabilities, configs, etc.):

- **Nmap option**: `-sC`
- **Use case**: when you want to run common vulnerability checks.

Example:
```bash
nmap -sS -sC 192.168.1.5
nmap --script vuln 192.168.1.5
nmap --script http-enum 192.168.1.5
```

---

## Common Scan Combinations

| Scenario | Command |
|----------|---------|
| **Quick top 100 ports** | `nmap -sS --top-ports 100 <target>` |
| **Full TCP scan** | `nmap -sS -p- <target>` |
| **Full scan + version + OS** | `nmap -sS -sV -O -p- <target>` |
| **Aggressive scan** | `nmap -A <target>` |
| **UDP scan (top 100)** | `nmap -sU --top-ports 100 <target>` |
| **TCP + UDP full scan** | `nmap -sS -sU -p- <target>` |
| **Without root** | `nmap -sT -p 80,443 <target>` |
| **Firewall mapping** | `nmap -sA <target>` |

---

## Scan States

Nmap reports ports in these states :

| State | Meaning |
|-------|---------|
| **open** | Service is accepting connections |
| **closed** | Host is reachable, but no service on port |
| **filtered** | Firewall/IDS is blocking the scan |
| **unfiltered** | Port is accessible, but open/closed unknown |
| **open\|filtered** | Nmap cannot determine if open or filtered |
| **closed\|filtered** | Nmap cannot determine if closed or filtered |

---

## Key Takeaways

- **TCP SYN scan (`-sS`)** is the default; fast, stealthy, requires root.
- **TCP Connect scan (`-sT`)** is for non-root users; slower, more visible.
- **UDP scan (`-sU`)** is slow but necessary for UDP services (DNS, SNMP).
- **ACK scan (`-sA`)** maps firewall rules, not open ports.
- **NULL/FIN/Xmas scans** are stealthy but unreliable on Windows.
- **`-sV`** probes service versions for exploitation.
- **`-O`** detects the target OS.
- **`-A`** is aggressive: OS + version + scripts + traceroute.
- **`-sC`** runs default NSE scripts for vulnerability checks.
- Use **`-p-`** for full port scan (all 65535 ports).
- Use **`--top-ports 100`** for quick scan of most common ports.
