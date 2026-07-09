# Windows File System Vulnerabilities

This lesson covers **Alternate Data Streams (ADS)**, an NTFS file system feature that attackers abuse to hide malicious payloads inside legitimate files — evading basic antivirus detection.

> ⚠️ **Note**: All commands in this lesson are executed on the **target Windows system** via a shell or command prompt session. Administrative privileges are required for the symlink step.

---

## Alternate Data Streams (ADS)

### What is ADS?

**Alternate Data Streams (ADS)** is an attribute of the **NTFS file system** (used by Windows). It was originally designed to provide compatibility with the **Apple HFS (Hierarchical File System)** when transferring files between macOS and Windows.

Every file created on an NTFS-formatted drive contains **two streams**:

| Stream | Purpose | Visible to User |
|--------|---------|----------------|
| **Data Stream** | The default stream — contains the actual file content | Yes |
| **Resource Stream** | Contains the file's metadata (author, timestamps, etc.) | No |

The **Resource Stream** is the key — it can store arbitrary data, including executable code, and it remains **completely hidden** from standard directory listings and Windows Explorer.

### Why Attackers Use ADS

ADS is a powerful evasion technique because:

- The host file appears **normal** — same name, same visible size.
- The hidden stream **does not show up** in `dir` listings or Explorer.
- Basic **signature-based antivirus** and **static scanning tools** do not inspect ADS.
- The hidden data persists as long as the host file exists.

---

## ADS in Practice

### Step 1: Hide Data Inside a File Stream

The following creates a hidden stream called `secret.txt` inside `test.txt`. Even if `test.txt` has no visible content, the secret stream stores data independently:

```cmd
notepad test.txt:secret.txt
```

When Notepad opens, type your hidden content and save. The result:
- `test.txt` shows **0 bytes** in Explorer and `dir` — looks empty.
- The content is stored invisibly in the `secret.txt` stream of `test.txt`.

### Step 2: Hide a Malicious Payload in a Legitimate File

Copy your payload into the resource stream of an innocent-looking log file:

```cmd
type payload.exe > C:\Temp\windowslog.txt:payload.exe
```

What happens:
- `windowslog.txt` appears as a normal, empty (0 byte) log file.
- `payload.exe` is now hidden inside its resource stream.
- You can now **delete the original `payload.exe`** — the copy persists inside `windowslog.txt`.

```cmd
del payload.exe
```

### Step 3: Create a Symlink to Execute the Hidden Payload

To execute the hidden payload without extracting it, create a **symbolic link** that maps a command name directly to the hidden stream. This must be done with **administrative privileges**:

```cmd
cd C:\Windows\System32
mklink winsvclog.exe C:\Temp\windowslog.txt:payload.exe
```

What this does:
- Creates a symlink named `winsvclog.exe` in `System32`.
- When `winsvclog.exe` is typed in any command prompt, Windows executes the hidden `payload.exe` from inside `windowslog.txt`.
- The payload runs with no visible file on disk — only an empty log file and a symlink exist.

### Step 4: Execute the Hidden Payload

```cmd
winsvclog.exe
```

The payload executes silently from within the ADS stream.

---

## Verifying ADS (Detection)

Standard `dir` does **not** reveal ADS streams. To list all streams on a file:

```cmd
dir /r C:\Temp\windowslog.txt
```

The `/r` flag reveals resource streams. Output example:

```
windowslog.txt          0 bytes
windowslog.txt:payload.exe:$DATA
```

You can also use **Sysinternals Streams** tool:

```cmd
streams.exe C:\Temp\windowslog.txt
```

---

## ADS Attack Summary

| Step | Command | Purpose |
|------|---------|---------|
| **1. Hide data** | `notepad file.txt:hidden.txt` | Store data invisibly in a stream |
| **2. Hide payload** | `type payload.exe > log.txt:payload.exe` | Copy executable into file's resource stream |
| **3. Delete original** | `del payload.exe` | Remove visible evidence |
| **4. Create symlink** | `mklink winsvclog.exe C:\Temp\log.txt:payload.exe` | Map a command to the hidden stream |
| **5. Execute** | `winsvclog.exe` | Run hidden payload with no file on disk |
| **6. Detect** | `dir /r file.txt` | Reveal hidden streams |

---

## Key Takeaways

- **ADS (Alternate Data Streams)** is an NTFS feature that allows files to contain multiple data streams — only the default stream is visible.
- Originally designed for **macOS HFS compatibility**, ADS is abused by attackers to hide malicious payloads inside legitimate files.
- Every NTFS file has a **Data Stream** (visible content) and a **Resource Stream** (hidden metadata — where attackers hide payloads).
- A file with a hidden ADS stream reports **0 bytes** in Explorer and standard `dir` — it looks completely empty.
- Use `type payload.exe > file.txt:payload.exe` to copy an executable into a file's hidden stream.
- Use `mklink` in `System32` to create a symlink that executes the hidden payload by name — requires **admin privileges**.
- Use `dir /r` or **Sysinternals Streams** to detect and reveal hidden ADS streams.
- ADS evades **basic signature-based AV** and **static scanners** — but modern EDR solutions and advanced AV can detect it.
