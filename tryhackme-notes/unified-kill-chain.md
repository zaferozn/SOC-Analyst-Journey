# TryHackMe - Unified Kill Chain

## Overview

The Unified Kill Chain (UKC) is a cybersecurity framework used to understand how an attacker progresses through an intrusion.

It was developed by Paul Pols and combines concepts from several attack frameworks. The UKC complements frameworks such as MITRE ATT&CK and provides a detailed view of attacker behaviour before, during, and after gaining access to a target.

The framework contains **18 attack phases**.

---

## Why the Unified Kill Chain Matters

The UKC helps defenders:

- Understand attacker behaviour and objectives
- Reconstruct the sequence of an attack
- Identify possible attack surfaces
- Improve threat modelling
- Detect opportunities to interrupt an intrusion
- Map observed activity to defensive controls
- Understand how attackers move between different stages

Unlike a strictly linear attack model, attackers may return to earlier phases during an intrusion.

For example, after compromising one system, an attacker may perform additional reconnaissance and discovery before moving to another system.

---

## Unified Kill Chain Phases

### Initial Access / Foothold

1. **Reconnaissance**  
   Gathering information about targets, systems, users, services, and infrastructure.

2. **Weaponization**  
   Preparing infrastructure, payloads, malware, or command-and-control systems for the attack.

3. **Delivery**  
   Transmitting the malicious object or payload to the target.

4. **Social Engineering**  
   Manipulating users into performing actions such as opening attachments or entering credentials.

5. **Exploitation**  
   Taking advantage of vulnerabilities to execute attacker-controlled code.

6. **Persistence**  
   Maintaining access to a compromised system.

7. **Defense Evasion**  
   Attempting to avoid security controls such as antivirus, firewalls, IDS, or EDR.

8. **Command & Control (C2)**  
   Establishing communication between the compromised system and attacker infrastructure.

---

## Through the Network

9. **Pivoting**  
   Using a compromised system to access systems that are otherwise unreachable.

10. **Discovery**  
    Gathering information about users, systems, applications, files, permissions, and network resources.

11. **Privilege Escalation**  
    Obtaining higher permissions such as administrator, SYSTEM, or root.

12. **Execution**  
    Running attacker-controlled commands, scripts, or malicious code.

13. **Credential Access**  
    Stealing usernames, passwords, hashes, or other authentication information.

14. **Lateral Movement**  
    Moving from one compromised system to other systems within the environment.

---

## Attack Objectives

15. **Collection**  
    Gathering valuable information before removing it from the environment.

16. **Exfiltration**  
    Transferring stolen information outside the victim network.

17. **Impact**  
    Disrupting, modifying, encrypting, or destroying systems and data.

18. **Objectives**  
    Achieving the attacker's strategic goal, such as financial gain, espionage, disruption, or reputational damage.

---

## UKC and MITRE ATT&CK

The Unified Kill Chain and MITRE ATT&CK should not be treated as competing frameworks.

The UKC helps explain **where an attacker is within the overall intrusion lifecycle**, while MITRE ATT&CK provides detailed tactics and techniques describing **how specific attacker actions are performed**.

Using both frameworks can provide stronger context during incident investigation.

---

## SOC Analyst Perspective

For a SOC analyst, the Unified Kill Chain helps connect individual alerts into a larger attack story.

For example:

`Reconnaissance → Exploitation → Persistence → Credential Access → Lateral Movement → Collection → Exfiltration`

A single alert may represent only one stage of an intrusion.

The analyst should therefore ask:

- What happened before this alert?
- What activity occurred after it?
- Is the attacker attempting to move to another system?
- Are credentials being targeted?
- Is persistence being established?
- Is there evidence of data collection or exfiltration?

This prevents alerts from being investigated in isolation.

---

## Example SOC Scenario

An attacker:

1. Scans a public server.
2. Exploits a vulnerable application.
3. Creates a persistence mechanism.
4. Discovers internal systems.
5. Steals credentials.
6. Moves laterally to another host.
7. Collects sensitive files.
8. Sends the files to external infrastructure.

Using the UKC, these events can be reconstructed as:

`Reconnaissance → Exploitation → Persistence → Discovery → Credential Access → Lateral Movement → Collection → Exfiltration`

---

## Key Lesson

The most important lesson from the Unified Kill Chain is that cyber attacks are not isolated events.

Individual alerts, authentication events, network connections, process executions, and file changes may represent different stages of the same intrusion.

A SOC analyst should correlate these events and determine the attacker's current position, previous activity, and likely next objective.
