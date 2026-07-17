# Networking

The purpose of networking is to exchange information between hosts. Packets are just bits (electrical signals) which, when translated by a computer, make up information.

---

## Networking Fundamentals

This lesson covers the fundamental networking concepts needed for network-based penetration testing — including how data flows across networks, the OSI model, key Network and Transport layer protocols, and essential IP addressing concepts.

### How Networks Communicate

In a computer network, hosts communicate through **network protocols** — standardized rules that ensure different systems can exchange information. This communication is carried by **packets**.

Each packet has two parts:

| Part | Description |
|------|-------------|
| **Header** | Protocol-specific structure (TCP, UDP, IP) that tells the destination host how to interpret the data. Contains source/destination IPs, version, TTL, protocol type. |
| **Payload** | The actual data or information being transmitted. |

---

### The OSI Model

The **OSI (Open Systems Interconnection)** model standardizes network communication into 7 layers. It is not a strict blueprint, but helps understand and design network architecture.

| Layer | Name | Function | Key Protocols |
|-------|------|----------|--------------|
| 7 | **Application** | User-oriented services and interfaces | HTTP, FTP, SMTP, DNS |
| 6 | **Presentation** | Data formatting, encryption, compression | SSL/TLS, JPEG |
| 5 | **Session** | Manages sessions between applications | NetBIOS, RPC |
| 4 | **Transport** | End-to-end communication, error control | TCP, UDP |
| 3 | **Network** | Logical addressing and routing | IP, ICMP, DHCP |
| 2 | **Data Link** | Physical addressing, frame delivery | Ethernet, MAC, ARP |
| 1 | **Physical** | Raw bit transmission over the medium | Cables, Wi-Fi, fiber |

> For penetration testing, **Layers 3 (Network) and 4 (Transport)** are the most critical — they govern IP addressing, routing, port scanning, and how connections are established.

---

### Layer 3 — The Network Layer

Responsible for **logical addressing**, **routing**, and **forwarding packets** between devices on different networks.

#### Key Protocols

| Protocol | Use |
|----------|-----|
| **IPv4** | Most used IP version — 32-bit addresses (e.g. `192.168.1.1`) |
| **IPv6** | Successor to IPv4 — 128-bit addresses, developed to address IPv4 exhaustion |
| **ICMP** | Internet Control Message Protocol — error reporting and diagnostics (ping, traceroute) |
| **DHCP** | Dynamic Host Configuration Protocol — automatically assigns IP addresses to devices |

#### Main IP Functions

| Function | Description |
|----------|-------------|
| **Logical addressing** | IP addresses uniquely identify each device on the network |
| **Packet structure** | Data organized into packets with a header and payload |
| **Fragmentation & reassembly** | Large packets are split to fit MTU limits, then reassembled at destination |
| **Routing** | Packets forwarded hop-by-hop toward the destination IP |

#### IP Addressing Types

| Type | Description |
|------|-------------|
| **Unicast** | One-to-one communication |
| **Broadcast** | One-to-all within a subnet |
| **Multicast** | One-to-many for a selected group |

#### Reserved IPv4 Addresses (RFC 5735)

| Range | Use |
|-------|-----|
| `0.0.0.0 – 0.255.255.255` | Represents the network you are connected to |
| `127.0.0.0 – 127.255.255.255` | Loopback / localhost (your own machine) |
| `192.168.0.0 – 192.168.255.255` | Private networks (LAN) |
| `10.0.0.0 – 10.255.255.255` | Private networks (LAN) |
| `172.16.0.0 – 172.31.255.255` | Private networks (LAN) |

#### IPv4 Packet Header Fields

| Field | Size | Description |
|-------|------|-------------|
| **Source IP** | 32 bits | Source address of the packet |
| **Destination IP** | 32 bits | Target address of the packet |
| **TTL (Time To Live)** | 8 bits | Decremented at each hop — packet dropped when TTL reaches 0 |
| **Protocol** | 8 bits | Transport layer protocol (6 = TCP, 17 = UDP, 1 = ICMP) |
| **ToS (Type of Service)** | 8 bits | Priority of each packet for QoS |
| **Version** | 4 bits | IPv4 or IPv6 |

#### ICMP — Internet Control Message Protocol

| ICMP Message | Type | Code | Use |
|-------------|------|------|-----|
| **Echo Request** | 8 | 0 | Sent by `ping` to test host reachability |
| **Echo Reply** | 0 | 0 | Response confirming the host is active |
| **Destination Unreachable** | 3 | — | Host or port unreachable |
| **Time Exceeded** | 11 | 0 | TTL expired in transit (used by `traceroute`) |

