# Lab 023 - Suricata IDS Alert Triage

## Executive Summary

This lab focused on deploying and validating Suricata as a network intrusion detection system and investigating a controlled TCP SYN scan.

Suricata was installed on the Ubuntu security monitoring host and initially failed to start because the default configuration referenced the non-existent `eth0` interface. The active network interface was identified as `enp0s1`, and the Suricata configuration was corrected.

The Emerging Threats Open ruleset was then installed and successfully loaded. A controlled TCP SYN scan was generated from the Mac host at `192.168.64.1` against the Ubuntu host at `192.168.64.10`.

Although Suricata initially observed thousands of TCP SYN packets, the enabled rules did not generate an alert. This represented a detection gap for the lab scenario.

A custom local Suricata rule was therefore created to detect repeated TCP SYN activity originating from a single source. After the rule was loaded and Suricata was restarted, the scan was repeated.

Suricata successfully generated multiple alerts with the signature:

`LOCAL TCP SYN Port Scan Detected`

The alert was investigated using both `fast.log` and the structured `eve.json` output.

Because the scan was intentionally generated inside a controlled laboratory environment, the final disposition was **Benign True Positive**.

---

## Objective

The objectives of this lab were to:

- Deploy Suricata IDS on an Ubuntu monitoring host
- Verify the active monitoring interface
- Install and validate an IDS ruleset
- Generate controlled TCP SYN scanning activity
- Determine whether Suricata generated an alert
- Identify a detection gap when no initial alert was produced
- Create and load a custom Suricata detection rule
- Generate and investigate a network-scan alert
- Compare human-readable and structured IDS evidence
- Assign an analyst disposition and escalation decision

---

## Environment / Data Source

**Monitored Host:** `ubuntu-agent`

**Monitored IP:** `192.168.64.10`

**Scan Source:** `192.168.64.1`

**Monitoring Interface:** `enp0s1`

**Operating System:** Ubuntu Linux

**IDS:** Suricata 7.0.3

**Ruleset:** Emerging Threats Open + custom local rule

**Traffic Generator:** Nmap

**Primary Log Sources:**

- `/var/log/suricata/fast.log`
- `/var/log/suricata/eve.json`
- `/var/log/suricata/suricata.log`

**Protocol:** TCP

**Scan Type:** SYN scan

---

## Step 1 - Suricata Installation

Suricata was installed on the Ubuntu monitoring host using:

```bash
sudo apt update
sudo apt install suricata -y
```

The installation was verified using:

```bash
suricata --build-info
```

The output confirmed:

```text
Suricata version 7.0.3 RELEASE
Detection enabled: yes
AF_PACKET support: yes
```

---

## Step 2 - Initial Service Failure

The initial Suricata service status was reviewed using:

```bash
sudo systemctl status suricata
```

The service initially showed:

```text
Active: failed
```

The failure was investigated with:

```bash
sudo journalctl -u suricata --no-pager -n 30
```

The relevant error was:

```text
Failure when trying to get MTU via ioctl for 'eth0': No such device
```

This showed that Suricata was configured to monitor an interface named `eth0`, but that interface did not exist on the host.

---

## Step 3 - Network Interface Identification

The active interfaces were reviewed with:

```bash
ip -br addr
```

Relevant output:

```text
lo       UNKNOWN        127.0.0.1/8
enp0s1   UP             192.168.64.10/24
```

The correct monitoring interface was therefore:

```text
enp0s1
```

The Suricata configuration was updated:

```bash
sudo sed -i 's/interface: eth0/interface: enp0s1/g' /etc/suricata/suricata.yaml
```

The change was verified with:

```bash
grep -n "interface:" /etc/suricata/suricata.yaml | head
```

Relevant output included:

```text
615:  - interface: enp0s1
807:  - interface: enp0s1
```

---

## Step 4 - Suricata Ruleset Installation

Initial configuration testing showed that no Suricata rules file was available:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Warning:

```text
No rule files match the pattern /var/lib/suricata/rules/suricata.rules
```

The Emerging Threats Open ruleset was downloaded using:

```bash
sudo suricata-update
```

The update process reported:

```text
Loaded 68172 rules.
Writing rules to /var/lib/suricata/rules/suricata.rules
enabled: 52235
```

The generated rules file was verified:

```bash
ls -lh /var/lib/suricata/rules/suricata.rules
```

Observed result:

```text
-rw-r--r-- 1 root root 44M /var/lib/suricata/rules/suricata.rules
```

The configuration was tested again:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Result:

```text
Configuration provided was successfully loaded. Exiting.
```

---

## Step 5 - Suricata Service Validation

Suricata was restarted:

