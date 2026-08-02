# 08 - Introduction to EDR

## Overview

Endpoint Detection and Response (EDR) is a security solution that continuously monitors endpoint activity, detects suspicious behaviour, supports investigations, and allows security teams to respond to threats.

Endpoints may include:

- Windows workstations
- Linux servers
- macOS devices
- Employee laptops
- Virtual machines

EDR is especially useful in modern environments because many devices operate outside the organisation’s traditional network perimeter.

---

## Core Capabilities

EDR has three main capabilities:

1. Visibility
2. Detection
3. Response

---

## Visibility

EDR collects detailed telemetry from endpoints.

Common telemetry includes:

- Process execution and termination
- Parent-child process relationships
- Command-line activity
- Network connections
- File and folder modifications
- Registry changes
- User activity

This data helps analysts understand what happened before, during, and after a suspicious event.

For example, the process name alone may not be enough to identify malicious activity. The analyst must also review:

- Which process launched it
- Which user executed it
- Which command was used
- Which files were created
- Which network connections followed

A suspicious process chain may look like this:

    winword.exe
    └── powershell.exe
        └── payload.exe

Microsoft Word launching PowerShell may indicate that a malicious macro executed a command.

---

## Detection

EDR platforms use several methods to detect suspicious activity.

### Signature-Based Detection

The EDR compares files or activity with known malicious signatures.

This is useful for known malware but may not detect new or modified threats.

### Behavioural Detection

Behavioural detection focuses on what a process does.

Example:

`winword.exe` launching `powershell.exe` is unusual and may indicate malicious document execution.

### Anomaly Detection

EDR may establish a baseline of normal endpoint activity.

If a process performs an action that is unusual for that endpoint, the activity may be flagged.

### IOC Matching

EDR can compare endpoint activity with known Indicators of Compromise, such as:

- File hashes
- IP addresses
- Domains
- URLs

### Machine-Learning Detection

Machine-learning models may identify suspicious patterns across multiple events.

This is useful when individual actions appear harmless but the complete sequence looks malicious.

---

## What Happens After an Alert?

When an EDR generates an alert, the SOC analyst must review and prioritise it.

Common severity levels include:

- Critical
- High
- Medium
- Low
- Informational

The analyst should review:

- Affected hostname
- Involved user
- Triggering process
- Parent and child processes
- Command line
- File hash
- Network connections
- File and registry changes
- Alert severity
- Related detections

The alert is then classified as:

- False positive
- True positive

A false positive is legitimate activity incorrectly identified as suspicious.

A true positive represents genuine malicious or unauthorised activity.

---

## Basic EDR Investigation Workflow

1. Review the alert severity.
2. Identify the affected endpoint.
3. Identify the involved user.
4. Examine the triggering process.
5. Review the process tree.
6. Inspect the command line.
7. Check file hashes and paths.
8. Review network connections.
9. Examine file and registry changes.
10. Determine whether the activity was authorised.
11. Classify the alert.
12. Take response action if required.
13. Document and escalate the investigation.

---

## Response Capabilities

EDR allows analysts to take action directly from the central console.

Common response actions include:

### Host Isolation

The endpoint is separated from the wider network.

This may prevent:

- Lateral movement
- Data exfiltration
- Malware propagation
- Command-and-control communication

### Process Termination

A suspicious or malicious process can be stopped.

This may be more appropriate than isolating the entire endpoint when the affected system is business-critical.

### File Quarantine

A suspicious file can be moved to an isolated location where it cannot execute.

### Remote Access

Analysts may remotely connect to the endpoint to:

- Review running processes
- Search files
- Check network connections
- Run scripts
- Collect evidence

### Artefact Collection

EDR may collect:

- Event logs
- Memory dumps
- Registry hives
- Suspicious files
- Folder contents

These artefacts can support deeper investigation and forensic analysis.

---

## Antivirus vs EDR

Traditional antivirus mainly focuses on detecting known malicious files.

EDR provides broader visibility by monitoring endpoint behaviour.

| Capability | Antivirus | EDR |
|---|---|---|
| Known malware detection | Yes | Yes |
| Behaviour monitoring | Limited | Yes |
| Process-tree visibility | Limited | Yes |
| Command-line visibility | Limited | Yes |
| Historical telemetry | Limited | Yes |
| Remote response | Usually unavailable | Yes |
| Host isolation | Usually unavailable | Yes |

A file may appear clean when checked by antivirus, but EDR may still detect suspicious behaviour around that file.

---

## Example Attack Chain

A possible attack sequence may look like this:

    Phishing email
        ↓
    Malicious Word document
        ↓
    User opens the document
        ↓
    Macro launches PowerShell
        ↓
    PowerShell downloads a payload
        ↓
    Payload is injected into another process
        ↓
    External connection is established

Traditional antivirus may miss parts of this sequence.

EDR can correlate the process activity, command line, file changes, and network connections to identify the full attack chain.

---

## EDR Architecture

A basic EDR architecture includes:

### Endpoint Agent

The agent is installed on the endpoint.

It collects telemetry and sends it to the central console.

### Central EDR Console

The console:

- Receives endpoint telemetry
- Correlates activity
- Applies detection logic
- Generates alerts
- Stores historical data
- Supports investigation
- Enables response actions

Basic data flow:

    Endpoint activity
        ↓
    EDR agent collects telemetry
        ↓
    Data is sent to the central console
        ↓
    Detection logic analyses the activity
        ↓
    Alert is generated
        ↓
    SOC analyst investigates
        ↓
    Response action is taken

---

## EDR and SIEM

EDR and SIEM are related but have different roles.

EDR provides deep visibility into endpoint activity.

SIEM collects and correlates logs from multiple sources, such as:

- Endpoints
- Firewalls
- Authentication systems
- Email security tools
- Cloud platforms
- Network devices

EDR alerts may be forwarded to a SIEM for wider investigation and correlation.

---

## Analyst Takeaway

EDR is valuable because it gives analysts context.

Instead of showing only that a file is suspicious, it can show:

- Who executed it
- Which process launched it
- Which commands were used
- Which files changed
- Which network connections followed
- What response actions are available

This context helps the analyst make a more accurate decision.

---

## Final Summary

Endpoint Detection and Response provides continuous endpoint monitoring, detailed telemetry, behavioural detection, and direct response capabilities. It helps SOC analysts identify suspicious process relationships, review command-line activity, reconstruct attack chains, and respond to confirmed threats.

EDR does not replace SIEM or network monitoring. Its main value is providing detailed endpoint evidence that supports effective alert triage and incident investigation.

---

## Lessons Learned

- EDR provides visibility, detection, and response.
- Endpoint telemetry is essential for investigation.
- Process relationships can reveal suspicious activity.
- Legitimate tools may be abused by attackers.
- Alerts must be validated before escalation.
- EDR can isolate hosts, terminate processes, quarantine files, and collect artefacts.
- EDR and SIEM work together in a larger security environment.
