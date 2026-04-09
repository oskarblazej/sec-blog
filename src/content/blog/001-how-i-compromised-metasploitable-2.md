---
title: "How I Compromised Metasploitable 2 (Walkthrough)"
description: "From enumeration to root - a practical breakdown of compromising an intentionally vulnerable system"
pubDate: 2026-04-09
updatedDate: 2026-04-09
heroImage: ""
---

# Context

Metasploitable 2 is an intentionally vulnerable virtual machine designed for security training. Since the goal is to simulate a misconfigured real-world environment, it provides a perfect playground for practicing offensive techniques.

I originally approached this machine as a formal penetration testing exercise, but this writeup focuses purely on the technical side - how I approached the system, what stood out, and how different attack paths led to full compromise.

# Enumeration

The objective was simple: gain root access from as many entry points as possible and understand the associated risks.

I started with a standard TCP port scan and service enumeration using Nmap. The results immediately showed a wide attack surface with multiple exposed and outdated services:

- 21/TCP (FTP)
- 22/TCP (SSH)
- 23/TCP (Telnet)
- 25/TCP (SMTP)
- 80/TCP (HTTP)
- 139/445/TCP (SMB)
- 512/513/TCP (REXEC/RLOGIN)
- 5900/TCP (VNC)
- 8180/TCP (Tomcat)

At this point, the system already looked heavily misconfigured. The next step was identifying the fastest and most reliable paths to initial access.

# Exploitation

## VSFTPD 2.3.4 Backdoor (CVE-2011-2523)

Port 21 immediately stood out. The service was running vsftpd 2.3.4 - a version known for a backdoor vulnerability.

Instead of blindly throwing exploits, I verified the version and confirmed that CVE-2011-2523 was applicable. Given the nature of Metasploitable, the likelihood of success was high.

The exploit works by sending a crafted username, which triggers a backdoor and opens a shell on port 6200. After setting up a listener, I was able to capture a reverse shell almost instantly.

![VSFTPD](/images/metasploitable/001-vsftpd.jpg)

## REXEC / RLOGIN (Ports 512/513)

Ports 512 and 513 exposed legacy r-services, which are almost always a red flag.

Instead of overcomplicating things, I tested direct access. RLOGIN allowed me to establish a shell without providing valid credentials, effectively granting immediate access.

This was one of the clearest examples of how dangerous legacy services can be when left exposed.

![RLOGIN exploitation](/images/metasploitable/001-rlogin.jpg)

## Samba (Port 445) – CVE-2007-2447

Port 445 was another obvious target. SMB services are often a strong entry point, especially on older systems.

The service was running Samba 3.0.20, which is outdated. After verifying the version, I identified CVE-2007-2447 - a vulnerability allowing command execution via the username map script.

The assumption was simple: if the vulnerable configuration is enabled, exploitation should be straightforward.

That turned out to be the case. The exploit worked without resistance, resulting in a shell on the target system.

## VNC (Port 5900) – Weak Authentication

Port 5900 exposed a VNC service, which immediately suggested potential misconfiguration.

Instead of jumping into brute force, I tested common/default credentials first. The service accepted the password `password`, which confirmed weak authentication.

Using a VNC client, I was able to establish a remote desktop session as root.

No exploit was needed here - just access. This is a good example of how misconfiguration alone can lead to full compromise.

In a real-world scenario, this level of access would allow:
- full system control
- data exfiltration
- persistence
- lateral movement

![VNC exploitation](/images/metasploitable/001-vnc.jpg)

## Apache Tomcat (Port 8180) – Credential Discovery & Pivoting

Port 8180 exposed an Apache Tomcat instance with management interfaces available.

I initially searched for public exploits for Tomcat 5.5, but none worked. I then attempted credential brute-forcing using Burp Suite, but it quickly became clear this wasn’t an efficient path.

At this point, I changed strategy.

Since I already had root access via RLOGIN, it made more sense to leverage that instead of continuing external attacks. I searched for Tomcat configuration files and located `tomcat-users.xml`, which contained plaintext credentials.

Using those credentials, I authenticated to the Tomcat Manager and gained administrative access.

This was significantly faster and more reliable than brute-forcing. The service itself wasn’t directly vulnerable - the real issue was poor credential management combined with exposed configuration files.

With access to Tomcat Manager, an attacker could deploy malicious applications, execute arbitrary code, and maintain persistence.

## Password Cracking – /etc/shadow

After gaining root access, I moved into post-exploitation.

I extracted `/etc/passwd` and `/etc/shadow` and used `unshadow` to prepare them for offline cracking. Instead of attempting anything online, I performed the attack locally to avoid detection and limitations.

Using John the Ripper with the rockyou.txt wordlist, I cracked multiple user passwords. The root password remained secure, but that didn’t reduce the value of the recovered credentials.

In real-world environments, credential reuse is common. Even non-root accounts can be leveraged for further access across systems.

At this stage, the focus shifts from exploitation to data extraction and leveraging access for deeper compromise.

![JOHNTHERIPPER password cracking](/images/metasploitable/001-john.jpg)

# Attack Path Summary

1. Enumeration revealed multiple exposed services
2. Initial access gained via VSFTPD backdoor
3. Direct shell access through RLOGIN (no authentication)
4. Additional access via Samba exploitation
5. Credentials extracted from system files
6. Tomcat compromised using recovered credentials
7. Password hashes cracked for further leverage

# Lessons Learned

- Enumeration is everything - most entry points were obvious after scanning
- Legacy services (RLOGIN, Telnet) are extremely high-risk when exposed
- Weak credentials can be more dangerous than vulnerabilities
- Pivoting from existing access is often faster than exploiting externally
- Post-exploitation (credentials, configs) provides massive leverage
- Not every compromise requires complexity - recognizing patterns is often enough