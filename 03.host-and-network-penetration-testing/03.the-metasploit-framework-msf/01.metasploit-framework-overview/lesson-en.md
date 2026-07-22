# The Metasploit Framework

## Overview

The **Metasploit Framework (MSF)** is an open-source penetration testing and exploitation framework used by penetration testers and security researchers. It provides the structure needed to automate many stages of the penetration testing lifecycle, from reconnaissance to post-exploitation.

It is also used to develop and test exploits, and it includes a very large database of public, tested exploits. Its modular design makes it easy to add new functionality.

## History

- Developed by **HD Moore** in **2003**.
- Originally written in **Perl**.
- Rewritten in **Ruby** in **2007**.
- Acquired by **Rapid7** in **2009**.
- **Metasploit 5.0** was released in **2019**.
- **Metasploit 6.0** was released in **2020**.

## Editions

- **Metasploit Pro** — commercial.
- **Metasploit Express** — commercial.
- **Metasploit Framework** — community edition.

## Essential Terminology

| Term | Meaning |
|---|---|
| **Interface** | A way to interact with Metasploit. |
| **Module** | A piece of code that performs a specific task, such as an exploit. |
| **Vulnerability** | A weakness or flaw in a system or network that can be exploited. |
| **Exploit** | A module or piece of code used to take advantage of a vulnerability. |
| **Payload** | Code delivered to the target system by an exploit, usually to run commands or provide remote access. |
| **Listener** | A utility that waits for an incoming connection from the target. |

## Main Interfaces

### MSFconsole

The **Metasploit Framework Console (MSFconsole)** is an easy-to-use all-in-one interface that gives access to all Metasploit functionality.

### MSFcli

The **Metasploit Framework Command Line Interface (MSFcli)** was a command-line utility used to help create automation scripts with Metasploit modules. It could redirect output between other tools and Metasploit. This interface was discontinued in 2015, but similar functionality is available through MSFconsole.

### Metasploit Community Edition

**Metasploit Community Edition** is a web-based GUI front-end that simplifies network discovery and vulnerability identification.

### Armitage

**Armitage** is a free Java-based GUI front-end for Metasploit that simplifies network discovery, exploitation, and post-exploitation.

## Architecture

Metasploit is modular, and each module performs a specific task. The framework libraries make it possible to run modules without writing the underlying execution code manually.

## Module Types

- **Exploit** — used to take advantage of a vulnerability, usually paired with a payload.
- **Payload** — code delivered and executed on the target after successful exploitation. A common example is a reverse shell that connects back to the attacker.
- **Encoder** — used to encode payloads to help avoid antivirus detection.
- **NOPs** — used to keep payload sizes consistent and improve execution stability.
- **Auxiliary** — used for tasks such as port scanning and enumeration.
- **Post** — used after initial access for tasks such as privilege escalation, credential gathering, and persistence.

## Payload Types

Metasploit supports two main payload types:

- **Non-staged payloads** are sent to the target as a single unit along with the exploit.
- **Staged payloads** are delivered in two parts:
  - The **stager** establishes the reverse connection and downloads the second part.
  - The **stage** is the payload component downloaded and executed by the stager.

## Meterpreter

**Meterpreter** is an advanced multi-function payload that runs in memory on the target system, which makes it harder to detect. It communicates through a stager socket and provides an interactive command interpreter for tasks like command execution, file navigation, keylogging, and more.

## File System Structure

The Metasploit file system is organized in a simple directory structure.

On Linux, Metasploit modules are stored in:

```bash
/usr/share/metasploit-framework/modules
```

User-defined modules are stored in:

```bash
~/.ms4/modules
```

## Penetration Testing Workflow

Metasploit can be used to perform and automate tasks across the penetration testing lifecycle. A useful way to understand its role is through the PTES methodology.

The main phases are:

- Information gathering and enumeration.
- Vulnerability scanning.
- Exploitation.
- Post-exploitation.
- Privilege escalation.
- Maintaining persistent access.

## Key Idea

Metasploit is a modular and flexible framework that helps automate penetration testing from start to finish. Its strength lies in its structure, its module library, and its ability to support exploitation and post-exploitation in one place.