---

### Layer 4 — The Transport Layer

Provides **end-to-end communication** between applications. Handles error detection, flow control, and data segmentation.

#### TCP — Transmission Control Protocol

**Connection-oriented** — establishes a reliable connection before data transfer.

| Feature | Description |
|---------|-------------|
| **Reliability** | Data sent accurately and in the correct order — uses ACK |
| **Retransmission** | Lost segments automatically resent |
| **Ordering** | Out-of-order segments reordered before delivery |
| **Connection** | Requires 3-way handshake before data exchange |

#### The TCP 3-Way Handshake

```
Client  →  SYN      →  Server    (I want to connect)
Client  ←  SYN-ACK  ←  Server    (Confirmed, I accept)
Client  →  ACK      →  Server    (Connection established)
```

> **SYN scans** (half-open) send a SYN but never complete the handshake — faster and avoids logging on the target.

#### TCP Control Flags

| Flag | Description |
|------|-------------|
| **SYN** | Initiates a connection request |
| **ACK** | Acknowledges received data |
| **FIN** | Initiates orderly connection termination |
| **RST** | Resets/abruptly terminates a connection |
| **PSH** | Pushes data immediately to the application |
| **URG** | Marks data as urgent |

#### TCP Port Ranges

| Range | Type | Description |
|-------|------|-------------|
| **0 – 1023** | Well-known | Reserved by IANA for standard services |
| **1024 – 49151** | Registered | Assigned by IANA for applications/vendors |
| **49152 – 65535** | Dynamic/ephemeral | Temporary ports used by clients |

#### Common Well-Known Ports

| Port | Protocol | Service |
|------|----------|---------|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 8080 | TCP | Alternate HTTP |

#### UDP — User Datagram Protocol

**Connectionless** — sends data without establishing a connection first.

| Feature | Description |
|---------|-------------|
| **Speed** | Faster than TCP — no handshake overhead |
| **Reliability** | No guarantee of delivery, ordering, or error correction |
| **Independence** | Each datagram treated independently |
| **Use cases** | DNS, DHCP, SNMP, VoIP, streaming, online gaming |

#### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery with ACK | No delivery guarantee |
| **Speed** | Slower | Faster |
| **Ordering** | Data arrives in order | No ordering |
| **Error correction** | Yes (retransmission) | No |
| **Use cases** | HTTP, HTTPS, SSH, FTP, SMB | DNS, VoIP, streaming |

---

### Subnetting and CIDR

**Subnetting** divides a large IP network into smaller, more manageable subnets — improves efficiency and security by isolating traffic between segments.

**CIDR (Classless Inter-Domain Routing)** notation expresses a network address with its prefix length:

| CIDR | Subnet Mask | Available Hosts |
|------|-------------|----------------|
| `/8` | 255.0.0.0 | 16,777,214 |
| `/16` | 255.255.0.0 | 65,534 |
| `/24` | 255.255.255.0 | 254 |
| `/28` | 255.255.255.240 | 14 |
| `/30` | 255.255.255.252 | 2 |
| `/32` | 255.255.255.255 | 1 (single host) |

**Example**: A scope of `200.200.0.0/16` = up to 65,536 IPs (`200.200.0.0 – 200.200.255.255`). During a pentest, your first job is to determine which addresses are active.

---

### Key Takeaways — Networking Fundamentals

- Networks communicate through **packets** — header (routing info) + payload (data).
- The **OSI model** has 7 layers — Layers 3 and 4 are the most relevant for pentesting.
- **Layer 3 (IP)**: logical addressing, routing — key protocols: IPv4, IPv6, ICMP, DHCP.
- **TTL** decrements at each hop — reaches 0, packet is dropped. Used by `traceroute`.
- **ICMP**: Echo Request (type 8) / Echo Reply (type 0) = `ping`; type 11 = TTL expired = `traceroute`.
- **TCP**: requires 3-way handshake — reliable, ordered, slower. SYN scans exploit the handshake.
- **UDP**: connectionless — faster, unreliable. Used by DNS, DHCP, SNMP.
- **TCP flags**: SYN, ACK, FIN, RST — each plays a role in scan types.
- Ports **0–1023** = well-known; **1024–49151** = registered; **49152–65535** = ephemeral.
- A `/16` scope = up to **65,534 hosts** — always start with host discovery before port scanning.

---

## Firewall Detection & IDS Evasion

This lesson covers how to detect firewalls and IDS/IPS systems during a scan, and the Nmap techniques used to evade or bypass basic filtering.

> ⚠️ **Note**: All commands assume you are operating from a **Kali Linux** attack machine. Replace `10.10.10.10` with your actual target IP.

