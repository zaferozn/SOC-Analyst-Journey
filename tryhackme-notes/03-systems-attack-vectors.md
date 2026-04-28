# Systems Attack Vectors

## Overview

This note summarizes how **systems** can be used as attack vectors and how a **SOC Analyst** should think about system-targeted attacks.

In **human attack vectors**, attackers manipulate people to gain access.

In **system attack vectors**, attackers target insecure systems directly.

A system can be a:

- Physical server
- Virtual machine
- Cloud platform
- Mail server
- Laptop
- Database
- Website management panel
- Industrial server
- Internal business application

Attackers target systems because systems store data, provide services, and control access to important environments.

---

## Human Attack Vectors vs System Attack Vectors

**Human attack vector** means the attacker manipulates people.

**System attack vector** means the attacker targets insecure systems directly.

A phishing attack may compromise one user mailbox.

A mail server compromise may expose thousands of mailboxes.

This is why **system-level compromise** can have a much larger impact.

---

## Why Systems Are Valuable Targets

The value of a system depends on:

- What it stores
- What it controls
- What access it provides
- How critical it is to business operations

Examples:

- A personal laptop may be used to steal gaming accounts or add the device to a botnet.
- A senior IT administrator’s laptop may provide access to internal banking systems.
- A mail server may expose sensitive emails and allow extortion.
- An industrial server may be targeted for ransomware and operational disruption.
- A website management panel may be used for defacement.

---

## Initial Access Against Systems

In many serious attacks, the first goal is to gain access to the target system.

After access is gained, the attacker’s next step depends on motivation.

Possible attacker goals include:

- Data theft
- Ransomware deployment
- Extortion
- Destruction of information
- Lateral movement
- Persistence
- Credential theft

System attacks can begin through:

- Human-led actions
- Software vulnerabilities
- Misconfigurations
- Supply chain compromise

---

## Human-Led System Attacks

A user can unintentionally start a system compromise.

Examples:

- Inserting a malicious USB device
- Downloading malware from pirated resources
- Reusing weak passwords
- Using the same password across multiple services
- Running unsafe files
- Ignoring security warnings

**SOC Analyst perspective:**

A human mistake can become a system compromise.

Example attack chain:

**Weak password → Account compromise → VPN access → Internal system access**

Another example:

**Pirated software → Malware execution → Endpoint compromise → Ransomware risk**

---

## Software Vulnerabilities

A **vulnerability** is a security flaw in software, hardware, or system design.

Every piece of software may contain flaws. Some vulnerabilities exist for years before they are discovered.

A **zero-day** is a vulnerability that is exploited or known before a patch is available.

Once a vulnerability becomes public, it is usually assigned a **CVE** number.

**CVE** stands for **Common Vulnerabilities and Exposures**.

Examples of well-known vulnerabilities:

- **EternalBlue**
- **Follina**
- **PrintNightmare**
- **Shellshock**

When a critical CVE is published, attackers may develop exploits while defenders rush to patch systems.

---

## Responding to Vulnerabilities

The long-term response to a CVE is usually a **patch**.

A patch is an update supplied by the software vendor to fix the vulnerability.

Before a patch is available, organizations may need temporary mitigations.

Examples of temporary mitigations:

- Restrict access to trusted IP addresses
- Apply vendor workarounds
- Block known attack patterns on IPS
- Block known attack patterns on WAF
- Monitor logs for exploitation attempts
- Review suspicious process activity
- Watch for unusual network connections
- Follow threat intelligence advisories

**SOC Analyst perspective:**

A SOC Analyst usually does not directly patch systems.

The SOC Analyst should:

- Identify affected systems
- Monitor for exploitation attempts
- Review relevant logs and alerts
- Collect evidence
- Escalate to IT or security engineering

---

## Misconfigurations

A **misconfiguration** is not a software bug.

A misconfiguration is a security weakness caused by incorrect setup.

Misconfigurations often happen because people choose convenience over security.

Examples:

- Weak passwords
- Default passwords
- Public databases
- Exposed cloud storage
- Disabled firewalls
- Open RDP or SSH
- Unrestricted admin panels
- Overly permissive cloud permissions
- MFA disabled
- Anonymous access enabled
- Unnecessary services exposed to the internet

Simple rule:

**Vulnerability needs a patch.**

**Misconfiguration needs a better setup.**

---

## Database Misconfiguration Example

A company installs a new SQL database.

At first, there are no known vulnerabilities or misconfigurations.

Then customer data is placed into the database.

The system becomes valuable because it now stores sensitive information.

The administrator sets the password to a weak value such as **1111**.

The firewall is disabled or unrestricted access is allowed.

Attackers discover the exposed database and steal the data.

Main misconfigurations:

- Weak password
- Unrestricted access

Possible impact:

- Data theft
- Regulatory risk
- Customer data exposure
- Reputation damage
- Extortion

---

## Responding to Misconfigurations

Misconfigurations do not usually require a software update.

They require a better setup.

Examples of remediation:

