# 08 - Introduction to EDR

## Overview

Endpoint Detection and Response (EDR) is a security solution that continuously monitors endpoint activity, detects suspicious behaviour, supports investigations, and allows security teams to respond to threats.

Endpoints may include:

- Windows workstations
- Linux servers
- macOS devices
- Laptops
- Virtual machines

EDR is especially important in remote-work environments because devices may operate outside the organisation’s traditional network perimeter.

---

## Learning Objectives

This room introduced:

- The purpose of EDR
- The differences between EDR and antivirus
- The main components of EDR architecture
- The telemetry collected from endpoints
- EDR detection methods
- EDR response capabilities
- The SOC analyst’s role during alert investigation

---

## The Three Pillars of EDR

The three main capabilities of an EDR solution are:

1. Visibility
2. Detection
3. Response

---

## 1. Visibility

EDR provides detailed visibility into activities occurring on an endpoint.

It may collect information about:

- Process execution and termination
- Parent-child process relationships
- Command-line activity
- Network connections
- File and folder modifications
- Registry modifications
- User activity

This information allows analysts to reconstruct the sequence of events surrounding suspicious activity.

### Process Trees

A process tree shows how processes are related.

Example:

    winword.exe
    └── powershell.exe
        └── payload.exe

Microsoft Word launching PowerShell may be suspicious because Word does not normally need to start a PowerShell process.

![EDR process tree showing parent-child process relationships](images/edr-process-tree.png)

The process name alone is not enough to determine whether an activity is malicious. Analysts must also examine:

- Parent process
- Child processes
- Command line
- Execution path
- User account
- File hash
- Network activity

---

## 2. Detection

EDR solutions use several detection methods.

### Signature-Based Detection

The EDR compares files and activity against known malicious signatures.

This method is useful for detecting known malware but may not identify new or modified threats.

### Behavioural Detection

Behavioural detection examines what a process does rather than relying only on its signature.

Example:

    winword.exe → powershell.exe

This unusual relationship may indicate that a malicious document macro launched PowerShell.

### Anomaly Detection

EDR platforms may learn the normal behaviour of endpoints and identify activities that deviate from the established baseline.

Example:

A process modifies an auto-start registry key on an endpoint where this activity is not normally observed.

Anomaly detection may also generate false positives, so analyst validation is required.

### IOC Matching

EDR can compare endpoint activity with known Indicators of Compromise, such as:

- File hashes
- IP addresses
- Domains
- URLs
- File names

Example:

A downloaded executable matches a malicious hash contained in a threat intelligence feed.

### Machine-Learning Detection

Machine-learning models can identify complex attack patterns where the individual actions may not appear malicious.

This is useful for detecting:

- Fileless attacks
- Multi-stage intrusions
- New malware variants
- Unusual process chains

### MITRE ATT&CK Mapping

EDR detections may be mapped to MITRE ATT&CK tactics and techniques.

Example:

- Tactic: Persistence
- Technique: Scheduled Task/Job
- Technique ID: T1053

MITRE ATT&CK mapping helps analysts understand the possible attack stage and attacker objective.

---

## Detection Details

An EDR alert may contain:

- Severity
- Detection time
- Hostname
- Username
- Process name
- Parent process
- Command line
- Triggering file
- File hash
- Network connection
- MITRE ATT&CK tactic and technique

![EDR detections page showing alert fields](images/edr-detections-page.png)

---

## What Happens After a Detection?

When an alert is generated, the SOC analyst must acknowledge and prioritise it.

Common severity levels include:

- Critical
- High
- Medium
- Low
- Informational

Higher-severity alerts are generally investigated first. However, the analyst should also consider:

- Whether the affected system is business-critical
- Whether privileged accounts are involved
- Whether the threat is still active
- Whether multiple endpoints are affected
- Whether lateral movement may be occurring

The analyst then reviews the available evidence and determines whether the alert is a false positive or a true positive.

### False Positive

A false positive is legitimate activity incorrectly identified as suspicious.

Example:

An authorised administrator runs a PowerShell maintenance script that triggers an alert.

### True Positive

A true positive is an alert that represents genuine malicious activity.

Example:

A Word document launches an obfuscated PowerShell command and downloads an external payload.

---

## Basic EDR Investigation Workflow

    1. Review the alert severity
    2. Identify the affected endpoint
    3. Identify the involved user
    4. Examine the triggering process
    5. Review the parent-child process relationship
    6. Inspect the command line
    7. Check related files and hashes
    8. Review network connections
    9. Examine file and registry modifications
    10. Determine whether the activity is expected
    11. Classify the alert
    12. Take response action if necessary
    13. Escalate and document the investigation

