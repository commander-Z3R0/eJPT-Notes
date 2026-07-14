# Networking

The purpose of networking is to exchange information between hosts. Packets are just bits (i.e. electrical signals), which when translated by a computer, make up information

## Networking Fundamentals

This lesson covers the fundamental networking concepts needed for network-based penetration testing — including how data flows across networks, the OSI model, key Network and Transport layer protocols, and essential IP addressing concepts.

### How Networks Communicate

In a computer network, hosts communicate through network protocols — standardized rules that ensure different systems can exchange information. This communication is carried by packets.

Each packet has two parts:

| Part | Description |
|---|---|
| Header | Protocol-specific structure (TCP, UDP, IP) that tells the destination host how to interpret the data. Contains source/destination IPs, version, TTL, protocol type. |
| Payload | The actual data or information being transmitted. |

### The OSI Model

The OSI model (Open Systems Interconnection) is a conceptual framework that standardizes network communication into 7 layers. It is not a strict blueprint, but it helps understand and design network architecture.

| Layer | Name | Function | Key Protocols |
|---|---|---|---|
| 7 | Application | User-oriented services and interfaces | HTTP, FTP, SMTP, DNS |
| 6 | Presentation | Data formatting, encryption, compression | SSL/TLS, JPEG |
| 5 | Session | Manages sessions between applications | NetBIOS, RPC |
| 4 | Transport | End-to-end communication, error control | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP, DHCP |
| 2 | Data Link | Physical addressing, frame delivery | Ethernet, MAC, ARP |
| 1 | Physical | Raw bit transmission over the medium | Cables, Wi-Fi, fiber |

For penetration testing, layers 3 (Network) and 4 (Transport) are the most critical — they govern IP addressing, routing, port scanning, and how connections are established.

### Layer 3 — The Network Layer

The Network layer is responsible for logical addressing, routing, and forwarding packets between devices on different networks. It determines the optimal path for data to reach its destination.

#### Key Protocols

| Protocol | Use |
|---|---|
| IPv4 | Most used IP version — 32-bit addresses (e.g. 192.168.1.1) |
| IPv6 | Successor to IPv4 — 128-bit addresses, developed to address IPv4 exhaustion |
| ICMP | Internet Control Message Protocol — error reporting and diagnostics (ping, traceroute) |
| DHCP | Dynamic Host Configuration Protocol — automatically assigns IP addresses to devices |

#### Main IP Functions

| Function | Description |
|---|---|
| Logical addressing | IP addresses uniquely identify each device on the network |
| Packet structure | Data is organized into packets with a header and payload |
| Fragmentation and reassembly | Large packets are split to fit MTU limits, then reassembled at the destination |
| Routing | Packets are forwarded hop by hop toward the destination IP |

#### IP Addressing Types

| Type | Description |
|---|---|
| Unicast | One-to-one communication |
| Broadcast | One-to-all communication within a subnet |
| Multicast | One-to-many communication for a selected group |

#### Reserved IPv4 Addresses (RFC 5735)

| Range | Use |
|---|---|
| 0.0.0.0 – 0.255.255.255 | Represents the network you are connected to |
| 127.0.0.0 – 127.255.255.255 | Loopback / localhost (your own machine) |
| 192.168.0.0 – 192.168.255.255 | Private networks (LAN) |
| 10.0.0.0 – 10.255.255.255 | Private networks (LAN) |
| 172.16.0.0 – 172.31.255.255 | Private networks (LAN) |

#### IPv4 Packet Header Fields

| Field | Size | Description |
|---|---|---|
| Source IP | 32 bits | Source address of the packet |
| Destination IP | 32 bits | Target address of the packet |
| TTL (Time To Live) | 8 bits | Decremented at each hop — packet is dropped when TTL reaches 0 |
| Protocol | 8 bits | Identifies the transport layer protocol (6 = TCP, 17 = UDP, 1 = ICMP) |
| ToS (Type of Service) | 8 bits | Priority of each packet for QoS |
| Version | 4 bits | IPv4 or IPv6 |

