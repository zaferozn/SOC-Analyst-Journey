# Lab 029 - Multi-Source Incident Correlation

## Executive Summary

This lab demonstrates a SOC analyst workflow by correlating multiple security telemetry sources into one incident timeline.

Data sources:

- Linux authentication logs
- Wazuh SIEM alerts
- Suricata IDS alerts
- PCAP network analysis
- DNS analysis
- HTTP/TLS traffic observation

The objective was to reconstruct suspicious activity by combining endpoint and network evidence.

Incident flow:

    Network Reconnaissance
            |
            ↓
    Suricata SYN Scan Detection
            |
            ↓
    Failed SSH Authentication
            |
            ↓
    Successful SSH Authentication
            |
            ↓
    Privileged Activity
            |
            ↓
    Wazuh Correlation
            |
            ↓
    Network Context Analysis


---

# Objective

The investigation focused on:

- Identifying the source host
- Detecting reconnaissance behavior
- Reviewing SSH authentication activity
- Confirming successful account access
- Investigating privilege execution
- Correlating events with Wazuh
- Validating network activity through PCAP


---

# Environment

Source:

192.168.64.1


Target:

ubuntu-agent

192.168.64.10


Wazuh Server:

wazuh-server

192.168.64.9


---

# Tools Used

- Ubuntu Linux
- journalctl
- Wazuh
- Suricata
- tshark
- Wireshark
- tcpdump
- Nmap


---

# Phase 1 - Evidence Collection Preparation

Monitoring components were verified before starting the investigation.

Validated:

- Wazuh agent status
- Suricata service status
- SSH service status
- Packet capture collection


![Lab 029 Evidence Collection](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/00-lab029-evidence-collection-start.png)


---

# Phase 2 - Network Reconnaissance Detection

A TCP SYN scan was performed against the endpoint.

Source:

192.168.64.1


Target:

192.168.64.10


Observed service:

22/tcp open ssh


Analysis:

The source host performed reconnaissance activity to identify available services.

This represented the initial stage of the investigation.


![Lab 029 SYN Port Scan](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/01-lab029-syn-port-scan.png)


---

# Phase 3 - Failed SSH Authentication Investigation

SSH authentication attempts were observed after reconnaissance activity.

Observed:

Invalid user lab029bad from 192.168.64.1


Failed authentication:

Failed password for invalid user lab029bad


Analysis:

The source attempted access using an invalid account.

The event required investigation because it occurred after network scanning activity.


![Lab 029 Failed SSH Authentication](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/02-lab029-failed-ssh-authentication.png)


---

# Phase 4 - Successful SSH Authentication

A successful SSH authentication occurred from the same source.

User:

analyst


Source:

192.168.64.1


Analysis:

The investigation identified a transition:

Failed authentication

↓

Successful authentication


This increased the importance of the event because the same source gained valid access.


![Lab 029 Successful SSH Authentication](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/03-lab029-successful-ssh-authentication.png)


---

# Phase 5 - Privilege Activity

After successful authentication, privileged activity was observed.

Command:

sudo whoami


Result:

root


Evidence:

/tmp/lab029-privileged-activity.txt


Analysis:

The analyst account executed a command with elevated privileges.


![Lab 029 Privilege Activity](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/04-lab029-privilege-activity.png)


---

# Phase 6 - Network Communication Observation

PCAP analysis was performed to validate network activity.

Observed:

- SSH communication
- DNS queries
- HTTP/TLS sessions


The network communication was treated as supporting telemetry.

No malicious determination was made without additional threat intelligence enrichment.


![Lab 029 DNS HTTP Communication](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/05-lab029-dns-http-communication.png)


---

# Phase 7 - PCAP Preservation

The packet capture was preserved for investigation.

File:

lab-029-multisource-incident.pcap


![Lab 029 PCAP Preservation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/06-lab029-pcap-preservation.png)


---

# Phase 8 - PCAP Basic Analysis

PCAP statistics were reviewed.

Analysis included:

- Packet count
- Communication pairs
- Network conversations


![Lab 029 PCAP Basic Analysis](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/07-lab029-pcap-basic-analysis.png)


---

# Phase 9 - PCAP SSH Analysis

Observed communication:

192.168.64.1 → 192.168.64.10:22


Analysis:

The packet capture confirmed SSH communication matching Linux authentication events.


![Lab 029 PCAP SSH Analysis](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/08-lab029-pcap-ssh-analysis.png)


---

# Phase 10 - DNS Analysis

Observed DNS activity:

example.com


Resolved addresses:

104.20.23.154

172.66.147.243


Analysis:

DNS resolution was observed during network analysis.

The domain was recorded as network context only and was not classified as malicious.


![Lab 029 PCAP DNS Analysis](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/09-lab029-pcap-dns-analysis.png)


---

# Phase 11 - HTTP/TLS Analysis

Observed:

HTTP request:

GET /


TLS SNI:

example.com


Analysis:

Outbound communication was observed from the endpoint.

This provided additional network context for the timeline.


![Lab 029 HTTP TLS Analysis](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/10-lab029-pcap-http-tls-analysis.png)


---

# Phase 12 - Wazuh SSH and Sudo Correlation

Wazuh correlated endpoint activity.

Observed:

Successful SSH authentication

Rule:

5501


Privilege activity:

Rule:

5402


Analysis:

Wazuh confirmed:

- Authentication activity
- Valid account usage
- Privileged execution


![Lab 029 Wazuh Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/11-lab029-wazuh-ssh-sudo-correlation.png)


---

# Phase 13 - Suricata Correlation

Suricata detected:

LOCAL TCP SYN Port Scan Detected


Source:

192.168.64.1


Destination:

192.168.64.10


Analysis:

Suricata provided network-level evidence supporting the reconnaissance stage.


![Lab 029 Suricata Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-029/12-lab029-suricata-syn-scan-correlation.png)


---

# Final Incident Timeline

21:04:43 UTC

Suricata:

TCP SYN Port Scan Detected


↓

21:07:26 UTC

SSH:

Invalid user lab029bad


↓

21:25:12 UTC

SSH:

Successful login as analyst


↓

21:26:34 UTC

Privilege:

sudo root execution


↓

21:28:06 UTC

Network:

DNS and HTTP/TLS communication observed


↓

Wazuh:

Endpoint correlation completed


---

# Final SOC Summary

The investigation successfully correlated multiple telemetry sources.

Evidence:

- Suricata detected reconnaissance
- Linux logs confirmed authentication activity
- Wazuh correlated endpoint events
- PCAP validated network communication

The activity sequence showed reconnaissance followed by authentication attempts and privileged execution.

The DNS and HTTP/TLS traffic was treated as additional network context rather than confirmed malicious infrastructure.


---

# Risk Assessment

Risk Level:

Medium


Reason:

- Network scanning detected
- Invalid account authentication attempted
- Valid account access achieved
- Privileged commands executed


---

# MITRE ATT&CK Mapping

Network Service Scanning:

T1046


Valid Accounts:

T1078


Sudo and Sudo Caching:

T1548.003


---

# Lessons Learned

- SOC analysts must correlate multiple data sources.
- IDS alerts require context.
- Authentication events should be investigated as a sequence.
- PCAP analysis validates network behavior.
- Not every observed domain is malicious.
- Timeline reconstruction improves incident understanding.
