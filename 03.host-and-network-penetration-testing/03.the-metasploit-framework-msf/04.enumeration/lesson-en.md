# Enumeration

## Overview

Enumeration is the process of gathering detailed information about a target service after identifying it. In this phase, the goal is to extract useful data such as service versions, users, shares, directories, configuration details, and other information that can help during exploitation or later post-exploitation.

Auxiliary modules are especially useful for enumeration because they support scanning, discovery, fuzzing, and service-specific checks. They can be used during information gathering, after initial access, and even after pivoting into another subnet.

## Port Scanning With Auxiliary Modules

Auxiliary modules can be used to scan TCP and UDP ports, discover hosts, and enumerate services on a target or internal network.

### Useful Commands

Scan a single host:

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_ip>
run
```

Scan a subnet:

```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS <target_network>/24
run
```

UDP scanning example:

```bash
use auxiliary/scanner/discovery/udp_probe
set RHOSTS <target_ip>
run
```

## FTP Enumeration

FTP uses TCP port 21 and is used for file transfer between a client and a server. It is also sometimes used to transfer files to and from a web server directory.

FTP authentication normally requires a username and password, but some servers allow anonymous login if they are misconfigured.

### Useful Commands

Check the FTP version:

```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target_ip>
run
```

Check for anonymous login:

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target_ip>
run
```

Brute-force FTP credentials:

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## SMB Enumeration

SMB is a file-sharing protocol used on local networks. It normally runs on TCP port 445, although it originally also used port 139 over NetBIOS. SAMBA is the Linux implementation of SMB.

With SMB auxiliary modules, you can enumerate the SMB version, shares, and users, and also perform brute-force attacks.

### Useful Commands

Check SMB version:

```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS <target_ip>
run
```

Enumerate shares:

```bash
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target_ip>
run
```

Enumerate users:

```bash
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS <target_ip>
run
```

Brute-force SMB credentials:

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS <target_ip>
set USER_FILE users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## Web Server Enumeration

A web server is software that serves website data through HTTP. HTTP uses TCP port 80, and popular web servers include Apache, Nginx, and Microsoft IIS.

With auxiliary modules, you can identify the web server version, inspect HTTP headers, and brute-force directories.

### Useful Commands

Check HTTP title:

```bash
use auxiliary/scanner/http/title
set RHOSTS <target_ip>
run
```

Check HTTP headers:

```bash
use auxiliary/scanner/http/http_header
set RHOSTS <target_ip>
run
```

Brute-force directories:

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target_ip>
set PATH /usr/share/wordlists/dirb/common.txt
run
```

## MySQL Enumeration

MySQL is an open-source relational database management system based on SQL. It is commonly used to store application and web data and usually runs on TCP port 3306.

Auxiliary modules can be used to identify the MySQL version, brute-force passwords, and run SQL queries.

### Useful Commands

Check MySQL version:

```bash
use auxiliary/scanner/mysql/mysql_version
set RHOSTS <target_ip>
run
```

Brute-force MySQL credentials:

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS <target_ip>
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## SSH Enumeration

SSH is a remote administration protocol that provides encrypted access and is the successor to Telnet. It usually runs on TCP port 22.

Auxiliary modules can be used to identify the SSH version and brute-force credentials.

### Useful Commands

Check SSH version:

```bash
use auxiliary/scanner/ssh/ssh_version
set RHOSTS <target_ip>
run
```

Brute-force SSH credentials:

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <target_ip>
set USERNAME <user>
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

## SMTP Enumeration

SMTP is the protocol used for sending email. It usually runs on TCP port 25, and it can also run on ports 465 and 587.

Auxiliary modules can be used to identify the SMTP version and enumerate users.

### Useful Commands

Check SMTP version:

```bash
use auxiliary/scanner/smtp/smtp_version
set RHOSTS <target_ip>
run
```

Enumerate users:

```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS <target_ip>
run
```

## Why This Matters

Enumeration helps you collect the details needed to decide which service to attack and how to attack it. Auxiliary modules make this faster because they automate scanning and service-specific checks.

## Key Idea

Enumeration is about turning a discovered service into useful information. Once you identify the service version, users, shares, headers, or credentials, you can move to exploitation with much better context.
