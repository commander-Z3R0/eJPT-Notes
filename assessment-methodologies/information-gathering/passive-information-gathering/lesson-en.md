# Reconnaissance

## Passive Reconnaissance

Passive information gathering involves collecting as much information as possible without actively engaging with the target system.

### What to Collect
- IP addresses and DNS information.
- Domain names and ownership information.
- Email addresses and social media profiles.
- Web technologies used on target sites.
- Subdomains.

### Host Command
The `host` command is a simple DNS lookup utility in Unix/Linux systems. You can use it to find the IP address associated with a domain, identify mail servers, and more.

```bash
host -a <domain>
```

If more than one IP address is returned, the target may be using a proxy or a CDN such as Cloudflare.

### robots.txt
Once you access a website, check the `/robots.txt` file. It tells web crawlers which parts of the site should not be indexed and may reveal hidden or sensitive paths.

### sitemap.xml
Check the `/sitemap.xml` file to see all pages indexed for the website. It can help you understand the site structure faster.

### Web Technologies
Use tools such as:
- `BuiltWith`
- `Wappalyzer`
- `whatweb`

These tools help identify the technologies used by a target site.

```bash
whatweb <url>
```

### HTTrack
Use `httrack` to download an entire website and inspect its directories and files locally.

```bash
httrack <url>
```

### Whois Enumeration
`whois` is a protocol used to query databases that store registration information about domain names, IP address blocks, and autonomous systems.

```bash
whois <domain>
whois <ip>
```

You can also use:
- [who.is](https://who.is/)
- [whois.com](https://www.whois.com/)

### Netcraft
[Netcraft](https://sitereport.netcraft.com/) is useful for passive website reconnaissance. It can reveal technologies, certificates, and server-side information.

### DNS Recon
`dnsrecon` is a DNS enumeration tool that can discover:
- `A`, `AAAA`, `NS`, `MX`, `SOA`, `TXT`, and `SPF` records.
- Zone transfers.
- Reverse lookups.
- Subdomains.

```bash
dnsrecon -d <domain>
```

Another useful tool is `dnsdumpster`, which provides a visual DNS map.

### Wafw00f
A WAF, or Web Application Firewall, protects web applications by filtering malicious traffic. `wafw00f` identifies which WAF a website is using.

```bash
wafw00f <url>
```

### Sublist3r
`Sublist3r` enumerates subdomains using OSINT sources. Do not use the brute-force option if you want to stay in passive reconnaissance.

```bash
sublist3r -d <domain>
```

### Google Dorks
Google dorks use specific search operators to find targeted information.

Examples:
- `site:<domain>`
- `filetype:pdf`
- `inurl:index.php`
- `intitle:index of`
- `cache:<url>`
- `related:<url>`

Useful resources:
- [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)
- [Wayback Machine](https://archive.org/)

### TheHarvester
`theHarvester` is designed for OSINT. It gathers emails, names, subdomains, IPs, hosts, and URLs from public sources.

```bash
theHarvester -d <domain> -b all
```

Useful tools and services:
- [Phonebook](https://phonebook.cz/)
- [Censys](https://search.censys.io/)
- [Shodan](https://www.shodan.io/)

### Leaked Databases
Use [haveibeenpwned.com](https://haveibeenpwned.com/) to check whether an email address has been exposed in a data breach.

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