```bash
sudo systemctl restart suricata
```

The service was verified:

```bash
sudo systemctl status suricata
```

Result:

```text
Active: active (running)
```

### Evidence

![Suricata Running](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/01-suricata-running.png?raw=true)

This confirmed that the Suricata IDS daemon was successfully operating on the Ubuntu host.

---

## Step 6 - Detection Engine Validation

The configuration and detection engine were validated using:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

The number of loaded rules was checked with:

```bash
grep -o '"rules_loaded":[0-9]*' /var/log/suricata/eve.json | tail -1
```

Observed output:

```text
Configuration provided was successfully loaded. Exiting.
"rules_loaded":52235
```

### Evidence

![Suricata Rules Loaded](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/02-suricata-rules-loaded.png?raw=true)

This confirmed that the detection engine had successfully loaded `52,235` rules.

---

## Step 7 - Controlled TCP SYN Scan

A controlled TCP SYN scan was generated from the Mac host against the Ubuntu monitoring host:

```bash
sudo nmap -sS -p 1-5000 192.168.64.10
```

Observed output:

```text
Starting Nmap 7.99

Nmap scan report for 192.168.64.10
Host is up.

Not shown: 4999 closed tcp ports (reset)

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up)
```

### Evidence

![Nmap Scan Generated](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/03-nmap-scan-generated.png?raw=true)

The scan tested TCP ports `1-5000`.

Only TCP port `22` was identified as open, while the remaining tested ports were closed and returned TCP reset responses.

---

## Step 8 - Initial Detection Gap

Suricata statistics showed that the scan traffic was being processed.

Relevant statistics included:

```text
tcp.syn: 5226
tcp.rst: 4905
tcp.sessions: 5226
rules_loaded: 52235
alert: 0
```

The most important observation was:

```text
alert: 0
```

Suricata was clearly observing substantial TCP SYN activity, but the currently enabled rules did not generate an alert for the controlled scan.

This demonstrated an important SOC concept:

**Network visibility does not automatically equal detection coverage.**

A sensor can successfully capture suspicious behavior without producing an alert if the relevant detection logic is missing, disabled, or does not match the activity.

---

## Step 9 - Custom Detection Rule Creation

A custom Suricata rule was created to detect repeated TCP SYN activity:

```text
alert tcp any any -> $HOME_NET any (msg:"LOCAL TCP SYN Port Scan Detected"; flags:S; flow:stateless; detection_filter:track by_src, count 20, seconds 10; classtype:network-scan; sid:1000001; rev:1;)
```

The rule was designed to detect:

- TCP traffic
- SYN packets
- Traffic directed toward `$HOME_NET`
- At least 20 matching SYN packets
- From the same source
- Within 10 seconds

The rule used:

```text
SID: 1000001
Revision: 1
Classification: network-scan
```

The threshold logic was:

```text
detection_filter:track by_src, count 20, seconds 10
```

This means Suricata tracks matching packets by source and triggers the rule once the threshold is reached.

---

## Step 10 - Local Rule Configuration

The Suricata configuration initially contained:

```yaml
rule-files:
  - suricata.rules
```

It was updated to include the custom rule:

```yaml
rule-files:
  - suricata.rules
  - local.rules
```

The custom rule file was placed at:

```text
/var/lib/suricata/rules/local.rules
```

The configuration was validated again:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

After successful validation, Suricata was restarted:

```bash
sudo systemctl restart suricata
```

The service remained operational:

```text
Active: active (running)
```

---

## Step 11 - Alert Generation

The controlled SYN scan was repeated:

```bash
sudo nmap -sS -p 1-5000 192.168.64.10
```

Suricata alerts were reviewed using:

```bash
sudo tail -n 20 /var/log/suricata/fast.log
```

Multiple alerts were generated with the signature:

```text
LOCAL TCP SYN Port Scan Detected
```

The alerts included:

```text
Classification: Detection of a Network Scan
Priority: 3
Protocol: TCP
Source IP: 192.168.64.1
Destination IP: 192.168.64.10
```

Multiple destination ports were observed, including:

```text
4593
1457
4249
1453
3947
2947
1060
914
859
4399
288
4587
2544
2838
603
1392
3599
4084
291
4768
```

### Evidence

![Suricata Alert Event](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/04-suricata-alert-event.png?raw=true)

The alerts confirmed that the custom detection logic successfully identified the repeated SYN activity.

---

## Step 12 - Structured Alert Investigation

Structured Suricata alerts were reviewed using:

```bash
grep '"event_type":"alert"' /var/log/suricata/eve.json | tail -3
```

The JSON records confirmed the following fields:

