# TryHackMe - Pyramid of Pain

## Overview

The **Pyramid of Pain**, created by David J. Bianco, explains how difficult different types of threat indicators are for an attacker to change.

The higher we move in the pyramid, the more difficult it becomes for an attacker to adapt.

From bottom to top:

1. Hash Values
2. IP Addresses
3. Domain Names
4. Network Artifacts
5. Tools
6. Tactics, Techniques, and Procedures (TTPs)

---

## Hash Values

Hashes can be used to uniquely identify files.

Common hashing algorithms include:

- MD5
- SHA-1
- SHA-256

Security analysts commonly use file hashes to identify known malicious files through threat intelligence platforms such as VirusTotal.

However, hashes are easy for attackers to change. Modifying even a small part of a file produces a completely different hash.

Example:

```powershell
Get-FileHash .\sample.exe -Algorithm SHA256
```

Because hashes can be changed easily, they are located near the bottom of the Pyramid of Pain.

---

## IP Addresses

IP addresses can be used as Indicators of Compromise (IOCs).

SOC analysts may identify suspicious IP addresses through:

- Firewall logs
- Proxy logs
- SIEM alerts
- IDS alerts
- Threat intelligence platforms

A malicious IP address can be blocked, but attackers can often switch to another IP address relatively easily.

### Fast Flux

**Fast Flux** is a DNS technique where a domain is associated with multiple IP addresses that change frequently.

It can be used to hide infrastructure related to:

- Phishing
- Malware delivery
- Botnets
- Command and Control (C2)

---

## Domain Names

Domains are generally more difficult for attackers to replace than IP addresses because they may need to register new domains and modify DNS infrastructure.

SOC analysts can investigate suspicious domains using:

- DNS logs
- Proxy logs
- Web logs
- SIEM data
- Threat intelligence platforms

### Punycode

Attackers can use Unicode characters to create domains that visually resemble legitimate websites.

Example:

```text
adıdas.de
xn--addas-o4a.de
```

This technique may be used in phishing attacks.

### URL Shorteners

Attackers may also use URL-shortening services to hide the final destination of a malicious link.

Examples:

```text
bit.ly
tinyurl.com
ow.ly
s.id
```

During investigation, analysts should identify the final redirect destination.

---

## Network Artifacts

Network artifacts provide information about how malicious activity behaves on the network.

Examples include:

- User-Agent strings
- URI patterns
- HTTP requests
- DNS requests
- C2 communication patterns

These artifacts can be identified using tools such as:

- Wireshark
- TShark
- Snort
- IDS/IPS
- ANY.RUN

Example TShark command:

```bash
tshark -Y http.request -T fields -e http.host -e http.user_agent -r analysis_file.pcap
```

A unique User-Agent or communication pattern can provide a stronger detection opportunity than a single IP address or file hash.

---

## Tools

Attackers may use:

- Malware
- Backdoors
- Malicious executables
- DLL files
- Password-cracking tools
- Malicious documents
- Custom payloads

Defenders can detect these tools through:

- Antivirus signatures
- EDR detections
- YARA rules
- Detection rules
- Malware analysis

### Fuzzy Hashing

Fuzzy hashing can identify similarities between files even when they have been slightly modified.

One example is:

```text
ssdeep
```

This can help analysts identify related malware samples.

---

## Tactics, Techniques, and Procedures - TTPs

TTPs describe **how an attacker operates**.

They can be mapped using the **MITRE ATT&CK framework**.

Examples include:

- Phishing
- Credential theft
- Persistence
- Pass-the-Hash
- Lateral movement
- Command and Control
- Data exfiltration

TTPs are at the top of the Pyramid of Pain because changing attacker behaviour, techniques, and procedures requires significantly more effort than simply changing an IP address or file hash.

---

## Pyramid of Pain

```text
            TTPs
           Tools
     Network Artifacts
        Domain Names
       IP Addresses
       Hash Values
```

The higher the indicator is in the pyramid, the more difficult it is for the attacker to change.

---

## SOC Analyst Takeaway

Hashes, IP addresses, and domains are useful during initial IOC investigation, but they can often be changed relatively easily.

Network artifacts, attacker tools, and especially TTPs provide stronger detection opportunities.

A SOC analyst should therefore move beyond simply asking:

> What IOC did I find?

and also ask:

> What attacker behaviour does this activity reveal?

Behaviour-based detection can provide more resilient defensive coverage than relying only on static indicators.