---

### What are Firewalls and IDS/IPS?

| System | Full Name | Purpose |
|--------|-----------|---------|
| **Firewall** | — | Filters traffic based on rules — blocks or allows packets by port, IP, or protocol |
| **IDS** | Intrusion Detection System | Monitors traffic and **alerts** on suspicious activity — does not block |
| **IPS** | Intrusion Prevention System | Monitors traffic and **actively blocks** suspicious activity |

Understanding these systems is critical — not just to evade them, but to **test whether they are functioning correctly** as part of a pentest.

---

### Firewall Detection

#### Method 1: Port State Analysis

| Port State | Meaning | Firewall Present? |
|-----------|---------|------------------|
| **Open** | Service is listening and responding | No (or port is allowed) |
| **Closed** | Host responds with RST — port not in use | No firewall on this port |
| **Filtered** | No response or ICMP unreachable received | **Yes — firewall is blocking** |

```bash
nmap -sS -p 80,443,22,3389 10.10.10.10
```

If you see **`filtered`** on ports that should be open, a firewall is blocking the probes.

#### Method 2: ACK Scan (`-sA`)

Designed to **map firewall rules** — does not detect open ports, detects whether ports are filtered.

```bash
nmap -sA -p 80,443,22 10.10.10.10
```

| Response | Interpretation |
|----------|---------------|
| **RST received** | Port unfiltered — firewall allows ACK packets through |
| **No response / ICMP unreachable** | Port filtered — firewall is blocking |

> The ACK scan distinguishes **stateful** (drops unexpected ACK) from **stateless** (may let ACK through) firewalls.

#### Method 3: Window Scan (`-sW`)

Analyzes the **TCP window size** in RST responses to differentiate open from closed ports through certain firewall types.

```bash
nmap -sW -p 80,443 10.10.10.10
```

#### Method 4: Quick Comparison Test

```bash
nmap -sS -p 80 10.10.10.10   # SYN scan — port state
nmap -sA -p 80 10.10.10.10   # ACK scan — filtered or unfiltered
```

SYN = **filtered** + ACK = **unfiltered** → stateful firewall dropping SYN but not ACK packets.

---

### IDS/IPS Evasion Techniques

#### 1. Packet Fragmentation (`-f`, `--mtu`)

Splits probes into smaller IP fragments — older IDS/firewalls may fail to reassemble and inspect them.

```bash
nmap -sS -f 10.10.10.10           # 8-byte fragments
nmap -sS -ff 10.10.10.10          # 16-byte fragments
nmap -sS --mtu 24 10.10.10.10     # Custom fragment size (multiple of 8)
```

> Modern stateful firewalls **can** reassemble fragments — most effective against older or misconfigured systems.

#### 2. Decoy Scanning (`-D`)

Mixes fake source IPs with the real scan — target sees traffic from many IPs, hiding the real attacker.

```bash
nmap -sS -D 10.10.10.5,10.10.10.6,ME 10.10.10.10   # Manual decoys
nmap -sS -D RND:5 10.10.10.10                        # 5 random decoys
```

#### 3. Source Port Spoofing (`-g` / `--source-port`)

Some firewalls allow traffic from trusted ports (DNS 53, HTTP 80) without deep inspection.

```bash
nmap -sS -g 53 10.10.10.10     # Appear to come from DNS port
nmap -sS -g 80 10.10.10.10     # Appear to come from HTTP port
```

#### 4. Random Data Length (`--data-length`)

Appends random bytes to packets — makes probes less consistent and harder for signature-based IDS to match.

```bash
nmap -sS --data-length 25 10.10.10.10
```

#### 5. Disable DNS Resolution (`-n`)

Prevents DNS queries — each query is a potential detection point.

```bash
nmap -sS -n 10.10.10.10
```

#### 6. Source IP Spoofing (`-S`)

Makes traffic appear from a different host. Replies go to the spoofed IP — primarily used for **IDS rule testing**.

```bash
nmap -sS -S 192.168.1.50 -e eth0 10.10.10.10
```

#### 7. MAC Address Spoofing (`--spoof-mac`)

Spoofs the source MAC — useful on LANs with MAC-based filtering.

```bash
nmap -sS --spoof-mac 0 10.10.10.10            # Random MAC
nmap -sS --spoof-mac Apple 10.10.10.10        # Vendor MAC
nmap -sS --spoof-mac 00:11:22:33:44:55 10.10.10.10  # Specific MAC
```

---

### Timing-Based Evasion

IDS systems detect scans based on **packet volume and rate**. Slowing down reduces the detection footprint.