```text
event_type: alert
src_ip: 192.168.64.1
src_port: 36741
dest_ip: 192.168.64.10
proto: TCP
signature_id: 1000001
signature: LOCAL TCP SYN Port Scan Detected
category: Detection of a Network Scan
severity: 3
in_iface: enp0s1
```

Observed destination ports in the final JSON records included:

```text
4084
291
4768
```

### Evidence

![Suricata EVE JSON Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/05-suricata-eve-json-alert.png?raw=true)

This demonstrated the difference between Suricata's two primary alert formats.

`fast.log` provides concise, human-readable alert information.

`eve.json` provides structured fields suitable for:

- SIEM ingestion
- filtering
- enrichment
- correlation
- automation
- dashboards
- incident investigation

---

# Alert Triage

## What Was Observed?

Repeated TCP SYN packets were sent from a single source host toward thousands of TCP ports on the Ubuntu host.

The behavior was detected by Suricata after a custom threshold-based SYN scan detection rule was deployed.

---

## Where Was It Observed?

The activity was observed on:

```text
Host: ubuntu-agent
Interface: enp0s1
Destination IP: 192.168.64.10
```

Relevant Suricata evidence was stored in:

```text
/var/log/suricata/fast.log
/var/log/suricata/eve.json
```

---

## Which User, Host, IP Address, Domain, or File Was Involved?

### Scan Source

```text
192.168.64.1
```

### Destination Host

```text
ubuntu-agent
```

### Destination IP

```text
192.168.64.10
```

### Open Service Identified

```text
22/tcp - SSH
```

No domain, user account, or malicious file was involved in this network-scanning scenario.

---

## Why Is It Suspicious?

Normal client activity generally communicates with a limited number of known services.

In this case, one source generated large numbers of TCP SYN packets toward thousands of destination ports within a short period.

This pattern is consistent with:

```text
TCP SYN port scanning
```

Port scanning is frequently used during reconnaissance to identify:

- active systems
- exposed ports
- accessible services
- potential attack surfaces

However, scanning behavior alone does not prove malicious intent.

Context is required before assigning a final disposition.

---

## What Evidence Supports It?

### Nmap Evidence

The scan was intentionally generated using:

```bash
sudo nmap -sS -p 1-5000 192.168.64.10
```

### Suricata Traffic Statistics

Suricata observed:

```text
5226 TCP SYN packets
4905 TCP RST packets
5226 TCP sessions
```

### Suricata Alert

The custom rule generated:

```text
LOCAL TCP SYN Port Scan Detected
```

### Structured EVE Evidence

Relevant alert fields included:

```text
src_ip: 192.168.64.1
dest_ip: 192.168.64.10
event_type: alert
signature_id: 1000001
category: Detection of a Network Scan
severity: 3
```

The source also targeted multiple destination ports.

Together, these artifacts strongly support classification of the observed traffic as TCP SYN scanning.

---

## What Is the Possible Risk?

If similar activity occurred unexpectedly in a production environment, it could represent network reconnaissance.

A threat actor may perform port scanning before attempting:

- service enumeration
- vulnerability discovery
- exploitation
- credential attacks
- remote access attempts
- lateral movement

The scan itself does not demonstrate successful compromise.

However, it can represent an early-stage reconnaissance activity that should be correlated with subsequent events.

---

## Alternative Explanation

Scanning activity can also have legitimate explanations.

Examples include:

- authorized vulnerability scanning
- penetration testing
- asset discovery
- network administration
- infrastructure monitoring
- security validation
- troubleshooting

In this laboratory, the scan was intentionally generated by the analyst from a controlled host.

The activity was therefore authorized.

---

## Alert Disposition

**Detection Accuracy:** True Positive

**Security Context:** Benign

**Final Disposition:** **Benign True Positive**

### Reason

Suricata correctly detected genuine TCP SYN scanning behavior.

However, the scanning activity was intentionally generated in an authorized laboratory environment.

The detection was therefore technically correct, but the underlying activity did not represent a security incident.

---

## Escalation Decision

**Escalation Required:** No

No escalation was required because:

- the source host was known
- the activity was intentionally generated
- the destination host was part of the controlled lab
- no unauthorized behavior occurred
- no exploitation followed the scan

In a production SOC environment, escalation would depend on:

- whether the source host was authorized
- whether the source belonged to an approved vulnerability scanner
- whether scanning was scheduled
- whether additional hosts were targeted
- whether exploitation followed the scan
- whether authentication attempts followed
- whether sensitive systems were targeted
- whether the source was internal or external

---

# Detection Gap Analysis

One of the most important findings occurred before the custom rule was deployed.

Suricata successfully observed the network traffic but generated:

