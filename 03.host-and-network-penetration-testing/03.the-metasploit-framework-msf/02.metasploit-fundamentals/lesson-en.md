# The Metasploit Framework

## Installing and Configuring

Before using Metasploit, you need to install it and make sure the database is working correctly. Metasploit is distributed by Rapid7 and can be installed on both Windows and Linux, but in this course we use Kali Linux because Metasploit and its dependencies are already included.

Metasploit uses a database to store hosts, scans, services, and other assessment data. PostgreSQL is the database backend, so the PostgreSQL service must be running before you initialize Metasploit’s database.

### Installation Steps

```bash
sudo apt update && sudo apt upgrade -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
msfdb init
msfconsole
```

You can verify the database connection inside `msfconsole` with:

```bash
db_status
```

If everything is configured correctly, Metasploit should show that PostgreSQL is connected. The database also lets you import scan results from tools like Nmap and Nessus. 

## MSFconsole Fundamentals

`msfconsole` is the main all-in-one interface for Metasploit. It is the primary tool you will use for the rest of the course because it gives access to all framework functionality from a single console. 

### What You Need To Know

- How to search for modules.
- How to select modules.
- How to configure module options and variables.
- How to search for payloads.
- How to manage sessions.
- How to use additional functionality.
- How to save your configuration.

### Useful Commands

Search for a module:

```bash
search <keyword>
```

Select a module:

```bash
use <module_path>
```

View module options:

```bash
show options
```

Show payloads:

```bash
show payloads
```

Run a module:

```bash
run
```

or

```bash
exploit
```

### Module Variables

Many modules require values such as target IPs, ports, and callback addresses. These are set using variables in `msfconsole`. 

| Variable | Purpose |
|---|---|
| `LHOST` | Attacker IP address. |
| `LPORT` | Port on the attacker side used for the reverse connection. |
| `RHOST` | Target IP address. |
| `RHOSTS` | Multiple target IPs or a network range. |
| `RPORT` | Target port. |

Example:

```bash
set RHOSTS 10.10.10.10
set LHOST 10.10.14.2
set LPORT 4444
```

You can set values locally for one module or globally for the whole session.

## Creating and Managing Workspaces

Workspaces help you organize hosts, scans, and activities during an assessment. They are useful for separating data by target or by client so your work stays clean and structured. 

### Workspace Commands

List workspaces:

```bash
workspace
```

Create a new workspace:

```bash
workspace -a client1
```

Switch to a workspace:

```bash
workspace client1
```

Delete a workspace:

```bash
workspace -d client1
```

### Why Workspaces Matter

- They keep assessments separated.
- They make it easier to track hosts and scan results.
- They help organize data during longer engagements.

## Key Idea

The first step in Metasploit is getting the environment ready: PostgreSQL must be running, `msfdb` must be initialized, and `msfconsole` is where you manage modules, variables, sessions, and workspaces. Once that is in place, you can start using Metasploit efficiently for the rest of the course.
