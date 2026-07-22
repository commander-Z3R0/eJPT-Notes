# MSF

## Overview

Vulnerability scanning is the process of checking a target for known vulnerabilities and verifying whether they can be exploited. In this stage, Metasploit is used with auxiliary and exploit modules to identify weaknesses in services, operating systems, and web applications.

This phase is important because it helps you narrow down which vulnerabilities are worth testing during exploitation. It also helps you organize scan results inside Metasploit so you can reuse them later.

## Lab Environment

For this section, the lab uses **Metasploitable3**, an intentionally vulnerable virtual machine based on Windows Server 2008. It was created by Rapid7 to demonstrate how Metasploit can be used against a Windows system.

## Workspace Management

Workspaces are useful for separating scan data by target or by network.

### Useful Commands

List workspaces:

```bash
workspace
```

Create a workspace:

```bash
workspace -a lab1
```

Switch to a workspace:

```bash
workspace lab1
```

Delete a workspace:

```bash
workspace -d lab1
```

## Scanning and Analysis

Before scanning, make sure the database is active and your workspace is selected.

### Check Database Status

```bash
db_status
```

### View Hosts

```bash
hosts
```

### View Services

```bash
services
```

### View Vulnerabilities

```bash
vulns
```

## Vulnerability Scanning Commands

Metasploit can scan a single target or an entire subnet.

Scan a single host:

```bash
set RHOSTS <target_ip>
```

Scan a subnet:

```bash
set RHOSTS <target_network>/24
```

### Search for Relevant Modules

Search by service or vulnerability name:

```bash
search <keyword>
```

Examples:

```bash
search smb
search http
search mysql
search ftp
```

### Check a Module

Some exploit modules support a safe check to verify whether a target is likely vulnerable:

```bash
check
```

### Run a Module

```bash
run
```

or

```bash
exploit
```

## Importing Scan Results

Metasploit can import scan results from external tools such as Nmap and Nessus.

Import an Nmap XML scan:

```bash
db_import scan.xml
```

After importing, review the results:

```bash
hosts
services
vulns
```

## Searchsploit

`searchsploit` is useful for looking up public exploits after identifying a vulnerable service or version.

Search by software name:

```bash
searchsploit <software_name>
```

Search by version:

```bash
searchsploit <software_name> <version>
```

Copy an exploit locally if needed:

```bash
searchsploit -m <exploit_id>
```

## Metasploit Autopwn

`metasploit-autopwn` was an old automation approach used to match services with possible exploits. It is no longer considered a standard workflow, but it is still mentioned in some training material as a historical reference.

## Nessus Integration

Nessus results can also be imported into Metasploit if you have a compatible scan export. This helps centralize vulnerability data inside the Metasploit database.

## Useful Workflow

1. Select or create a workspace.
2. Verify the database with `db_status`.
3. Import scan results or scan the target directly.
4. Review `hosts`, `services`, and `vulns`.
5. Search for modules with `search`.
6. Confirm vulnerability status with `check`.
7. Use `searchsploit` to compare findings with public exploits.

## Key Idea

Vulnerability scanning with Metasploit is about identifying likely weaknesses and organizing the results efficiently. Workspaces, the database, imported scans, and module checks make it easier to move from discovery to exploitation.