#### ICMP — Internet Control Message Protocol

ICMP is closely associated with IP and used for diagnostics and error reporting:

| ICMP Message | Type | Code | Use |
|---|---|---|---|
| Echo Request | 8 | 0 | Sent by ping to test host reachability |
| Echo Reply | 0 | 0 | Response confirming the host is active |
| Destination Unreachable | 3 | — | Host or port unreachable |
| Time Exceeded | 11 | 0 | TTL expired in transit (used by traceroute) |

### Layer 4 — The Transport Layer

The Transport layer provides end-to-end communication between applications on different hosts. It handles error detection, flow control, and data segmentation.

The two main protocols are TCP and UDP.

#### TCP — Transmission Control Protocol

Connection-oriented — establishes a reliable connection before data transfer begins.

| Feature | Description |
|---|---|
| Reliability | Data is sent accurately and in the correct order — uses ACK |
| Retransmission | Lost segments are automatically resent |
| Ordering | Out-of-order segments are reordered before delivery |
| Connection | Requires a 3-way handshake before data exchange |

#### The TCP 3-Way Handshake

Client → SYN → Server (I want to connect)  
Client ← SYN-ACK ← Server (Confirmed, I accept)  
Client → ACK → Server (Connection established)

This handshake is critical for penetration testing — SYN scans (half-open) exploit it by sending a SYN but never completing the handshake, avoiding logging on the target.

#### TCP Control Flags

| Flag | Description |
|---|---|
| SYN | Initiates a connection request |
| ACK | Acknowledges data |
| FIN | Initiates orderly connection termination |
| RST | Resets/abruptly terminates a connection |
| PSH | Pushes data immediately to the application |
| URG | Marks data as urgent |

#### TCP Port Ranges

| Range | Type | Description |
|---|---|---|
| 0 – 1023 | Well-known ports | Reserved by IANA for standard services |
| 1024 – 49151 | Registered ports | Assigned by IANA for applications/vendors |
| 49152 – 65535 | Dynamic/ephemeral | Temporary ports used by clients |

#### Common Well-Known Ports

| Port | Protocol | Service |
|---|---|---|
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

Connectionless — sends data without first establishing a connection.

| Feature | Description |
|---|---|
| Speed | Faster than TCP — no handshake overhead |
| Reliability | No guarantee of delivery, ordering, or error correction |
| Independence | Each datagram is treated independently |
| Use cases | DNS, DHCP, SNMP, VoIP, streaming, online gaming |

#### TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery with ACK | No delivery guarantee |
| Speed | Slower | Faster |
| Ordering | Data arrives in order | No ordering |
| Error correction | Yes (retransmission) | No |
| Use cases | HTTP, HTTPS, SSH, FTP, SMB | DNS, VoIP, streaming |

### Subnetting and CIDR

Subnetting divides a large IP network into smaller, more manageable subnets. It improves network efficiency and security by isolating traffic between segments.

CIDR (Classless Inter-Domain Routing) notation expresses a network address with its prefix length:

| CIDR | Subnet Mask | Available Hosts |
|---|---|---|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 |
| /32 | 255.255.255.255 | 1 (single host) |

Example: A perimeter of 200.200.0.0/16 means the network could contain up to 65,536 IP addresses (200.200.0.0 – 200.200.255.255). During a pentest, your first job is to determine which of those addresses are active.

### Key Points