---

## 3. Response

EDR provides both automated and manual response capabilities.

Common response actions include:

- Isolating the host
- Terminating a process
- Quarantining a file
- Blocking an indicator
- Accessing the endpoint remotely
- Running investigation scripts
- Collecting forensic artefacts

---

## Host Isolation

Host isolation disconnects the affected endpoint from the wider network while maintaining communication with the EDR platform.

This may prevent:

- Lateral movement
- Malware propagation
- Command-and-control communication
- Data exfiltration
- Additional payload downloads

Host isolation should be used carefully because isolating a critical server may disrupt business operations.

---

## Process Termination

An analyst may terminate a malicious process without isolating the entire endpoint.

Example:

A suspicious PowerShell process is actively downloading a payload.

Terminating a process may stop the immediate activity, but analysts must confirm that the process is not required for legitimate operations.

---

## File Quarantine

Quarantine moves a suspicious file into an isolated location where it cannot execute normally.

The analyst may then:

- Review the file
- Calculate its hash
- Submit it for further analysis
- Permanently delete it
- Restore it if confirmed legitimate

---

## Remote Access

EDR platforms may allow analysts to remotely access an endpoint shell.

Remote response can be used to:

- Examine running processes
- Search directories
- Review files
- Check network connections
- Run investigation scripts
- Collect evidence
- Remove malicious components

![EDR real-time response console](images/edr-rtr-console.png)

Remote actions must follow organisational procedures because commands executed on a live endpoint may affect operations or evidence integrity.

---

## Artefact Collection

EDR can collect forensic artefacts without requiring physical access to the endpoint.

Common artefacts include:

- Memory dumps
- Event logs
- Registry hives
- Suspicious files
- Folder contents
- Process information
- Network connection records

Artefacts may be used for:

- Incident investigation
- Root-cause analysis
- Timeline reconstruction
- Forensic examination
- Legal reporting

---

## What Is Telemetry?

Telemetry is the detailed data collected by the EDR agent from an endpoint.

Telemetry provides the evidence required to detect threats and investigate alerts.

### Process Telemetry

EDR may record:

- Process name
- Process ID
- Parent process
- Command line
- Execution path
- File hash
- User account
- Start and termination times

### Network Telemetry

EDR may record:

- Destination IP address
- Destination domain
- Destination port
- Protocol
- Connection time
- Process responsible for the connection

This can help identify command-and-control activity, lateral movement, unusual port usage, and data exfiltration.

### Command-Line Telemetry

EDR records commands executed through tools such as:

- PowerShell
- Command Prompt
- Bash
- Administrative utilities

Command-line telemetry may reveal:

- Encoded commands
- Obfuscated scripts
- Payload downloads
- Account creation
- Security control modifications
- Persistence commands

### File and Folder Telemetry

EDR may record:

- File creation
- File modification
- File deletion
- File path
- File hash
- Process responsible for the change

### Registry Telemetry

Registry modifications may indicate:

- Persistence
- Security control changes
- Startup execution
- System configuration changes

Registry activity must be interpreted carefully because legitimate applications also modify the registry.

---

## Antivirus vs EDR

Traditional antivirus and EDR both protect endpoints, but their capabilities differ.

| Capability | Traditional Antivirus | EDR |
|---|---|---|
| Known malware signatures | Yes | Yes |
| Continuous monitoring | Limited | Yes |
| Behavioural detection | Limited | Yes |
| Process-tree visibility | Limited | Yes |
| Command-line visibility | Limited | Yes |
| Historical telemetry | Limited | Yes |
| Threat hunting | Limited | Yes |
| Host isolation | Usually unavailable | Yes |
| Remote investigation | Usually unavailable | Yes |
| Artefact collection | Usually unavailable | Yes |

Traditional antivirus mainly detects known malicious files.

EDR monitors the complete behaviour of the endpoint and may identify attacks that use legitimate applications.

![Antivirus and EDR comparison](images/antivirus-vs-edr.png)

---

## Example Attack Chain

    Phishing email
        ↓
    Malicious Word document
        ↓
    User opens the document
        ↓
    Malicious macro executes
        ↓
    winword.exe launches powershell.exe
        ↓
    Obfuscated PowerShell command runs
        ↓
    Second-stage payload is downloaded
        ↓
    Payload is injected into svchost.exe
        ↓
    Outbound connection is established
        ↓
    Attacker gains remote access