```text
alert: 0
```

despite having:

```text
rules_loaded: 52235
```

This demonstrated that:

```text
Network visibility ≠ detection coverage
```

The IDS was operational and receiving packets.

The issue was not a sensor failure.

Instead, the existing detection logic did not generate an alert for the controlled SYN scan.

This represented a detection coverage gap for the lab scenario.

The gap was addressed by implementing a threshold-based custom SYN scan rule.

After the new rule was loaded, repeating the same behavior successfully produced alerts.

---

# Detection Logic Analysis

The custom Suricata rule contained:

```text
flags:S
```

This matches TCP packets with the SYN flag set.

The rule also contained:

```text
flow:stateless
```

This allows the rule to inspect the SYN activity without requiring an established TCP session.

The threshold logic was:

```text
detection_filter:track by_src, count 20, seconds 10
```

This instructs Suricata to track matching activity from each individual source.

When at least 20 matching SYN packets are observed from one source within 10 seconds, the detection threshold is reached.

The rule then generates:

```text
LOCAL TCP SYN Port Scan Detected
```

This approach reduces the likelihood of treating an isolated normal TCP connection as scanning activity.

---

# MITRE ATT&CK Mapping

## T1046 - Network Service Discovery

The observed activity is consistent with:

**MITRE ATT&CK T1046 - Network Service Discovery**

Network service discovery can be used to identify services running on remote systems and determine potential attack paths.

In this lab, Nmap was used to scan ports `1-5000` on the Ubuntu target.

The behavior matched the technical characteristics of network service discovery, although the activity was authorized and intentionally generated.

---

# Investigation Timeline

```text
Suricata installed
        ↓
Initial Suricata service failure identified
        ↓
Incorrect eth0 interface discovered
        ↓
Active interface enp0s1 identified
        ↓
Suricata interface configuration corrected
        ↓
Emerging Threats Open rules installed
        ↓
52,235 rules successfully loaded
        ↓
Suricata service started successfully
        ↓
TCP SYN scan generated from 192.168.64.1
        ↓
Suricata observed thousands of SYN packets
        ↓
No initial IDS alert generated
        ↓
Detection coverage gap identified
        ↓
Custom SYN scan detection rule created
        ↓
local.rules added to Suricata
        ↓
Configuration validated
        ↓
Suricata restarted
        ↓
TCP SYN scan repeated
        ↓
LOCAL TCP SYN Port Scan Detected
        ↓
fast.log reviewed
        ↓
eve.json reviewed
        ↓
Source and destination confirmed
        ↓
Alert classified as Benign True Positive
        ↓
No escalation required
```

---

# Raw Log vs Structured Alert

## fast.log

Suricata `fast.log` provided concise alert information such as:

```text
LOCAL TCP SYN Port Scan Detected
Classification: Detection of a Network Scan
Priority: 3
TCP 192.168.64.1 -> 192.168.64.10
```

### Advantages

- Fast manual review
- Human-readable format
- Useful for initial validation
- Concise detection summary

### Limitations

- Less structured
- Harder to automate
- Less suitable for large-scale correlation

---

## eve.json

Structured EVE alert fields included:

```json
{
  "event_type": "alert",
  "src_ip": "192.168.64.1",
  "dest_ip": "192.168.64.10",
  "proto": "TCP",
  "signature_id": 1000001,
  "signature": "LOCAL TCP SYN Port Scan Detected",
  "category": "Detection of a Network Scan",
  "severity": 3,
  "in_iface": "enp0s1"
}
```

### Advantages

- Structured JSON format
- SIEM compatible
- Supports filtering
- Supports field extraction
- Easier correlation
- Easier enrichment
- Suitable for automation
- Useful for dashboards and alert pipelines

---

# Recommended Next Steps

If this alert appeared unexpectedly in a production SOC environment, the analyst should:

1. Validate whether `192.168.64.1` belongs to an authorized security scanner.

2. Determine whether vulnerability scanning or asset discovery was scheduled.

3. Identify the complete range of destination ports targeted.

4. Determine whether the same source scanned additional hosts.

5. Review firewall and network telemetry for related activity.

6. Review endpoint telemetry from the source host.

7. Search for exploitation attempts following the scan.

8. Search for authentication attempts originating from the same IP.

9. Correlate the source with other SIEM alerts.

10. Determine whether the target system contains sensitive services.

11. Escalate if the scanning activity is unauthorized or followed by additional suspicious behavior.

---

# Final SOC Summary

Suricata identified repeated TCP SYN scanning activity originating from `192.168.64.1` and targeting the Ubuntu host `192.168.64.10`.

