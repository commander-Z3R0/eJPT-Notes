# Linux Vulnerabilities

Linux is a free and open-source operating system composed of the Linux kernel and the GNU toolkit. It is widely used in many environments, but it is most commonly deployed as a server operating system.

Because of this, Linux servers typically run specific services and protocols. These services can expose attack surfaces that attackers may exploit to gain access to a target system.

## Why Linux Is a Target

Linux systems are targeted by attackers for several reasons:

1. **Server dominance:** A large portion of web servers and infrastructure systems run Linux.
2. **Service exposure:** Linux servers often expose network services such as SSH, FTP, and web servers.
3. **Misconfigurations:** Improper permissions, weak configurations, or exposed services can introduce vulnerabilities.
4. **Outdated software:** Unpatched packages or old kernels may contain known exploits.
5. **Open-source nature:** While transparent, vulnerabilities are also publicly studied and can be weaponized quickly.
6. **Credential weaknesses:** Weak passwords or reused SSH keys can lead to unauthorized access.

## Types Of Linux Vulnerabilities

Linux vulnerabilities can arise from the system, installed services, or configuration issues. Common examples include:

- **Privilege escalation:** Exploiting kernel or SUID binaries to gain higher privileges.
- **Remote code execution (RCE):** Vulnerabilities in services that allow attackers to execute commands remotely.
- **Misconfigured permissions:** Incorrect file or directory permissions exposing sensitive data.
- **Weak authentication:** Poor SSH configurations, weak passwords, or lack of key-based authentication.
- **Unpatched software:** Vulnerabilities in outdated packages or services.
- **Insecure services:** Running unnecessary or insecure services like Telnet or outdated FTP servers.
- **Kernel exploits:** Bugs in the Linux kernel that allow attackers to compromise the system.
- **Cron job abuse:** Exploiting scheduled tasks with weak permissions.
- **Environment variable abuse:** Manipulating PATH or LD_PRELOAD for exploitation.

## Frequently Exploited Linux Services

| Service | Ports | Purpose |
|---|---|---|
| Apache | TCP ports 80/443 | Free, open-source cross-platform web server. Widely used to host websites and web applications. |
| SSH | TCP port 22 | Secure protocol used to remotely access and manage systems over a network. |
| FTP | TCP port 21 | Protocol used for file transfer between systems. Often insecure if not properly configured. |
| SAMBA | TCP port 445 | Linux implementation of SMB, used for file and printer sharing across networks. |

## Key Idea

Linux is a critical part of modern infrastructure, especially in server environments. Its widespread use, combined with exposed services and potential misconfigurations, makes it an important target for attackers. Understanding common services and how they can be exploited is essential for both defenders and penetration testers.
