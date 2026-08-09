# TryHackMe Notes - Cyber Kill Chain

## Overview

The **Cyber Kill Chain**, developed by Lockheed Martin, describes the stages an attacker may follow during a cyberattack.

For SOC analysts, it helps identify **where an attacker is in the attack lifecycle** and where defenders can detect or disrupt the attack.

## Cyber Kill Chain Stages

### 1. Reconnaissance

The attacker gathers information about the target.

Examples:

- OSINT
- WHOIS lookups
- Social media research
- Email harvesting
- Port scanning
- Banner grabbing

**Passive Recon:** No direct interaction with the target.  
**Active Recon:** Direct interaction, such as scanning or probing systems.

---

### 2. Weaponization

The attacker prepares the malicious payload.

Examples:

- Malware
- Exploits
- Malicious Office documents
- Malicious macros
- Backdoors
- C2 infrastructure

**Malware:** Malicious software.  
**Exploit:** Code that abuses a vulnerability.  
**Payload:** Malicious code executed on the target.

---

### 3. Delivery

The attacker sends the payload to the victim.

Common methods:

- Phishing emails
- Malicious attachments
- Malicious links
- USB drops
- Watering hole attacks

For SOC analysts, phishing analysis is an important detection point at this stage.

---

### 4. Exploitation

The malicious code executes or a vulnerability is exploited.

Examples:

- Malicious macro execution
- Known CVE exploitation
- Zero-day exploitation

Possible indicators:

- Unexpected processes
- Suspicious command-line activity
- Registry changes
- New services

---

### 5. Installation

The attacker establishes persistence to maintain access.

Examples:

- Web shells
- Backdoors
- Modified Windows services
- Registry Run Keys
- Startup Folder persistence
- Timestomping

Relevant MITRE ATT&CK examples:

- `T1543.003` - Windows Service
- `T1547.001` - Registry Run Keys / Startup Folder
- `T1070.006` - Timestomp

---

### 6. Command and Control (C2)

The compromised system communicates with attacker-controlled infrastructure.

Common C2 channels:

- HTTP / HTTPS
- DNS
- DNS tunneling

Possible indicators:

- Regular outbound connections
- Beaconing
- Suspicious DNS requests
- Connections to unusual domains or IP addresses

---

### 7. Actions on Objectives

The attacker performs the final objective.

Examples:

- Credential theft
- Privilege escalation
- Lateral movement
- Data collection
- Data exfiltration
- Backup deletion
- Data encryption or destruction

---

## Cyber Kill Chain Summary

| Stage | Main Purpose |
|---|---|
| Reconnaissance | Gather information |
| Weaponization | Prepare malicious payload |
| Delivery | Send payload |
| Exploitation | Execute code / exploit vulnerability |
| Installation | Establish persistence |
| Command & Control | Remotely control victim |
| Actions on Objectives | Achieve final goal |

## Limitations

The traditional Cyber Kill Chain is useful but does not describe every modern attack well.

Limitations include:

- Attacks are not always linear.
- Attackers may skip or repeat stages.
- It focuses heavily on malware and perimeter attacks.
- It is less effective for insider threats.

It can therefore be combined with **MITRE ATT&CK** for more detailed analysis.

## SOC Analyst Takeaway

A SOC analyst should not investigate an alert only as an isolated event.

Think about the larger sequence:

`Phishing → Execution → Persistence → C2 → Actions on Objectives`

The key question is:

**What stage of the attack could this alert represent, and what activity happened before or after it?**