#### Timing Templates (`-T`)

| Template | Name | Speed | Stealth | Use Case |
|----------|------|-------|---------|---------|
| `-T0` | Paranoid | Extremely slow | Maximum | Evade most IDS |
| `-T1` | Sneaky | Very slow | High | Stealth scans |
| `-T2` | Polite | Slow | Medium | Reduce network load |
| `-T3` | Normal | Default | Default | Balanced starting point |
| `-T4` | Aggressive | Fast | Low | Reliable internal networks |
| `-T5` | Insane | Very fast | None | ⚠️ High packet loss risk |

```bash
nmap -sS -T1 10.10.10.10    # Slow stealth scan
nmap -sS -T3 10.10.10.10    # Standard scan
nmap -sS -T4 10.10.10.10    # Fast internal scan
```

#### Manual Timing Controls

```bash
nmap -sS --scan-delay 200ms 10.10.10.10   # Delay between probes
nmap --min-rate 50 10.10.10.10             # Minimum 50 packets/sec
nmap --host-timeout 5m 10.10.10.10         # Give up after 5 minutes per host
```

---

### Output Formats

Always save scan results for analysis, reporting, and Metasploit import.

| Flag | Format | Use |
|------|--------|-----|
| `-oN` | Normal text | Human-readable notes |
| `-oX` | XML | Metasploit import, tool integration |
| `-oG` | Grepable | Shell parsing and scripting |
| `-oA` | All formats | Saves all three at once |

```bash
nmap -sS -oN scan.txt 10.10.10.10       # Text
nmap -sS -oX scan.xml 10.10.10.10       # XML
nmap -sS -oA scan_results 10.10.10.10   # All formats
```

#### Import XML into Metasploit

```bash
nmap -sS -sV -oX scan.xml 10.10.10.10
msfconsole
msf6 > db_import scan.xml
msf6 > hosts
msf6 > services
```

---

### Combined Evasion Workflow

```bash
# Fragment + decoys + source port spoof + no DNS + slow timing + XML output
nmap -sS -Pn -f -D RND:5 -g 53 -n -T2 --data-length 15 -oX evasion_scan.xml 10.10.10.10
```

| Flag | Purpose |
|------|---------|
| `-sS` | SYN stealth scan |
| `-Pn` | Skip host discovery |
| `-f` | Fragment packets |
| `-D RND:5` | 5 random decoy IPs |
| `-g 53` | Source port 53 (DNS) |
| `-n` | No DNS resolution |
| `-T2` | Polite timing |
| `--data-length 15` | 15 random bytes appended |
| `-oX` | Save as XML |

---

### Evasion Techniques Summary

| Technique | Flag | Evades |
|-----------|------|-------|
| Packet fragmentation | `-f`, `--mtu` | Simple packet filters, older IDS |
| Decoy scanning | `-D RND:n` | IP-based detection, source tracking |
| Source port spoof | `-g <port>` | Port-based firewall rules |
| Random data length | `--data-length` | Signature-based IDS |
| Disable DNS | `-n` | DNS-based detection |
| Source IP spoof | `-S` | IDS alert rule testing |
| MAC spoof | `--spoof-mac` | MAC-based filtering (LAN) |
| Slow timing | `-T0`, `-T1`, `--scan-delay` | Rate-based IDS thresholds |
| Skip host discovery | `-Pn` | ICMP-blocking firewalls |

---

### Key Takeaways — Firewall Detection & IDS Evasion

- **Firewalls** filter by port/IP/protocol — detected by **filtered** port states in Nmap output.
- **IDS** alerts; **IPS** blocks — both analyze packet patterns and rates.
- **ACK scan** (`-sA`): RST = unfiltered, no response = filtered — best tool for mapping firewall rules.
- **Stateful firewalls** drop unexpected ACK packets; **stateless** may let them through.
- **Fragmentation** (`-f`) = 8-byte chunks — effective against older systems, not modern stateful firewalls.
- **Decoys** (`-D RND:5`) hide the real scanner among random source IPs.
- **Source port spoof** (`-g 53`) bypasses firewall rules that trust traffic from port 53 or 80.
- **Slow timing** (`-T0`, `-T1`, `--scan-delay`) evades rate-based IDS by reducing packets per second.
- **`-Pn`** skips ICMP host discovery — use when target blocks pings but has open ports.
- **Never use `-T5`** in real assessments — causes packet loss and inaccurate results.
- Best evasion combo: `-f -D RND:5 -g 53 -n -T2 --data-length 15`.
- Always save with **`-oX`** (XML) for Metasploit import via `db_import`.
