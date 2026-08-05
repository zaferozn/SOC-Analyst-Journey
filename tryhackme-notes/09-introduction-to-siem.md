# Introduction to SIEM

## Overview

SIEM stands for **Security Information and Event Management**.

A SIEM collects logs from different systems, places them in one platform, standardizes their format, correlates related events, and generates alerts for suspicious activity.

## Log Sources

### Host-Centric Logs

These logs record activity occurring on endpoints and servers.

Examples:

- Login attempts
- File access
- Process execution
- PowerShell activity
- Registry changes

Common sources include Windows and Linux systems.

### Network-Centric Logs

These logs record communication between systems and networks.

Examples:

- SSH connections
- VPN access
- Web traffic
- FTP activity
- Firewall events

Common sources include firewalls, routers, IDS/IPS, and VPN gateways.

## Why SIEM Is Needed

Reviewing logs separately creates several problems:

- Logs are distributed across many systems.
- Different devices use different log formats.
- Manual analysis is slow.
- Individual events may lack context.
- Important activity may be missed because of high log volume.

A SIEM solves these problems by centralizing and correlating the logs.

## Core SIEM Features

### Centralized Collection

Logs from endpoints, servers, firewalls, and applications are collected in one platform.

### Parsing and Normalization

Parsing separates raw logs into fields such as:

- Timestamp
- Username
- Source IP
- Hostname
- Event ID
- Process name

Normalization presents logs from different sources in a consistent format.

### Correlation

SIEM connects related events to identify suspicious patterns.

For example:

1. A user connects through VPN from an unusual IP.
2. The user accesses sensitive files.
3. PowerShell is executed.
4. The system makes an outbound connection.

Together, these events may indicate compromised credentials or data exfiltration.

### Alerting

Detection rules generate alerts when specific conditions are met.

Examples:

- Multiple failed login attempts
- Successful login after repeated failures
- Event log clearing
- Suspicious command execution
- Large outbound data transfer

### Dashboards

Dashboards may display:

- Recent alerts
- Failed logins
- Triggered rules
- Event counts
- Source IP addresses
- System health

## Log Ingestion Methods

Common ingestion methods include:

- Endpoint agents or forwarders
- Syslog
- Manual log upload
- Port-based forwarding

In my lab, the Wazuh agent sends Linux events to the Wazuh manager.

## Alert Investigation

When an alert is triggered, the analyst should review:

- The detection rule
- The original raw log
- The affected user or host
- Source and destination IP addresses
- Related events
- Activity before and after the alert
- Possible legitimate explanations

The analyst then determines whether the alert is:

- False positive
- True positive
- Confirmed security incident

Possible actions include rule tuning, further investigation, escalation, account disabling, host isolation, or blocking a suspicious IP.

## Connection to My Wazuh Lab

My Wazuh environment applies the same SIEM process:

1. Ubuntu generates authentication logs.
2. The Wazuh agent collects the logs.
3. Wazuh parses the events.
4. Detection rules evaluate the activity.
5. Alerts are displayed for investigation.

Examples include failed SSH logins, invalid users, successful authentication, sudo activity, and brute-force alerts.

## Key Takeaways

- SIEM centralizes security logs.
- Parsing extracts important fields.
- Normalization creates a consistent structure.
- Correlation connects related activity.
- Detection rules generate alerts.
- Alerts are starting points, not proof of compromise.
- Analysts must validate alerts using raw logs and context.

## Source Note

These notes are a paraphrased summary of the TryHackMe **Introduction to SIEM** room.
