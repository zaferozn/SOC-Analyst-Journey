# Lab 028 - Wazuh Suricata Alert Triage

## Executive Summary

This lab investigated a controlled TCP SYN port scan through an integrated Suricata and Wazuh monitoring pipeline.

A SYN scan was intentionally generated from the Wazuh server at `192.168.64.9` against the monitored Ubuntu endpoint at `192.168.64.10`.

Suricata detected the activity using local signature ID `1000001`, classified it as `Detection of a Network Scan`, and recorded the event in `/var/log/suricata/eve.json`.

The Wazuh agent ingested the Suricata JSON telemetry and forwarded it to the Wazuh manager, where Wazuh rule `86601` processed the detection under the `ids` and `suricata` rule groups.

The original Suricata event and the centralized Wazuh alert were correlated using:

- source IP address
- destination IP address
- source port
- destination port
- protocol
- Suricata signature ID
- Suricata signature
- timestamps
- monitored agent
- original log source

The final analyst disposition was:

**True Positive - Benign / Authorized Test Activity**

No escalation was required because the scan was intentionally generated as part of this controlled laboratory exercise.

---

## Objective

The objective of this lab was to investigate a Suricata network detection after it had been ingested into Wazuh and demonstrate an end-to-end IDS-to-SIEM alert triage workflow.

The investigation focused on the following questions:

- What activity generated the alert?
- Which source and destination systems were involved?
- Which Suricata signature detected the activity?
- Was the original Suricata telemetry preserved after Wazuh ingestion?
- Which Wazuh rule processed the event?
- Could the Suricata and Wazuh events be correlated?
- Was the detection accurate?
- Was the underlying activity malicious or authorized?
- Was escalation required?

---

## Environment / Data Source

### Wazuh Server

- Hostname: `wazuh-server`
- IPv4 address: `192.168.64.9`

### Monitored Ubuntu Endpoint

- Hostname: `ubuntu-agent`
- IPv4 address: `192.168.64.10`
- Wazuh Agent ID: `001`

### Tools

- Suricata IDS
- Wazuh Agent
- Wazuh Manager
- Nmap
- jq
- Linux CLI

### Log Sources

Suricata:

`/var/log/suricata/eve.json`

Wazuh:

`/var/ossec/logs/alerts/alerts.json`

### Suricata Interface

`enp0s1`

### Investigation Time Window

Primary controlled scan:

`2026-08-21 approximately 19:13 UTC`

---

# Investigation Workflow

    Wazuh Server
    192.168.64.9
          |
          | Controlled TCP SYN Scan
          v
    Ubuntu Agent
    192.168.64.10
          |
          v
    Suricata IDS
          |
          v
    eve.json
          |
          v
    Wazuh Agent
          |
          v
    Wazuh Manager
          |
          v
    Wazuh Rule 86601
          |
          v
    Centralized IDS Alert
          |
          v
    SOC Analyst Triage
          |
          v
    True Positive - Authorized Activity
          |
          v
    No Escalation

---

# Evidence

## Evidence 01 - Suricata and Wazuh Pipeline Baseline

Before generating test activity, the monitoring pipeline was verified.

The following conditions were confirmed:

- Host: `ubuntu-agent`
- IPv4 address: `192.168.64.10`
- Suricata service: `active`
- `/var/log/suricata/eve.json`: present
- Wazuh agent service: `active`

