# Host Discovery

Host discovery is the process of identifying which hosts are **alive/up** in a network. Instead of scanning every port of every IP, we first find the active machines to focus our efforts.

## What is Host Discovery?

In penetration testing and network reconnaissance, host discovery (also called **ping scan**) reduces a large IP range to a smaller list of active hosts. Nmap supports many discovery techniques beyond simple ICMP ping :

- Some hosts block ICMP echo requests (classic ping).
- Firewalls may filter certain types of traffic.
- Different techniques work better in different network environments.

---

## Host Discovery Techniques

### 1. Ping Sweeps (ICMP Echo Ping)

A **ping sweep** sends ICMP echo request packets (type 8) to multiple hosts and waits for ICMP echo replies (type 0). If a host replies, it's up.

- **Nmap option**: `-PE`
- **How it works**:
  - Send ICMP Echo Request → if Echo Reply → host is up.
- **Pros**:
  - Simple, widely understood.
  - Fast on internal networks.
- **Cons**:
  - Many hosts and firewalls **block ICMP echo** now.
  - Not reliable on the Internet against unknown targets.

Example:
```bash
nmap -sn -PE 192.168.1.0/24
```

---

### 2. ARP Scanning (ARP Ping)

**ARP scanning** is used on **local Ethernet networks** (LANs). It sends ARP requests asking "Who has IP X?" and waits for ARP replies.

- **Nmap option**: `-PR` (default for local Ethernet)
- **How it works**:
  - Send ARP request: "Who has 192.168.1.5?"
  - If host replies with ARP response → host is up.
- **Pros**:
  - Very **fast** and **accurate** on LANs.
  - Hosts generally **cannot block ARP** and still communicate on the network.
- **Cons**:
  - Only works on **local networks** (same subnet).
  - Cannot be used for remote scanning across routers.

Example:
```bash
nmap -sn -PR 192.168.1.0/24
```

---

### 3. TCP SYN Ping (Half-Open Scan)

**TCP SYN ping** sends a TCP packet with the **SYN flag** set (like the first step of a TCP handshake), but never completes the connection. This is why it's called a **half-open scan**.

- **Nmap option**: `-PS<port>` (e.g. `-PS80`, `-PS22,80,443`)
- **How it works**:
  1. Send TCP SYN packet to a target port.
  2. If port is **closed**: host responds with **RST** → host is up.
  3. If port is **open**: host responds with **SYN/ACK** → host is up.
  4. Nmap then sends **RST** to tear down the connection (never sends ACK).

- **Pros**:
  - Often **bypasses firewalls** that block ICMP.
  - Works well when targeting common services (HTTP port 80, HTTPS 443, SSH 22).
- **Cons**:
  - Requires **raw packet** capability (usually root on Unix).
  - Unprivileged users use `connect()` workaround (less efficient).

Example:
```bash
nmap -sn -PS80,443 192.168.1.0/24
```

This technique is called **half-open** because we never complete the TCP 3-way handshake.

---

### 4. UDP Ping

**UDP ping** sends UDP packets to specific ports. If a port is **closed**, the target typically responds with an **ICMP port unreachable** message, which tells Nmap the host is up .

- **Nmap option**: `-PU<port>` (e.g. `-PU53`, `-PU161`)
- Default port: **40125** (uncommon port).
- **How it works**:
  1. Send UDP packet to a port.
  2. If port is **closed**: receive **ICMP port unreachable** → host is up.
  3. If port is **open**: most services ignore the packet → no response.
  4. Lack of response or certain ICMP errors → host may be down/unreachable.

- **Pros**:
  - Bypasses firewalls that **filter TCP but allow UDP**.
  - Useful when TCP probes are blocked.
- **Cons**:
  - Slower than TCP (UDP is connectionless, many timeouts).
  - Less reliable because many hosts don't send ICMP errors.

Example:
```bash
nmap -sn -PU53,161,40125 192.168.1.0/24
```

---

### 5. TCP ACK Ping

**TCP ACK ping** sends a TCP packet with the **ACK flag** set. This packet looks like it's acknowledging data on an **existing connection**, but no such connection exists.

- **Nmap option**: `-PA<port>` (e.g. `-PA80`, `-PA443`)
- **How it works**:
  1. Send TCP ACK packet to a port.
  2. Host should respond with **RST** (because no connection exists) → host is up.
- **Pros**:
  - Can **bypass stateless firewalls** that block incoming SYN but allow ACK.
  - Useful when SYN ping is filtered.
- **Cons**:
  - Less effective against **stateful firewalls** that drop unexpected ACK packets.
  - Like SYN ping, usually requires root on Unix.

Example:
```bash
nmap -sn -PA80,443 192.168.1.0/24
```

---

## Comparison of Host Discovery Techniques

| Technique            | Packet Type         | Nmap Option | Works Best On                 | Bypasses ICMP Block? |
|----------------------|---------------------|-------------|-------------------------------|----------------------|
| Ping Sweep (ICMP)    | ICMP Echo Request   | `-PE`       | Internal networks             | No                   |
| ARP Scan             | ARP Request         | `-PR`       | Local Ethernet (LAN)          | N/A (no ICMP used)   |
| TCP SYN Ping         | TCP SYN             | `-PS`       | Internet, common services     | Yes                  |
| TCP ACK Ping         | TCP ACK             | `-PA`       | Stateless firewalls           | Yes                  |
| UDP Ping             | UDP                 | `-PU`       | UDP-permissive firewalls      | Yes                  |

---

## Default Host Discovery in Nmap

If you don't specify any ping type, Nmap uses a **default combination**:

- For **privileged users** (root):
  - `-PE -PS443 -PA80 -PP`
  - ICMP echo + TCP SYN to 443 + TCP ACK to 80 + ICMP timestamp.
- For **local Ethernet**:
  - Uses **ARP scan** by default (`-PR`), even if you specify other options.
- For **unprivileged users**:
  - `-PS80,443` using `connect()` system call.

Example of host discovery only (no port scan):
```bash
nmap -sn 192.168.1.0/24
```

The `-sn` option tells Nmap: **only do host discovery, no port scan**.

---

## Key Takeaways

- **Ping sweeps** (ICMP) are simple but often blocked.
- **ARP scanning** is the most reliable on local LANs.
- **TCP SYN ping** is a half-open scan; great for bypassing ICMP filters.
- **UDP ping** is useful when TCP is filtered but UDP is allowed.
- **TCP ACK ping** can bypass stateless firewalls that block SYN.
- In practice, use **multiple techniques** together to maximize host discovery accuracy.