- Networks communicate through packets — each with a header (routing information) and a payload (data).
- The OSI model has 7 layers — layers 3 (Network) and 4 (Transport) are the most relevant for pentesting.
- Layer 3 (IP) handles logical addressing, routing, and packet forwarding — key protocols: IPv4, IPv6, ICMP, DHCP.
- The IPv4 header contains Source IP, Destination IP, TTL, Protocol, and ToS — all critical for understanding packet behavior.
- TTL is decremented at each hop — it reaches 0 and the packet is dropped. It is used by traceroute to map routes.
- ICMP is used for diagnostics: Echo Request (type 8) / Echo Reply (type 0) = ping; type 11 = TTL expired (traceroute).
- Layer 4 (Transport) handles end-to-end communication — protocols: TCP (reliable) and UDP (fast, unreliable).
- TCP requires a 3-way handshake (SYN → SYN-ACK → ACK) before data transfer — critical for understanding port scanning.
- SYN scans (half-open) never complete the handshake — faster and less likely to be logged.
- TCP flags: SYN (connection), ACK (acknowledgment), FIN (closure), RST (reset) — each plays a role in scan types.
- Ports 0–1023 are well-known (IANA), 1024–49151 registered, 49152–65535 ephemeral.
- UDP is connectionless — faster but unreliable. Used by DNS (53), DHCP (67/68), SNMP (161).
- CIDR notation (/24, /16) defines network size — essential for pentest scope and host discovery.
- A /16 perimeter = up to 65,534 potential hosts to discover — always start with host discovery before port scanning.

---


## Firewall Detection & IDS Evasion

This lesson covers how to detect firewalls and IDS/IPS systems during a scan, and the techniques Nmap provides to evade or bypass basic filtering — essential skills for network-based penetration testing.

> ⚠️ **Note**: All commands assume you are operating from a **Kali Linux** attack machine. Replace target IPs with your actual values.

---

### What are Firewalls and IDS/IPS?

| System | Full Name | Purpose |
|--------|-----------|---------|
| **Firewall** | — | Filters network traffic based on rules — blocks or allows packets by port, IP, or protocol |
| **IDS** | Intrusion Detection System | Monitors traffic and **alerts** on suspicious activity — does not block |
| **IPS** | Intrusion Prevention System | Monitors traffic and **actively blocks** suspicious activity |

Understanding how these systems work is critical — not just to evade them, but to **test whether they are functioning correctly** as part of a penetration test.

---

### Firewall Detection

Before attempting evasion, identify whether a firewall is present and what it is filtering.

#### Method 1: Port State Analysis

Nmap reports three possible states for a port:

| Port State | Meaning | Firewall Present? |
|-----------|---------|------------------|
| **Open** | Service is listening and responding | No (or port is allowed) |
| **Closed** | Host responds with RST — port not in use | No firewall on this port |
| **Filtered** | No response or ICMP unreachable received | **Yes — firewall is blocking** |

```bash
# A filtered result strongly suggests a firewall is in place
nmap -sS -p 80,443,22,3389 <target_ip>
```

If you see **`filtered`** on ports that should be open, a firewall is blocking the probes.

### Method 2: ACK Scan (`-sA`)

The **ACK scan** is specifically designed to map firewall rules — it does not detect open ports, it detects **whether ports are filtered**.

```bash
nmap -sA -p 80,443,22 <target_ip>
```

How it works:
- Sends a TCP packet with only the **ACK flag** set.
- An **unfiltered** port (with or without a firewall allowing the packet) responds with **RST**.
- A **filtered** port (firewall blocking it) produces **no response** or an ICMP unreachable.

| Response | Interpretation |
|----------|---------------|
| **RST received** | Port is unfiltered — firewall allows ACK packets through |
| **No response / ICMP unreachable** | Port is filtered — firewall is blocking |

> The ACK scan is the best way to distinguish between **stateful** and **stateless** firewalls — stateful firewalls drop unexpected ACK packets; stateless ones may let them through.

#### Method 3: Window Scan (`-sW`)

Similar to the ACK scan but analyzes the **TCP window size** in RST responses to differentiate open from closed ports through certain firewall types.

```bash
nmap -sW -p 80,443 <target_ip>
```

#### Method 4: Firewall vs No Firewall — Quick Test

```bash
# Compare SYN scan vs ACK scan on the same ports
nmap -sS -p 80 <target_ip>   # Port state from SYN scan
nmap -sA -p 80 <target_ip>   # Filtered/unfiltered from ACK scan
```