![01 - Suricata Wazuh Pipeline Baseline](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/01-suricata-wazuh-pipeline-baseline.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/01-suricata-wazuh-pipeline-baseline.png

This established that both the IDS and Wazuh collection components were operational before suspicious traffic was generated.

---

## Evidence 02 - Initial Controlled SYN Scan

An initial TCP SYN scan was performed against:

`192.168.64.10`

Command:

    sudo nmap -sS -Pn -p 1-1000 192.168.64.10

Nmap reported:

    999 closed tcp ports
    22/tcp open ssh

![02 - Controlled SYN Scan](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/02-controlled-syn-scan.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/02-controlled-syn-scan.png

This first test was performed locally.

Because a local scan did not provide the clearest source-to-destination relationship for the investigation, a second scan was generated from the separate Wazuh server.

---

## Evidence 03 - External Scan Source Verification

Before generating the external scan, the source system was verified.

Hostname:

`wazuh-server`

IPv4 address:

`192.168.64.9`

![03 - Verify External Scan Source](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/03-verify-external-scan-source.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/03-verify-external-scan-source.png

This confirmed that the second scan would originate from a separate host and therefore create a clean source-to-target relationship.

---

## Evidence 04 - External TCP SYN Scan

A controlled TCP SYN scan was generated from:

`192.168.64.9`

against:

`192.168.64.10`

Command:

    sudo nmap -sS -Pn -p 1-1000 192.168.64.10

The scan identified:

    999 closed tcp ports
    22/tcp open ssh

![04 - External SYN Scan](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/04-external-syn-scan.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/04-external-syn-scan.png

This provided the controlled suspicious network activity used throughout the remainder of the investigation.

---

## Evidence 05 - Suricata Detection

Suricata generated multiple alerts corresponding to SYN packets transmitted from the Wazuh server toward different TCP ports on the monitored Ubuntu endpoint.

Relevant fields included:

    src_ip: 192.168.64.9
    dest_ip: 192.168.64.10
    proto: TCP

    signature_id: 1000001

    signature:
    LOCAL TCP SYN Port Scan Detected

    category:
    Detection of a Network Scan

    severity: 3

![05 - Suricata Alert From Wazuh Server](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/05-suricata-alert-from-wazuh-server.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/05-suricata-alert-from-wazuh-server.png

The large number of SYN packets directed toward different TCP ports was consistent with network reconnaissance behavior.

---

## Evidence 06 - Suricata Alert Field Extraction

A single Suricata event was extracted with `jq` to isolate the fields most relevant to SOC triage.

Observed event:

    timestamp:
    2026-08-21T19:13:13.775519+0000

    event_type:
    alert

    in_iface:
    enp0s1

    src_ip:
    192.168.64.9

    src_port:
    54715

    dest_ip:
    192.168.64.10

    dest_port:
    283

    proto:
    TCP

    action:
    allowed

    signature_id:
    1000001

    signature:
    LOCAL TCP SYN Port Scan Detected

    category:
    Detection of a Network Scan

    severity:
    3

    direction:
    to_server

![06 - Suricata Alert Fields](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/06-suricata-alert-fields.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/06-suricata-alert-fields.png

This provided a clean representation of the original IDS telemetry before centralized SIEM processing.

---

## Evidence 07 - Suricata Alert Received by Wazuh

The centralized Wazuh alert log was searched for the Suricata signature:

`LOCAL TCP SYN Port Scan Detected`

The corresponding event was successfully located in:

`/var/ossec/logs/alerts/alerts.json`

Relevant Wazuh fields included:

    rule.description:
    Suricata: Alert - LOCAL TCP SYN Port Scan Detected

    rule.id:
    86601

    rule.level:
    3

    rule.groups:
    ids
    suricata

    agent.id:
    001

    agent.name:
    ubuntu-agent

    agent.ip:
    192.168.64.10

![07 - Wazuh Suricata Alert](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/07-wazuh-suricata-alert.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/07-wazuh-suricata-alert.png

This confirmed successful transmission of the Suricata telemetry through the IDS-to-SIEM pipeline.

---

## Evidence 08 - Wazuh Suricata Alert Field Extraction

The Wazuh event was extracted with `jq` to isolate the fields required for analyst triage.

Observed Wazuh alert:

    wazuh_timestamp:
    2026-08-21T19:13:17.199+0000

    rule_id:
    86601

    rule_level:
    3

    rule_description:
    Suricata: Alert - LOCAL TCP SYN Port Scan Detected

    rule_groups:
    ids
    suricata

    agent_id:
    001

    agent_name:
    ubuntu-agent

    agent_ip:
    192.168.64.10

    source_ip:
    192.168.64.9

    source_port:
    54715

    destination_ip:
    192.168.64.10

    destination_port:
    283

    protocol:
    TCP

    suricata_signature_id:
    1000001

    suricata_signature:
    LOCAL TCP SYN Port Scan Detected

    suricata_category:
    Detection of a Network Scan

    suricata_severity:
    3

    log_source:
    /var/log/suricata/eve.json

![08 - Wazuh Suricata Alert Fields](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/08-wazuh-suricata-alert-fields.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/08-wazuh-suricata-alert-fields.png

This demonstrated that Wazuh preserved the important Suricata network telemetry while adding centralized SIEM context.

---

## Evidence 09 - Suricata to Wazuh Event Correlation

The original Suricata telemetry was compared with the centralized Wazuh alert.

The following fields were consistent between the two records:

    SURICATA                     WAZUH

    Source IP
    192.168.64.9          --->   192.168.64.9

    Destination IP
    192.168.64.10         --->   192.168.64.10

    Source Port
    54715                 --->   54715

    Destination Port
    283                   --->   283

    Protocol
    TCP                   --->   TCP

    Signature ID
    1000001               --->   1000001

    Signature
    LOCAL TCP SYN
    Port Scan Detected    --->   LOCAL TCP SYN
                                 Port Scan Detected

Suricata event timestamp:

`2026-08-21T19:13:13.775519+0000`

Wazuh alert timestamp:

`2026-08-21T19:13:17.199+0000`

The difference between the original Suricata event and the Wazuh alert was approximately:

`3.4 seconds`

![09 - Suricata Wazuh Event Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/09-suricata-wazuh-event-correlation.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/09-suricata-wazuh-event-correlation.png

This provided direct evidence that the original IDS event had been successfully processed and represented inside Wazuh.

---

## Evidence 10 - Final Analyst Triage

After validating the detection and reviewing the known test context, the alert was formally classified.

Observed activity:

`Controlled TCP SYN port scan`

Source:

`192.168.64.9 - wazuh-server`

Target:

`192.168.64.10 - ubuntu-agent`

Suricata detection:

    Suricata SID: 1000001
    Signature: LOCAL TCP SYN Port Scan Detected
    Category: Detection of a Network Scan
    Severity: 3

Wazuh correlation:

    Wazuh Rule ID: 86601
    Rule Level: 3
    Groups: ids, suricata

Analyst verdict:

`True Positive - Benign / Authorized Test Activity`

Escalation:

`No escalation required`

![10 - Final Analyst Triage](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/10-final-analyst-triage.png)

Full image URL:

https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-028/10-final-analyst-triage.png

---

# Analysis

## What Was Observed?

A TCP SYN port scan targeting multiple TCP ports on the monitored Ubuntu endpoint was observed.

The scan originated from:

`192.168.64.9`

and targeted:

`192.168.64.10`

The activity consisted of SYN packets directed toward a large number of destination ports within a very short period.

---

## Where Was It Observed?

The activity was initially detected by Suricata and recorded in:

`/var/log/suricata/eve.json`

The event was subsequently collected by the Wazuh agent and forwarded to the Wazuh manager.

The centralized alert was stored in:

`/var/ossec/logs/alerts/alerts.json`

---

## Which Host, IP Address, or Log Source Was Involved?

### Source Host

    Hostname: wazuh-server
    IP Address: 192.168.64.9

### Target Host

    Hostname: ubuntu-agent
    IP Address: 192.168.64.10
    Wazuh Agent ID: 001

### Original IDS Log Source

`/var/log/suricata/eve.json`

### Centralized SIEM Alert Source

`/var/ossec/logs/alerts/alerts.json`

---

## Why Is It Suspicious?

TCP SYN scanning is commonly associated with network reconnaissance.

An attacker may conduct this type of scan to identify:

- active systems
- open TCP ports
- exposed services
- SSH services
- web services
- remote administration interfaces
- vulnerable applications
- potential attack surfaces

A large number of SYN packets directed toward different destination ports within a short period is therefore suspicious when the activity is unexpected.

Suricata correctly categorized the observed traffic as:

`Detection of a Network Scan`

---

## What Evidence Supports the Detection?

The investigation produced multiple independent evidence points:

1. Nmap intentionally generated a TCP SYN scan.

2. The scanning source was verified as `wazuh-server`.

3. The source IP was `192.168.64.9`.

4. The destination IP was `192.168.64.10`.

5. Suricata observed TCP traffic from the expected source.

6. Suricata generated signature ID `1000001`.

7. The Suricata signature was:

   `LOCAL TCP SYN Port Scan Detected`

8. Suricata categorized the activity as:

   `Detection of a Network Scan`

9. The original event was written to:

   `/var/log/suricata/eve.json`

10. Wazuh received the Suricata event.

11. Wazuh processed the event using rule `86601`.

12. The Wazuh rule belonged to:

    `ids`

    and

    `suricata`

13. Wazuh preserved the source IP.

14. Wazuh preserved the destination IP.

15. Wazuh preserved the source port.

16. Wazuh preserved the destination port.

17. Wazuh preserved the TCP protocol.

18. Wazuh preserved Suricata signature ID `1000001`.

19. Wazuh preserved the original signature description.

20. Wazuh preserved the original log source.

21. The Suricata and Wazuh timestamps were consistent with the same event-processing pipeline.

These observations provided sufficient evidence to correlate the original IDS detection with the centralized SIEM alert.

---

# Raw IDS Event vs SIEM Alert

One of the main objectives of this lab was to understand the relationship between original IDS telemetry and centralized SIEM monitoring.

## Suricata Layer

Suricata supplied the network detection context:

    timestamp
    event_type
    src_ip
    src_port
    dest_ip
    dest_port
    proto
    direction
    signature_id
    signature
    category
    severity

Suricata primarily answered:

**What happened on the network?**

---

## Wazuh Layer

Wazuh preserved the Suricata event while adding centralized monitoring context:

    rule.id
    rule.level
    rule.description
    rule.groups

    agent.id
    agent.name
    agent.ip

    manager.name
    location

Wazuh primarily answered:

**Which monitored asset generated the telemetry, which SIEM rule processed it, and how can the event be centrally investigated?**

---

## IDS-to-SIEM Pipeline

    Network Activity
          |
          v
    Suricata
          |
          | Detection
          v
    eve.json
          |
          | Log ingestion
          v
    Wazuh Agent
          |
          | Forwarding
          v
    Wazuh Manager
          |
          | Rule processing
          v
    Wazuh Alert
          |
          v
    SOC Analyst

Suricata and Wazuh therefore performed complementary roles.

Suricata provided network detection.

Wazuh provided:

- centralized ingestion
- endpoint association
- SIEM rule processing
- alert storage
- centralized analyst visibility

---

# Detection Assessment

## Was the Detection Accurate?

Yes.

A real TCP SYN port scan occurred.

Suricata correctly identified that activity.

Therefore, the detection itself was a:

**True Positive**

---

## Was the Activity Malicious?

No.

The SYN scan was intentionally generated as part of an authorized laboratory exercise.

The source was known.

The destination was known.

The purpose was known.

The activity was expected.

Therefore, the underlying activity was:

**Benign / Authorized**

---

## Why Was This Not a False Positive?

A false positive would mean the security control reported a scan when the relevant behavior had not actually occurred.

That did not happen.

A real SYN scan occurred.

Suricata detected the real scan correctly.

Therefore:

    Detection Accuracy:
    True Positive

    Activity Context:
    Benign / Authorized

    Final Disposition:
    True Positive - Benign / Authorized Test Activity

---

# Risk

In a production environment, unexplained TCP SYN scanning could indicate reconnaissance performed by an attacker.

Possible risks include:

- network mapping
- service discovery
- identification of open ports
- SSH service discovery
- identification of exposed applications
- identification of vulnerable services
- discovery of administrative interfaces
- preparation for exploitation
- preparation for lateral movement

Port scanning alone does not prove that a system has been compromised.

However, unexpected scanning can represent an early-stage indicator of malicious activity and should be investigated within the broader security context.

---

# Recommended Next Steps

If similar activity were observed in a production SOC environment, the analyst should:

1. Identify the scanning source IP.

2. Determine whether the source belongs to an authorized vulnerability scanner.

3. Determine whether the source belongs to an administrator or security team.

4. Verify whether penetration testing or maintenance activity was scheduled.

5. Identify all destination systems contacted by the source.

6. Identify all destination ports contacted during the scan.

7. Review additional Suricata detections involving the same source.

8. Review Wazuh endpoint telemetry from the target system.

9. Search for SSH authentication attempts following the scan.

10. Search for failed authentication attempts.

11. Search for successful authentication activity.

12. Review privileged activity following any successful authentication.

13. Search for exploitation-related network alerts.

14. Review firewall telemetry if available.

15. Determine whether the source contacted additional internal systems.

16. Establish a timeline of related activity.

17. Escalate if the scanning source cannot be explained.

18. Escalate if reconnaissance is followed by exploitation, authentication abuse, or other suspicious behavior.

---

# Escalation Decision

## Lab Context

Escalation was not required.

Reasons:

- The source system was known.
- The source IP was known.
- The target was known.
- The activity was intentionally generated.
- The activity was authorized.
- The Suricata alert matched the known test.
- The Wazuh event matched the Suricata telemetry.
- No malicious follow-on activity was identified.

Final decision:

**No escalation required.**

---

## Production Context

Equivalent activity should be considered for escalation when:

- the source IP is unknown
- the source system is unauthorized
- multiple internal hosts are being scanned
- sensitive systems are targeted
- scanning is followed by exploitation attempts
- scanning is followed by authentication attacks
- the source is external or otherwise unexpected
- lateral movement indicators appear
- suspicious endpoint activity follows the reconnaissance

---

# MITRE ATT&CK Mapping

## T1046 - Network Service Discovery

The observed TCP SYN scan is consistent with:

**MITRE ATT&CK T1046 - Network Service Discovery**

Attackers may scan hosts and ports to identify available services that could be targeted during later stages of an intrusion.

In this lab, the behavior was intentionally generated for detection testing.

The ATT&CK mapping therefore describes the observed technique rather than malicious intent.

---

# Final Analyst Decision

    Alert Type:
    Network Reconnaissance / Port Scan

    Detection:
    True Positive

    Source:
    192.168.64.9

    Source Host:
    wazuh-server

    Target:
    192.168.64.10

    Target Host:
    ubuntu-agent

    Protocol:
    TCP

    Suricata SID:
    1000001

    Suricata Signature:
    LOCAL TCP SYN Port Scan Detected

    Suricata Category:
    Detection of a Network Scan

    Wazuh Rule:
    86601

    Wazuh Rule Level:
    3

    Wazuh Groups:
    ids, suricata

    Business Context:
    Authorized Laboratory Activity

    Security Impact:
    None

    Escalation:
    No

    Final Disposition:
    True Positive - Benign / Authorized

---

# Final SOC Summary

A controlled TCP SYN scan was generated from the Wazuh server at `192.168.64.9` against the monitored Ubuntu endpoint at `192.168.64.10`.

Suricata detected the network reconnaissance activity using local signature ID `1000001`, `LOCAL TCP SYN Port Scan Detected`, and recorded the event in `/var/log/suricata/eve.json`.

The Wazuh agent successfully ingested the Suricata telemetry and forwarded it to the Wazuh manager, where Wazuh rule `86601` processed the event under the `ids` and `suricata` rule groups.

The original Suricata event and centralized Wazuh alert were correlated using source and destination IP addresses, source and destination ports, TCP protocol, Suricata signature information, timestamps, monitored agent data, and original log source.

The detection was technically accurate because a real TCP SYN scan occurred. However, the activity was intentionally generated as part of an authorized laboratory exercise.

The alert was therefore classified as:

**True Positive - Benign / Authorized Test Activity**

No escalation was required.

---

# Lessons Learned

This lab demonstrated that an IDS alert should not be evaluated solely on whether a signature triggered.

A SOC analyst must determine:

- whether the activity actually occurred
- whether the detection accurately represents the behavior
- which systems were involved
- whether the activity was authorized
- whether additional suspicious behavior exists
- whether escalation is required

The lab also demonstrated the distinction between raw IDS telemetry and centralized SIEM monitoring.

Suricata generated detailed network detection information.

Wazuh successfully ingested that telemetry, associated it with the monitored endpoint, applied a centralized rule, and preserved the original Suricata fields for analyst investigation.

The most important analytical lesson was the distinction between:

    False Positive

and:

    True Positive - Benign / Authorized

The detection was accurate.

The activity was simply expected and authorized.

This distinction is important during SOC alert triage because a technically correct alert does not automatically represent malicious activity.

---

# Skills Demonstrated

- Suricata IDS monitoring
- Wazuh SIEM integration
- IDS-to-SIEM log ingestion
- Network alert triage
- JSON log analysis
- Suricata `eve.json` investigation
- Wazuh `alerts.json` investigation
- Nmap SYN scan analysis
- Source IP analysis
- Destination IP analysis
- Source port analysis
- Destination port analysis
- Protocol analysis
- Alert field extraction with `jq`
- Suricata signature analysis
- Wazuh rule analysis
- IDS-to-SIEM event correlation
- Timestamp correlation
- Centralized security monitoring
- Alert validation
- True-positive classification
- Benign-authorized classification
- Escalation decision-making
- Evidence-based SOC documentation

---

# SOC English Practice

## Sentence 1

The Suricata alert was successfully correlated with the corresponding Wazuh event using source and destination attributes, protocol information, signature metadata, and timestamps.

## Sentence 2

The IDS signature accurately identified a TCP SYN port scan originating from the Wazuh server and targeting the monitored Ubuntu endpoint.

## Sentence 3

The original Suricata telemetry was preserved after ingestion into Wazuh, allowing the analyst to validate the detection using centralized SIEM data.

## Sentence 4

Although the detection was a true positive, the underlying activity was classified as benign because the scan had been intentionally generated as part of an authorized laboratory exercise.

## Sentence 5

No escalation was required because the source system was known, the activity was expected, and no evidence of malicious follow-on behavior was identified.


# Interview Explanation

**Question: How would you investigate an IDS alert in a SIEM?**

I would first review the SIEM alert metadata, including the source and destination systems, ports, protocol, rule description, severity, and detection signature.

I would then identify the original log source and compare the SIEM event with the underlying IDS telemetry.

In this lab, I correlated a Suricata SYN-scan detection from `eve.json` with the corresponding Wazuh alert.

The source IP, destination IP, ports, TCP protocol, Suricata signature ID, signature description, and timestamps were consistent across both data sources.

After validating the activity, I reviewed the operational context and determined that the scan had been intentionally generated as part of an authorized test.

I therefore classified the event as a true positive with benign authorized activity and determined that escalation was unnecessary.


Final workflow:

    Controlled SYN Scan
            |
            v
    Suricata Detection
            |
            v
    eve.json
            |
            v
    Wazuh Agent
            |
            v
    Wazuh Manager
            |
            v
    Wazuh Rule 86601
            |
            v
    Centralized Alert
            |
            v
    Field Correlation
            |
            v
    Analyst Validation
            |
            v
    True Positive - Authorized Activity
            |
            v
    No Escalation
