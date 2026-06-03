## Active Reconnaissance

Active information gathering means interacting with the target system, which requires authorization.

### DNS Zone Transfers
DNS (Domain Name System) resolves domain names and hostnames to IP addresses. DNS servers store records for most domains on the internet.

Common DNS record types:
- `A` — Hostname or domain to an IPv4 address.
- `AAAA` — Hostname or domain to an IPv6 address.
- `NS` — Domain nameserver.
- `MX` — Domain to a mail server.
- `CNAME` — Used for domain aliases.
- `TXT` — Text record.
- `HINFO` — Host information.
- `SOA` — Domain authority.
- `SRV` — Service records.
- `PTR` — IP address to a hostname.

DNS interrogation is the process of enumerating DNS records for a specific domain. Zone transfers can be abused if DNS servers are misconfigured.

You can perform a zone transfer with:

```bash
dig axfr @<dns-server> <domain>
dnsenum <domain>
```

`Fierce` is another useful tool often used before Nmap.

### Nmap
To find your own IP address, use:

```bash
ip a
```

For host discovery:

```bash
nmap -sn <subnet>
netdiscover
```

Remember to use `sudo` when needed.

## Nmap Switches

| Switch | Use case |
|---|---|
| `-v` | Verbose, shows what’s happening in real time. |
| `-sV` | Finds the version of a service running on a port. |
| `-O` | Detects what operating system is running. |
| `-A` | Enables all detection (OS, version, scripts, etc.). |
| `-T0` to `-T5` | Sets scan intensity from low to high speed. |
| `-sC` | Scans with default NSE scripts. |
| `-F` | Uses faster scanning with a smaller set of common ports. |
| `-p` | Scans all or specific ports. |
| `-Pn` | Disables host discovery and only scans ports. |
| `-sn` | Disables port scanning and only does host discovery. |
| `-sU` | UDP port scan. |
| `-sS` | TCP SYN port scan. |
