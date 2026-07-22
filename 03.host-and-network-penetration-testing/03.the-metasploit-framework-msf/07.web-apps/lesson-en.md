# Web Apps

## Overview

**WMAP** is a web application vulnerability scanner integrated into Metasploit. It is designed to automate web server enumeration and scan web applications for vulnerabilities from inside `msfconsole`.

WMAP is useful because it keeps web scanning inside the Metasploit workflow, so discovered sites, paths, and vulnerabilities can be stored and reused in the framework database.

## Lab Environment

For practice, the course uses an intentionally vulnerable target such as **Metasploitable3**. In the lab, you will typically scan a web service running on `<target_ip>` or a web application reachable on `<target_network>/24`.

## Setup

Before using WMAP, make sure Metasploit and the database are running.

```bash
sudo systemctl start postgresql
msfdb init
msfconsole
```

Check the database connection:

```bash
db_status
```

Create or switch to a workspace:

```bash
workspace -a webapp
workspace webapp
```

## Load WMAP

Inside `msfconsole`, load the plugin with:

```bash
load wmap
```

If the plugin loads correctly, WMAP commands become available in the console.

## Add a Site

Before scanning, add the target web application as a site.

Add a single target:

```bash
wmap_sites -a <target_ip>
```

Or add a target by URL:

```bash
wmap_targets -t http://<target_ip>
```

For a network-hosted web app, use:

```bash
wmap_targets -t http://<target_network>/24
```

List configured sites:

```bash
wmap_sites -l
```

List configured targets:

```bash
wmap_targets -l
```

## Run WMAP Scans

WMAP can map the site and then run enabled scanners against it.

List available scan modules:

```bash
wmap_run -t
```

Run all enabled modules:

```bash
wmap_run -e
```

Depending on the size of the application, this can take some time.

## Review Results

After the scan finishes, review the discovered vulnerabilities and paths.

```bash
wmap_vulns -l
```

You can also review the general Metasploit database data with:

```bash
hosts
services
vulns
```

## Useful Workflow

1. Start PostgreSQL and Metasploit.
2. Check `db_status`.
3. Create a workspace.
4. Load WMAP with `load wmap`.
5. Add the site with `wmap_sites -a <target_ip>`.
6. Define the target with `wmap_targets -t http://<target_ip>`.
7. List targets and sites to confirm.
8. Run `wmap_run -t` and then `wmap_run -e`.
9. Check discovered issues with `wmap_vulns -l`.
10. Review `hosts`, `services`, and `vulns` in the database.

## Why This Matters

WMAP is useful because it automates web application reconnaissance and vulnerability scanning from within Metasploit. That makes it easier to connect discovery with later exploitation, especially when you want everything organized in one place.

## Key Idea

WMAP is Metasploit’s built-in web application scanner. It helps you map a site, scan it for vulnerabilities, and store the results in the Metasploit database for later use.
