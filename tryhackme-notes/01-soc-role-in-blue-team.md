# SOC Role in Blue Team

## Overview

This note summarizes the role of a Security Operations Center (SOC) within a Blue Team environment.

A SOC is the first line of defense in an organization. Its main responsibility is to detect, investigate, document, respond to, and escalate security events and incidents.

The main focus of a SOC is:

- Detection
- Investigation
- Response
- Documentation
- Escalation

## SOC Core Workflow

A simple SOC workflow can be described as:

```text
Detect → Investigate → Document → Escalate
As a Junior SOC Analyst, my role is to perform initial alert triage, review logs, collect evidence, document findings, and escalate suspicious or critical cases according to the SOC playbook.
```
SOC Pillars

A mature SOC is built on three pillars:

* People
* Process
* Technology

People include SOC analysts, SOC engineers, detection engineers, incident responders, and SOC managers.

Process includes playbooks, escalation rules, ticketing procedures, reporting standards, and incident handling workflows.

Technology includes SIEM, EDR, firewalls, IDS/IPS, XDR, SOAR, antivirus, and endpoint protection tools.

Key SOC Roles

SOC L1 Analyst

A SOC L1 Analyst performs initial alert triage. The L1 analyst reviews basic evidence, checks logs, identifies suspicious activity, documents findings, and escalates complex cases.

SOC L2 Analyst

A SOC L2 Analyst performs deeper investigations and correlates data from multiple sources such as SIEM logs, EDR telemetry, firewall logs, proxy logs, and authentication logs.

SOC L3 Analyst

A SOC L3 Analyst handles advanced investigations, supports incident response, performs threat hunting, and helps with containment, eradication, and recovery.

SOC Engineer

A SOC Engineer configures, maintains, and troubleshoots SOC tools such as SIEM, EDR, log collectors, dashboards, and alerting systems.

Detection Engineer

A Detection Engineer creates and improves detection rules, writes SIEM queries, tunes alerts, reduces false positives, and maps detections to MITRE ATT&CK.

SOC Manager

A SOC Manager manages the SOC team, processes, reporting, shift coverage, performance, and communication with leadership.

Incident Responder / CIRT / CSIRT

Incident responders and CIRT/CSIRT teams handle critical incidents such as ransomware, breaches, malware outbreaks, and compromised infrastructure.

Supporting Security Roles

GRC Auditor

A GRC Auditor focuses on governance, risk, and compliance. Examples include PCI DSS, ISO 27001, GDPR, NIST, and SOC 2.

Penetration Tester

A Penetration Tester performs authorized security testing to identify vulnerabilities before real attackers exploit them.

Threat Researcher / Threat Intelligence Analyst

A Threat Researcher studies threat actors, attack campaigns, indicators of compromise, and attacker tactics, techniques, and procedures.

Digital Forensics Analyst

A Digital Forensics Analyst examines digital evidence such as disk images, memory dumps, files, logs, and system artifacts to reconstruct incidents and identify root cause.

Alert Triage and 5Ws

During alert triage, a SOC analyst should answer the 5Ws:

* What happened?
* When did it happen?
* Where did it happen?
* Who was involved?
* Why did it happen?

This helps structure the investigation and supports clear reporting.

Key Technologies

SIEM

A SIEM collects and correlates logs from multiple sources and generates alerts based on detection rules.

EDR

EDR provides endpoint visibility and response capability on systems such as laptops, desktops, and servers.

Firewall

A firewall filters network traffic and can block suspicious or unauthorized connections before they reach the internal network.

Internal SOC vs MSSP

In an internal SOC, analysts protect one organization.

In an MSSP, analysts protect multiple customer environments.

Internal SOC provides deeper knowledge of one organization, while MSSP provides broader exposure to many customers, tools, alerts, and incidents.

Analyst Summary

In a SOC environment, different roles handle different parts of security operations. A SOC L1 Analyst performs initial triage and escalates complex cases. A SOC L2 Analyst performs deeper investigation. A SOC Engineer maintains security tools. A Detection Engineer builds detection logic. A GRC Auditor focuses on compliance. A Penetration Tester identifies vulnerabilities. A Threat Researcher studies attacker behavior. For critical incidents, CIRT or CSIRT teams handle incident response.

Personal Reflection

My previous work experience required discipline, reporting, escalation, shift work, and staying calm under pressure. These skills are directly relevant to SOC work. As a SOC L1 Analyst, I would focus on triaging alerts, reviewing evidence, documenting findings, following procedures, and escalating cases to the correct team or role.