Initial investigation confirmed that Suricata was successfully processing the network traffic, including thousands of TCP SYN packets, but the existing ruleset did not initially produce an alert.

This was identified as a detection coverage gap rather than a sensor failure.

A custom threshold-based Suricata rule was implemented to identify repeated SYN activity originating from a single source.

After validating the configuration and restarting Suricata, the controlled Nmap scan was repeated.

Suricata successfully generated multiple `LOCAL TCP SYN Port Scan Detected` alerts classified as `Detection of a Network Scan`, with signature ID `1000001` and severity `3`.

The alerts were investigated using both `fast.log` and structured `eve.json` evidence.

The source, destination, protocol, interface, signature, severity, and multiple destination ports were confirmed.

Because the activity was intentionally generated in an authorized laboratory environment, the alert was classified as a **Benign True Positive**.

No escalation was required.

---

# Lessons Learned

This lab demonstrated that network visibility and detection coverage are separate concepts.

I learned how to:

- Install Suricata on Ubuntu
- Troubleshoot a failed Suricata service
- Identify an incorrect monitoring interface
- Identify the active Linux network interface
- Configure Suricata to monitor `enp0s1`
- Install the Emerging Threats Open ruleset
- Validate the Suricata configuration
- Verify the number of loaded detection rules
- Generate controlled SYN scanning activity using Nmap
- Review Suricata network statistics
- Identify a detection coverage gap
- Create a custom Suricata detection rule
- Configure a local rules file
- Apply threshold-based detection logic
- Validate a custom IDS signature
- Investigate alerts using `fast.log`
- Investigate structured alerts using `eve.json`
- Extract alert fields
- Distinguish visibility from detection
- Classify a Benign True Positive
- Make an escalation decision based on context

The most important lesson was:

> An IDS can successfully observe suspicious network activity without generating an alert if the appropriate detection logic does not exist or does not match the behavior.

This makes detection validation and analyst verification important components of SOC operations.

---

# SOC English

Useful analyst sentences from this lab:

- Suricata observed repeated TCP SYN packets originating from a single source host.

- The traffic pattern was consistent with TCP SYN port scanning.

- The IDS initially processed the traffic without generating a corresponding alert.

- This indicated a detection coverage gap rather than a sensor failure.

- A custom threshold-based Suricata rule was implemented to detect repeated SYN activity.

- The custom signature successfully generated alerts after the scan was repeated.

- The alert was validated using both human-readable and structured log sources.

- The source host targeted multiple destination ports within a short period.

- The activity represented network reconnaissance behavior but was authorized within the laboratory environment.

- The alert was therefore classified as a Benign True Positive.

- No escalation was required because the source and activity were known and controlled.

- Network visibility alone does not guarantee complete detection coverage.

---


# Interview Explanation

In this lab, I installed Suricata on an Ubuntu monitoring host and initially encountered a service failure because the default configuration referenced `eth0`, while the actual interface on my VM was `enp0s1`.

I identified the correct interface, updated the Suricata configuration, installed the Emerging Threats Open ruleset, and verified that more than 52,000 detection rules were loaded.

I then generated a controlled Nmap TCP SYN scan from another host against ports 1 through 5000 on the Ubuntu system.

Suricata successfully observed thousands of SYN packets, but no alert was initially generated.

Instead of assuming that the IDS was malfunctioning, I verified that the sensor was receiving traffic and identified the situation as a detection coverage gap.

I then created a custom Suricata rule that tracked SYN activity by source and triggered when at least 20 matching packets were observed within 10 seconds.

After validating the rule configuration and restarting Suricata, I repeated the Nmap scan.

The custom rule successfully generated multiple alerts.

I investigated the alerts using both `fast.log` and `eve.json`, confirming the source IP, destination IP, protocol, interface, signature ID, severity, and multiple destination ports.

Because the scan was intentionally generated inside my own controlled environment, I classified the alert as a Benign True Positive and determined that escalation was unnecessary.

The key lesson from the lab was that network visibility and detection coverage are not the same thing.

---

# Screenshots

## 01 - Suricata Running

![Suricata Running](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/01-suricata-running.png?raw=true)

## 02 - Suricata Rules Loaded

![Suricata Rules Loaded](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/02-suricata-rules-loaded.png?raw=true)

## 03 - Nmap SYN Scan Generated

![Nmap Scan Generated](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/03-nmap-scan-generated.png?raw=true)

## 04 - Suricata Alert Event

![Suricata Alert Event](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/04-suricata-alert-event.png?raw=true)

## 05 - Suricata EVE JSON Alert

![Suricata EVE JSON Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-023/05-suricata-eve-json-alert.png?raw=true)

---
