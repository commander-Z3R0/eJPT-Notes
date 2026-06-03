# Networking Primer

## 1. OSI Model (7 Layers)

The OSI (Open Systems Interconnection) model is a conceptual framework that describes how data moves across a network. It has **7 layers**:

| Layer | Name           | PDU        | Main Function                                      | Example Protocols/Devices          |
|-------|----------------|------------|----------------------------------------------------|------------------------------------|
| 7     | Application    | Data       | User-facing services, network applications         | HTTP, FTP, DNS, SMTP               |
| 6     | Presentation   | Data       | Data formatting, encryption, compression           | SSL/TLS, JPEG, ASCII               |
| 5     | Session        | Data       | Manages sessions between applications              | NetBIOS, RPC                       |
| 4     | Transport      | Segment    | End-to-end communication, reliability, flow control| TCP, UDP                           |
| 3     | Network        | Packet     | Routing, logical addressing (IP)                   | IP, ICMP, routers                  |
| 2     | Data Link      | Frame      | MAC addressing, error detection, switching         | Ethernet, switches, MAC            |
| 1     | Physical       | Bits       | Physical transmission (cables, signals)            | Cables, hubs, NICs, radio signals  |

Data flows **down** the layers on the sender (7 → 1) and **up** on the receiver (1 → 7).

---

## 2. IP (Internet Protocol)

IP is a **network layer (Layer 3)** protocol responsible for routing packets from source to destination using IP addresses.

### Packet Structure (General)

An IP packet consists of:

- **IP Header** – control information (source/destination IP, TTL, protocol, etc.)
- **Payload** – the data carried by the IP packet (e.g., TCP segment, UDP datagram)

```text
+---------------------+----------------------+
|      IP Header      |      Payload         |
| (20–60 bytes)       | (TCP/UDP + data)     |
+---------------------+----------------------+
```

The payload typically contains a **transport layer segment** (TCP or UDP) plus application data.

---

## 3. TCP vs UDP

### TCP (Transmission Control Protocol)

- **Layer**: Transport (Layer 4)
- **Connection-oriented**: establishes a connection before sending data
- **Reliable**: guarantees delivery, ordering, and retransmission
- **Flow control & congestion control**
- Used by: HTTP, HTTPS, SSH, FTP, SMTP

### UDP (User Datagram Protocol)

- **Layer**: Transport (Layer 4)
- **Connectionless**: no handshake, just sends packets
- **Unreliable**: no guarantee of delivery or order
- **Faster**, lower overhead
- Used by: DNS, DHCP, VoIP, streaming, SNMP

---

## 4. IPv4 Header in Detail

An IPv4 datagram has a **variable-length header** (minimum 20 bytes, up to 60 bytes with options).

### IPv4 Header Fields

| Field              | Size (bits) | Description                                                                 |
|--------------------|-------------|-----------------------------------------------------------------------------|
| Version            | 4           | IP version; for IPv4 this is `4`                                           |
| IHL (Header Length)| 4           | Header length in 32-bit words; min 5 (20 bytes), max 15 (60 bytes)         |
| Type of Service    | 8           | QoS: priority, delay, throughput, reliability                              |
| Total Length       | 16          | Total length of header + data (min 20 bytes, max 65,535 bytes)             |
| Identification     | 16          | Unique ID for fragmentation/reassembly                                     |
| Flags              | 3           | Reserved (0), Do Not Fragment (DF), More Fragments (MF)                    |
| Fragment Offset    | 13          | Offset of this fragment in 8-byte units                                    |
| TTL (Time to Live) | 8           | Max hops before packet is dropped                                          |
| Protocol           | 8           | Next protocol: `1=ICMP`, `6=TCP`, `17=UDP`                                 |
| Header Checksum    | 16          | Checksum for header only (not payload)                                     |
| Source IP Address  | 32          | Sender's IP address                                                        |
| Destination IP     | 32          | Receiver's IP address                                                      |
| Options (optional) | variable    | Extra features (e.g., source routing, record route) + padding              |

### IPv4 Packet Layout

```text
+------------------+-------------------+
|    IPv4 Header   |    Payload        |
| (20–60 bytes)    | (TCP/UDP + data)  |
+------------------+-------------------+
```

The **Protocol** field tells the receiver what is inside the payload:
- `1` → ICMP
- `6` → TCP
- `17` → UDP

---

## 5. TCP 3-Way Handshake

TCP is **connection-oriented**. Before data transfer, a **3-way handshake** establishes a reliable connection:

### Steps

1. **SYN**  
   - Client sends a segment with `SYN=1`, `ACK=0`.
   - Chooses an **Initial Sequence Number (ISN)**, e.g. `Seq = 1001`.

2. **SYN-ACK**  
   - Server responds with `SYN=1`, `ACK=1`.
   - Server chooses its own ISN, e.g. `Seq = 2001`.
   - Acknowledges client's ISN: `Ack = 1002` (client ISN + 1).

3. **ACK**  
   - Client sends `ACK=1`, `SYN=0`.
   - `Seq = 1002` (client ISN + 1).
   - `Ack = 2002` (server ISN + 1).

After this, the connection is **established** and data transfer begins.

```text
Client                      Server
  | SYN (Seq=1001)            |
  |-------------------------> |
  |                           |
  |<------------------------- |
  | SYN-ACK (Seq=2001, Ack=1002)
  |                           |
  | SYN-ACK (Seq=1002, Ack=2002)
  |-------------------------> |
  |                           |
  |   Connection established  |
```

---

## 6. TCP Ports and Ranges

TCP uses **16-bit port numbers** (0–65535) to identify applications/services.

### Port Ranges

| Range            | Name                | Description                              |
|------------------|---------------------|------------------------------------------|
| 0–1023           | **Well-known**      | System/privileged services (root)        |
| 1024–49151       | **Registered**      | User applications, common services       |
| 49152–65535      | **Dynamic/Private** | Ephemeral ports for client connections   |

### Common Well-Known Ports

| Port | Protocol | Service        |
|------|----------|----------------|
| 20   | TCP      | FTP (data)     |
| 21   | TCP      | FTP (control)  |
| 22   | TCP      | SSH            |
| 23   | TCP      | Telnet         |
| 25   | TCP      | SMTP           |
| 53   | TCP/UDP  | DNS            |
| 80   | TCP      | HTTP           |
| 443  | TCP      | HTTPS          |
| 3389 | TCP      | RDP            |

Clients typically use **ephemeral ports** (dynamic range) as source ports when connecting to servers.

---

## 7. Quick Summary

- **OSI model**: 7 layers from Physical (1) to Application (7).
- **IP (Layer 3)**: routes packets using source/destination IP addresses.
- **IPv4 header**: contains version, length, TTL, protocol, checksum, IPs, etc.
- **TCP**: reliable, connection-oriented, uses 3-way handshake.
- **UDP**: fast, connectionless, no guarantee of delivery.
- **Ports**: identify services; well-known (0–1023), registered (1024–49151), dynamic (49152–65535).
