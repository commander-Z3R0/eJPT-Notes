# Evasion, Scan Performance, and Output

Nmap is not only a port scanner. It also provides options to **evade basic filtering**, **control scan speed**, and **save results in different output formats** for later analysis.

---

## Evasion

**Evasion** refers to techniques used to make scans harder to detect or block by firewalls, IDS, or IPS systems. In penetration testing, the goal is not “hacking faster,” but understanding how defenses react to different packet shapes and scan patterns.

Nmap includes several evasion-related options that can help when a target network is filtering or inspecting traffic:

- **Fragmentation** breaks probes into smaller IP fragments.
- **Decoys** hide the real source by mixing fake source IPs into the scan.
- **Custom packet size or payload** can change how a probe looks on the wire.
- **Source spoofing** can make traffic appear to come from another address, although replies usually will not return correctly.

These techniques are useful in controlled testing because they help you evaluate whether a network can detect or block unusual scan traffic.

---

## Packet Fragmentation

**Packet fragmentation** splits Nmap probes into smaller IP fragments so that some packet filters or IDS tools may have more difficulty inspecting the full TCP/UDP header at once.

### `-f`
The `-f` flag tells Nmap to fragment packets into **tiny pieces**. Nmap splits the packet into fragments of **8 bytes or less after the IP header**. If you use `-f` twice, Nmap increases the fragment size to **16 bytes**.

Example:
```bash
nmap -sS -f <target_ip>
nmap -sS -ff <target_ip>
```

### `--mtu`
The `--mtu` option lets you choose the fragment size manually. The value must be a **multiple of 8**, and you should not combine it with `-f` .

Example:
```bash
nmap -sS --mtu 24 <target_ip>
```

### `--data-length`
The `--data-length` option appends random bytes to sent packets. This makes probes less minimal and can slightly change how they appear to monitoring tools.

Example:
```bash
nmap -sS --data-length 16 <target_ip>
```

### `-D`
The `-D` option uses **decoys**. Nmap makes the target see several source IPs scanning at the same time, so the real scanner is harder to identify.

Example:
```bash
nmap -sS -D 10.10.10.10,10.10.10.11,ME <target_ip>
nmap -sS -D RND:5 <target_ip>
```

### When to use these options
Use these features when you want to test whether a firewall or IDS can detect unusual scan traffic, or when you need to understand how defensive controls behave under non-standard probes. They can also slow scans down and may reduce reliability, so they are not always the best choice for a normal baseline scan .

---

## Scan Performance

**Scan performance** is the balance between speed, accuracy, and network impact. In pentesting, performance matters because a scan that is too slow wastes time, but a scan that is too aggressive can trigger alerts, lose packets, or create inaccurate results.

A good scan strategy depends on the environment:

- On stable internal networks, faster scans are often acceptable.
- On noisy, fragile, or monitored networks, slower scans may be more reliable.
- In real assessments, timing choices can affect both **detection risk** and **report quality**.

Nmap gives you several knobs to tune this behavior.

### `-T`
The `-T` timing template controls how aggressive Nmap should be. Nmap defines six profiles: **paranoid (`-T0`)**, **sneaky (`-T1`)**, **polite (`-T2`)**, **normal (`-T3`)**, **aggressive (`-T4`)**, and **insane (`-T5`)** .

Examples:
```bash
nmap -T3 <target_ip>
nmap -T4 <target_ip>
```

- `-T0` and `-T1` are slower and are often used when stealth is more important than speed.
- `-T3` is the default and is usually a safe starting point.
- `-T4` and `-T5` are faster but more likely to create noise or miss responses.

### `--host-timeout`
`--host-timeout` sets the maximum time Nmap will spend on one host before giving up. This is useful when a target is slow, filtered, or causing scans to hang.

Example:
```bash
nmap --host-timeout 2m <target_ip>
```

### `--min-rate`
`--min-rate` forces Nmap to send probes at at least a minimum packets-per-second rate. It can speed up large scans, but it may also increase packet loss or detection risk.

Example:
```bash
nmap --min-rate 100 <target_ip>
```

### `--scan-delay`
`--scan-delay` inserts a pause between probes sent to the same host. This is useful when you want to slow traffic down and reduce the chance of triggering alarms.

Example:
```bash
nmap --scan-delay 200ms <target_ip>
```

### Practical view of performance
For footprinting, performance is not only about speed. It is also about **control**: knowing when to scan fast, when to slow down, and when to accept longer scan times in exchange for cleaner results. Nmap’s timing controls let you adapt the scan to the environment instead of using the same speed everywhere .

---

## Output Formats

Nmap can save results in different formats so you can review, filter, or import them later. This is especially useful in footprinting because scan output often becomes the basis for notes, reporting, and later analysis.

### `-oN`
`-oN` saves the scan in **normal text output**. It looks similar to the output shown on the terminal, but it is written to a file.

Example:
```bash
nmap -sS -oN scan.txt <target_ip>
```

### `-oX`
`-oX` saves the scan in **XML format**. This is the most useful format when you want to reuse Nmap results in other tools or automate post-processing.

Example:
```bash
nmap -sS -oX scan.xml <target_ip>
```

### `-oG`
`-oG` saves the scan in **grepable format**. This older format is designed to be easy to parse with shell tools and simple scripts.

Example:
```bash
nmap -sS -oG scan.gnmap <target_ip>
```

### `-oA`
`-oA` saves output in **all major formats at once**: normal, XML, and grepable. This is practical when you want one scan to produce multiple report types.

Example:
```bash
nmap -sS -oA scan_results <target_ip>
```

### Why XML matters
XML is especially important because many tools can import it directly. It preserves structured scan data such as hosts, ports, service info, and scan metadata.

---

## Metasploit Import

In footprinting, you can use **Metasploit’s database features** to import Nmap XML results and organize discovered hosts and services for later review . The key point is that this is still part of reconnaissance and reporting, not exploitation.

Typical workflow:
```bash
nmap -sS -sV -oX scan.xml <target_ip>
msfconsole
db_import scan.xml
hosts
services
```

What happens here:

- `nmap -oX` generates an XML report.
- `msfconsole` opens Metasploit.
- `db_import` loads the XML into the Metasploit database.
- `hosts` and `services` let you review the imported footprinting data.

This is useful because it centralizes reconnaissance results and makes later analysis easier, especially in larger assessments.

---

## Recommended Use

A simple footprinting workflow could look like this:

```bash
nmap -sS -T3 -oN scan.txt <target_ip>
nmap -sS -T4 --scan-delay 100ms -oX scan.xml <target_ip>
nmap -sS -f -D RND:5 -oG scan.gnmap <target_ip>
```

Use fast settings when the network is stable, and slower or more careful settings when you want to reduce noise or test defensive controls .

---

## Key Takeaways

- **Evasion** means changing how probes look so firewalls and IDS tools have a harder time detecting them.
- `-f` fragments packets, `--mtu` sets fragment size, `--data-length` adds random payload, and `-D` uses decoys.
- **Performance** is about balancing speed, accuracy, and stealth during scanning.
- `-T`, `--host-timeout`, `--min-rate`, and `--scan-delay` are the main timing controls for scan behavior.
- `-oN`, `-oX`, and `-oG` save results in different formats for reporting and analysis.
- Nmap XML output is especially useful because it can be imported into `msfconsole` with `db_import` during footprinting.