Traditional antivirus may fail because the files and legitimate processes do not match known malicious signatures.

EDR can correlate the complete sequence and identify:

- The malicious document
- The Word-to-PowerShell process relationship
- The obfuscated command
- The downloaded payload
- The process injection
- The outbound connection

---

## EDR Architecture

A basic EDR architecture contains:

1. Endpoint agents
2. A central EDR console

### Endpoint Agent

The EDR agent is installed on the monitored endpoint.

It collects telemetry, performs basic detections, sends information to the central console, and receives response instructions.

### Central EDR Console

The central console:

- Receives telemetry from agents
- Correlates endpoint events
- Applies detection rules
- Uses threat intelligence
- Generates alerts
- Displays process trees
- Stores historical data
- Supports remote response

![Central EDR dashboard](images/edr-dashboard.png)

### Basic Data Flow

    Endpoint activity occurs
            ↓
    The EDR agent collects telemetry
            ↓
    Telemetry is sent to the central console
            ↓
    Detection logic analyses the activity
            ↓
    An alert is generated
            ↓
    The SOC analyst investigates
            ↓
    Response action is taken if required

---

## EDR and Other Security Tools

EDR is part of a wider security ecosystem.

Other security solutions may include:

- Firewalls
- Email security gateways
- Data Loss Prevention systems
- Identity and Access Management systems
- Network monitoring solutions
- Threat intelligence platforms
- SIEM platforms

EDR provides deep endpoint visibility, while SIEM provides broader visibility across multiple data sources.

An EDR alert may be sent to a SIEM and correlated with:

- Firewall logs
- Authentication events
- Email alerts
- VPN activity
- Cloud logs
- Threat intelligence

The SIEM may become the central investigation platform, while the EDR provides detailed endpoint evidence and response capabilities.

---

## Example SOC Analysis

### What Was Observed?

Microsoft Word launched PowerShell after a document was opened.

### Where Was It Observed?

The activity was detected on a Windows endpoint monitored by an EDR agent.

### Which Entities Were Involved?

    Parent process: winword.exe
    Child process: powershell.exe
    User: To be confirmed
    Hostname: To be confirmed
    Document: To be confirmed
    Command line: To be reviewed

### Why Is It Suspicious?

Microsoft Word does not normally launch PowerShell during standard document use. This may indicate malicious macro execution.

### What Evidence Supports It?

- Unusual parent-child process relationship
- PowerShell execution after document activity
- Possible encoded or obfuscated command
- Possible payload download
- Possible outbound connection

### What Is the Possible Risk?

The activity could result in:

- Malware execution
- Remote access
- Persistence
- Credential theft
- Data exfiltration
- Lateral movement

### What Should Be Done Next?

- Review the complete PowerShell command
- Identify the document source
- Check file hashes
- Review network connections
- Search for the same indicators across other endpoints
- Confirm whether the activity was authorised
- Isolate the endpoint if the threat remains active

### Should It Be Escalated?

Yes, if the PowerShell execution is unauthorised, obfuscated, connected to an unknown destination, or followed by additional suspicious activity.

---

## Final SOC Summary

Endpoint Detection and Response provides continuous endpoint monitoring, detailed telemetry, behavioural detection, and remote response capabilities. Unlike traditional antivirus, EDR can detect suspicious relationships between processes, analyse command-line activity, reconstruct attack timelines, and identify attacks involving legitimate system utilities. After an alert is generated, the SOC analyst must review the evidence, determine whether the activity is a false positive or true positive, and take proportionate response actions. EDR provides deep endpoint visibility but should be integrated with SIEM, network monitoring, email security, and identity systems to provide broader organisational coverage.

---

## Lessons Learned

- EDR provides visibility, detection, and response capabilities.
- Endpoint agents collect telemetry and send it to a central console.
- Process relationships can reveal suspicious behaviour.
- Legitimate tools may be abused by threat actors.
- Command-line activity is important investigation evidence.
- EDR uses behavioural detection, anomaly detection, IOC matching, and machine learning.
- Alert severity supports prioritisation but does not replace analyst judgment.
- Analysts must distinguish false positives from true positives.
- EDR can isolate hosts, terminate processes, quarantine files, and collect artefacts.
- EDR provides endpoint-level visibility and works alongside SIEM and other security tools.