- Replace weak passwords with strong passwords
- Disable default credentials
- Restrict access to trusted IPs
- Keep databases internal-only
- Enable firewall rules
- Require VPN for admin access
- Enable MFA
- Remove unnecessary public access
- Review cloud permissions
- Follow secure configuration benchmarks

**SOC Analyst perspective:**

In a large company, the SOC Analyst may not directly fix the configuration.

The analyst should:

- Identify the risky configuration
- Collect evidence
- Assess the impact
- Escalate to IT or security engineering

In smaller organizations, the SOC may also support proactive activities such as vulnerability scanning, configuration audits, and penetration testing coordination.

---

## Supply Chain Attacks

A **supply chain attack** happens when attackers compromise software, libraries, vendors, or update mechanisms used by the organization.

Instead of attacking the organization directly, attackers compromise something the organization trusts.

Example attack chain:

**Trusted software update → Malicious code delivered → Many customers compromised**

Supply chain attacks are dangerous because the malicious activity may come from trusted software or trusted vendors.

Examples of supply chain attacks:

- **SolarWinds**
- **3CX**

**SOC Analyst perspective:**

Supply chain attacks can be difficult to detect because the source may appear legitimate.

A SOC Analyst should monitor for:

- Unusual process behavior
- Suspicious outbound connections
- Unexpected update activity
- New files from trusted applications
- Abnormal DNS requests
- EDR alerts after software updates
- Indicators of compromise
- Threat intelligence advisories

---

## Protecting Systems

Attackers choose the easiest path.

They may attack humans, systems, or both.

Defenders must protect both humans and systems.

System protection requires both **mitigation** and **detection**.

**Mitigation** reduces the chance and impact of attacks.

**Detection** identifies suspicious or malicious activity when prevention fails.

---

## Common System Mitigations

### Patch Management

**Patch management** is the process of tracking and patching vulnerable systems.

It reduces the chance of successful exploitation.

**SOC relevance:**

The SOC may monitor affected systems, exploitation attempts, and whether risk remains after a vulnerability is announced.

### Training for IT

IT teams should understand the risks of misconfigurations.

Training helps reduce simple but dangerous mistakes such as:

- Weak passwords
- Exposed databases
- Disabled firewalls
- Public admin panels
- Unrestricted access

### Network Protection

**Network protection** reduces exposure by restricting system access to trusted users, trusted networks, or trusted IP addresses.

Examples:

- Restrict SSH to trusted IPs
- Restrict admin panels to VPN users
- Keep databases internal-only
- Use firewall allowlists
- Avoid exposing RDP to the public internet

### Antivirus / EDR Protection

**Antivirus** and **EDR** can stop or detect many system-level attacks.

They can detect:

- Malicious files
- Suspicious processes
- Known malware
- Endpoint behavior that may indicate compromise

---

## SOC Investigation Questions

When investigating a system attack, a SOC Analyst should ask:

- Which system was targeted?
- Why is this system valuable?
- What access does this system provide?
- Is the system exposed to the internet?
- Is there a known vulnerability or CVE?
- Is the system patched?
- Is there a misconfiguration?
- Are there weak or default credentials?
- Were there suspicious login attempts?
- Was exploitation successful?
- Are there new processes, services, files, or users?
- Are there unusual network connections?
- Is there evidence of lateral movement?
- Does the case require escalation?

---

## Analyst Summary

Systems are valuable targets because they store data, provide services, and control access to important environments.

Attackers may target systems through human-led actions, software vulnerabilities, misconfigurations, or supply chain compromise.

A system compromise can lead to credential theft, data theft, ransomware, lateral movement, extortion, operational disruption, or defacement.

As a SOC Analyst, I should identify which system was targeted, why it matters, what access it provides, whether the attack was successful, and what evidence supports escalation.

---

## Interview Answer

System attacks often begin with initial access.

This access can come from user actions, software vulnerabilities, weak configurations, exposed services, or compromised supply chain components.

As a SOC Analyst, I would identify the affected system, understand why it is valuable, review relevant logs and alerts, check for vulnerabilities or misconfigurations, collect evidence, and escalate to IT or security engineering for patching, mitigation, or incident response.

---

## Core Memory Lines

**Human attack vector** = attacker manipulates people.

**System attack vector** = attacker targets insecure systems directly.

**Initial access** is often the first goal of a system attack.

**Vulnerability** = software or system flaw.

**Zero-day** = vulnerability without an available patch.

**CVE** = Common Vulnerabilities and Exposures.

**Patch** = vendor update that fixes a vulnerability.

**Misconfiguration** = insecure setup.

**Vulnerability needs a patch.**

**Misconfiguration needs a better setup.**

**Supply chain attack** = compromise of trusted software, libraries, vendors, or update mechanisms.

**Patch management** reduces exploitation risk.

**Network protection** reduces exposure.

**Antivirus and EDR** help detect or block malware activity.

A SOC Analyst should identify the affected system, assess impact, collect evidence, and escalate properly.
