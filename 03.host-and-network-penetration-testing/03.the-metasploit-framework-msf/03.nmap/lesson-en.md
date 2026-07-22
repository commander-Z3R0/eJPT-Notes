# Nmap

## Overview

**Nmap** is a free and open-source network scanner used to discover hosts on a network, scan targets for open ports, and identify the services and operating systems running on those targets.

It can also export scan results in a format that can be imported into Metasploit for vulnerability detection and exploitation.

## Port Scanning and Enumeration

Nmap is commonly used to:

- Discover live hosts on a network.
- Scan for open ports.
- Identify services running on open ports.
- Detect the operating system of the target.

### Basic Nmap Syntax

```bash
nmap <target_ip>
```

### Common Examples

Scan a single host:

```bash
nmap 192.168.1.10
```

Scan specific ports:

```bash
nmap -p 22,80,443 192.168.1.10
```

Scan for service versions:

```bash
nmap -sV 192.168.1.10
```

Scan for OS detection:

```bash
nmap -O 192.168.1.10
```

## Importing Nmap Results Into MSF

Metasploit can import Nmap scan results and use them to populate its database with hosts and services.

### Export Nmap Results

To import scan results into Metasploit, Nmap output should be saved in XML format:

```bash
nmap -sV -O -oX scan.xml 192.168.1.10
```

### Import Into Metasploit

Inside `msfconsole`, use:

```bash
db_import scan.xml
```

After importing, you can view the discovered hosts and services with:

```bash
hosts
services
```

## Why This Matters

Importing Nmap results into Metasploit saves time and helps organize assessment data. It allows you to reuse scan results for later exploitation and keeps host and service information in one place.

## Key Idea

Nmap is one of the first tools used in a penetration test because it gives a fast view of live hosts, open ports, services, and operating systems. Its XML output can be imported directly into Metasploit to continue the assessment efficiently.