If SYN shows **filtered** but ACK shows **unfiltered** → stateful firewall is dropping SYN packets but not ACK packets.

---

## IDS/IPS Evasion Techniques

Once you've identified filtering, use these techniques to make scans harder to detect or block. These are also used to **test whether your IDS/IPS detects them**.

#### 1. Packet Fragmentation (`-f`, `--mtu`)

**Packet fragmentation** splits Nmap probes into smaller IP fragments. Some older IDS/firewall systems struggle to reassemble and inspect fragmented packets, allowing them through.

```bash
# Fragment into 8-byte chunks (after IP header)
nmap -sS -f <target_ip>

# Fragment into 16-byte chunks (double -f)
nmap -sS -ff <target_ip>

# Set custom fragment size (must be multiple of 8)
nmap -sS --mtu 24 <target_ip>
```

> Modern stateful firewalls and IDS/IPS systems **can** reassemble fragments — fragmentation is most effective against older or misconfigured systems.

#### 2. Decoy Scanning (`-D`)

**Decoys** mix fake source IPs with the real scan, making it appear that multiple hosts are scanning simultaneously. The target and any monitoring system see traffic from many IPs, making it harder to identify the real attacker.

```bash
# Specify decoy IPs manually (ME = your real IP)
nmap -sS -D 10.10.10.10,10.10.10.11,ME <target_ip>

# Use random decoys (RND generates random IPs)
nmap -sS -D RND:5 <target_ip>
```

> Decoys work best when the IDS is configured to block individual IPs based on scan volume. With decoys, the real IP is hidden among many fake ones.

#### 3. Source Port Manipulation (`-g` / `--source-port`)

Some firewalls allow traffic from **trusted source ports** (e.g., DNS port 53, HTTP port 80) without deep inspection. By spoofing the source port, packets may bypass simple port-based rules.

```bash
# Make scan traffic appear to come from port 53 (DNS)
nmap -sS -g 53 <target_ip>
nmap -sS --source-port 53 <target_ip>

# Appear to come from port 80 (HTTP)
nmap -sS -g 80 <target_ip>
```

> This technique exploits **misconfigured firewalls** that trust traffic based on source port alone rather than full packet inspection.

#### 4. Random Data Length (`--data-length`)

Appending random bytes to packets makes probes less consistent and harder for signature-based IDS systems to match against known scan patterns.

```bash
nmap -sS --data-length 25 <target_ip>
```

#### 5. Disable DNS Resolution (`-n`)

By default, Nmap resolves hostnames — each DNS query is a potential detection point. Disabling DNS resolution makes scans faster and leaves fewer traces.

```bash
nmap -sS -n <target_ip>
```

#### 6. Spoof Source IP (`-S`)

Spoofing the source IP makes traffic appear to come from a different host. However, **replies go back to the spoofed IP**, so you won't receive them. Useful for testing IDS alert rules.

```bash
nmap -sS -S 192.168.1.50 -e eth0 <target_ip>
```

> `-e` specifies the network interface to send from. Without receiving replies, you cannot determine open ports — this is primarily for **IDS rule testing**.

#### 7. MAC Address Spoofing (`--spoof-mac`)

Spoofs the source MAC address in Ethernet frames — useful on local networks where MAC-based filtering is in place.

```bash
# Spoof a random MAC address
nmap -sS --spoof-mac 0 <target_ip>

# Spoof a specific vendor MAC (Apple in this case)
nmap -sS --spoof-mac Apple <target_ip>

# Spoof a specific MAC address
nmap -sS --spoof-mac 00:11:22:33:44:55 <target_ip>
```

---

### Timing-Based Evasion

IDS systems often detect scans based on the **volume and rate** of packets. Slowing down a scan reduces the signature footprint.

#### Timing Templates (`-T`)

