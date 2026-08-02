# 08 - Introduction to EDR

## Overview

Endpoint Detection and Response (EDR) is a security solution that monitors endpoint activity, detects suspicious behaviour, and allows analysts to respond to threats.

EDR focuses on devices such as workstations, laptops, and servers.

---

## Core Capabilities

EDR has three main functions:

- Visibility
- Detection
- Response

### Visibility

EDR collects endpoint telemetry such as:

- Process activity
- Command-line activity
- Network connections
- File changes
- Registry changes
- User activity

This helps analysts reconstruct what happened on the endpoint.

### Detection

EDR can detect threats through:

- Known signatures
- Suspicious behaviour
- Anomaly detection
- IOC matching
- Machine-learning analysis

Example:

`winword.exe` launching `powershell.exe` may indicate malicious macro execution.

### Response

Analysts may use EDR to:

- Isolate a host
- Terminate a process
- Quarantine a file
- Access the endpoint remotely
- Collect forensic artefacts

---

## Antivirus vs EDR

Traditional antivirus mainly detects known malicious files.

EDR provides deeper visibility by monitoring behaviour and process relationships.

A file may appear clean, but EDR can still detect suspicious activity around it.

---

## Basic EDR Workflow

1. The endpoint agent collects telemetry.
2. Data is sent to the central EDR console.
3. Detection logic analyses the activity.
4. An alert is generated.
5. The SOC analyst investigates.
6. The alert is classified as a false positive or true positive.
7. Response action is taken when necessary.

---

## Analyst Focus

During an investigation, the analyst should review:

- Affected host
- Involved user
- Parent and child processes
- Command line
- File hash
- Network connections
- File and registry changes
- Alert severity
- Whether the activity was authorised

---

## EDR and SIEM

EDR provides deep endpoint visibility.

SIEM combines logs from multiple sources such as endpoints, firewalls, authentication systems, and cloud services.

EDR alerts can be sent to a SIEM for wider correlation and investigation.

---

## Final Summary

EDR helps SOC analysts detect and investigate suspicious endpoint behaviour. Its value comes from detailed telemetry, behavioural detection, and direct response capabilities. It does not replace SIEM or network monitoring, but it provides the endpoint context required for effective incident triage.

---

## Lessons Learned

- EDR monitors endpoint behaviour continuously.
- Process relationships are important investigation evidence.
- Legitimate tools can be abused by attackers.
- Alerts must be validated before escalation.
- Response actions should be proportionate to the risk.