| Template | Name | Speed | Stealth | Use Case |
|----------|------|-------|---------|---------|
| `-T0` | Paranoid | Extremely slow | Maximum | Evade most IDS — very long waits between probes |
| `-T1` | Sneaky | Very slow | High | Stealth scans — minutes between probes |
| `-T2` | Polite | Slow | Medium | Reduce load on target network |
| `-T3` | Normal | Default | Default | Balanced — standard starting point |
| `-T4` | Aggressive | Fast | Low | Fast scans on reliable networks |
| `-T5` | Insane | Very fast | None | Maximum speed — high packet loss risk |

```bash
# Slow stealthy scan — evades many IDS rate thresholds
nmap -sS -T1 <target_ip>

# Standard scan
nmap -sS -T3 <target_ip>

# Fast scan for internal reliable networks
nmap -sS -T4 <target_ip>
```

#### Manual Timing Controls

```bash
# Add delay between probes to the same host
nmap -sS --scan-delay 200ms <target_ip>

# Set minimum packet rate (packets per second)
nmap --min-rate 50 <target_ip>

# Set maximum time per host before giving up
nmap --host-timeout 5m <target_ip>
```

---

### Combined Evasion Workflow

A practical evasion-focused scan combining multiple techniques:

```bash
# Fragment + decoys + source port spoof + no DNS + slow timing
nmap -sS -Pn -f -D RND:5 -g 53 -n -T2 --data-length 15 -oX evasion_scan.xml <target_ip>
```

Flag breakdown:
- `-sS` — SYN stealth scan
- `-Pn` — skip host discovery (assume host is up)
- `-f` — fragment packets
- `-D RND:5` — 5 random decoy IPs
- `-g 53` — source port 53 (DNS)
- `-n` — no DNS resolution
- `-T2` — polite timing
- `--data-length 15` — append 15 random bytes
- `-oX` — save results as XML

---

### Evasion Techniques Summary

| Technique | Nmap Flag | Evades |
|-----------|-----------|-------|
| **Packet fragmentation** | `-f`, `--mtu` | Simple packet filters, older IDS |
| **Decoy scanning** | `-D RND:n` | IP-based detection, source tracking |
| **Source port spoof** | `-g <port>` | Port-based firewall rules |
| **Random data length** | `--data-length` | Signature-based IDS |
| **Disable DNS** | `-n` | DNS-based detection |
| **Source IP spoof** | `-S` | IDS alert rule testing |
| **MAC spoof** | `--spoof-mac` | MAC-based filtering (local networks) |
| **Slow timing** | `-T0`, `-T1`, `--scan-delay` | Rate-based IDS thresholds |
| **Skip host discovery** | `-Pn` | ICMP-blocking firewalls |

---

### Key Takeaways

- **Firewalls** filter traffic by port/IP/protocol — detected by **filtered** port states in Nmap output.
- **IDS** alerts on suspicious traffic; **IPS** actively blocks it — both analyze packet patterns and rates.
- The **ACK scan** (`-sA`) is the best tool for mapping firewall rules — RST = unfiltered, no response = filtered.
- **Filtered** ports in a SYN scan = firewall is blocking those probes.
- **Stateful firewalls** track connection state — they drop unexpected ACK packets. **Stateless firewalls** may let them through.
- **Packet fragmentation** (`-f`) splits probes into 8-byte chunks — bypasses some older filters but not modern stateful firewalls.
- **Decoys** (`-D RND:5`) hide the real scanner among fake source IPs — harder to trace back to the attacker.
- **Source port spoofing** (`-g 53`) exploits misconfigured firewall rules that trust traffic from certain ports.
- **Slow timing** (`-T0`, `-T1`, `--scan-delay`) evades rate-based IDS detection by reducing packet volume per second.
- **`-Pn`** skips ICMP host discovery — essential when the target blocks pings but has open ports.
- **Never use `-T5`** in real assessments — too aggressive, causes packet loss and inaccurate results.
- Combine techniques for maximum evasion: `-f -D RND:5 -g 53 -n -T2 --data-length 15`.
- Always **save scan results** with `-oX` (XML) for Metasploit import or `-oN` (text) for notes.
