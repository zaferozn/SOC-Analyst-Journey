# Lab 022 - Network Scanning Detection

## Executive Summary

This lab investigated TCP SYN-based network service scanning in a controlled laboratory environment using Nmap and Wireshark.

A SYN scan was generated from `192.168.64.1` against the Ubuntu target `192.168.64.10`. Twelve TCP destination ports were probed within a short time window.

Packet analysis showed a clear multi-port scanning pattern originating from a single source. TCP port `22` responded with `SYN/ACK`, indicating that the SSH service was listening, while the remaining eleven tested ports returned `RST/ACK`, indicating closed TCP ports.

Further analysis of the port 22 exchange showed that the scanner responded to the target's `SYN/ACK` with a `RST` rather than completing the normal TCP three-way handshake. This behavior confirmed the characteristics of an Nmap `-sS` SYN, or half-open, scan.

The observed behavior was consistent with network service scanning. Because the activity was intentionally generated inside a controlled lab environment, the final disposition was **Benign Positive - Authorized Simulated Reconnaissance**.

---

## Objective

The objectives of this lab were to:

- Generate controlled TCP SYN scanning activity.
- Identify a single source probing multiple destination ports.
- Analyze SYN packets generated during reconnaissance.
- Differentiate open and closed TCP ports using packet responses.
- Identify `SYN/ACK` behavior associated with an open port.
- Identify `RST/ACK` behavior associated with closed ports.
- Analyze the half-open behavior of an Nmap SYN scan.
- Reconstruct the network scanning sequence from packet evidence.
- Practice SOC-style detection and disposition of reconnaissance activity.

---

## Environment / Data Source

**Scanner:** `192.168.64.1`  
**Target Host:** `192.168.64.10`  
**Target Role:** Ubuntu security lab endpoint  
**Capture Interface:** `bridge100`  
**Tools:** Nmap, Wireshark  
**Scan Type:** TCP SYN Scan (`-sS`)  
**Packet Source:** Wireshark PCAP capture  
**Protocol:** TCP  
**Primary Open Port:** `22/tcp - SSH`

### Scanned TCP Ports

```text
21
22
23
25
53
80
110
139
443
445
3389
8080
```

---

## Tools Used

- Nmap 7.99
- Wireshark
- macOS Terminal
- UTM virtualized laboratory network
- Ubuntu target host

---

## Scan Generation

The following controlled Nmap command was used:

```bash
sudo nmap -sS -Pn -p 21,22,23,25,53,80,110,139,443,445,3389,8080 192.168.64.10
```

### Command Breakdown

```text
sudo
```

Runs Nmap with the privileges required for raw-packet SYN scanning.

```text
-sS
```

Performs a TCP SYN scan.

```text
-Pn
```

Skips host-discovery checks and treats the target as online.

```text
-p
```

Specifies the destination ports to probe.

```text
192.168.64.10
```

Specifies the target host.

---

## Observed Activity

A single source host:

```text
192.168.64.1
```

sent TCP SYN packets toward multiple destination ports on:

```text
192.168.64.10
```

within a very short time period.

The destination ports changed rapidly while the source and target remained consistent.

The observed pattern was:

```text
Single source IP
        ↓
Single destination host
        ↓
Multiple destination ports
        ↓
Rapid TCP SYN probes
        ↓
Different TCP responses
        ↓
Network service scanning behavior
```

---

## Evidence

### Evidence 00 - Nmap SYN Scan Results

The Nmap scan reported one open TCP port and eleven closed TCP ports.

![Nmap SYN Scan Results](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/00-nmap-syn-scan-results.png?raw=true)

Observed result:

```text
PORT      STATE   SERVICE
21/tcp    closed  ftp
22/tcp    open    ssh
23/tcp    closed  telnet
25/tcp    closed  smtp
53/tcp    closed  domain
80/tcp    closed  http
110/tcp   closed  pop3
139/tcp   closed  netbios-ssn
443/tcp   closed  https
445/tcp   closed  microsoft-ds
3389/tcp  closed  ms-wbt-server
8080/tcp  closed  http-proxy
```

The scan completed in approximately:

```text
0.56 seconds
```

Initial result:

```text
12 TCP ports probed
        ↓
1 open port
        ↓
22/tcp SSH

11 closed ports
```

Nmap provided the initial scan result, while Wireshark was used to independently verify the underlying packet behavior.

---

### Evidence 01 - Multiple-Port SYN Scan

Wireshark was filtered to identify initial TCP SYN packets from the scanner to the target.

#### Wireshark Filter

```text
ip.src == 192.168.64.1 && ip.dst == 192.168.64.10 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

![Multiple Port SYN Scan](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/01-multiple-port-syn-scan.png?raw=true)

The filtered packets showed SYN probes toward:

```text
8080
21
139
443
445
23
80
25
53
3389
110
22
```

The packets originated from:

```text
192.168.64.1
```

and targeted:

```text
192.168.64.10
```

The destination port changed rapidly between packets.

#### Analyst Interpretation

This is a strong network-scanning indicator because one source systematically attempted to identify listening services across multiple TCP ports.

```text
192.168.64.1
      |
      +---- SYN → 192.168.64.10:21
      +---- SYN → 192.168.64.10:22
      +---- SYN → 192.168.64.10:23
      +---- SYN → 192.168.64.10:25
      +---- SYN → 192.168.64.10:53
      +---- SYN → 192.168.64.10:80
      +---- SYN → 192.168.64.10:110
      +---- SYN → 192.168.64.10:139
      +---- SYN → 192.168.64.10:443
      +---- SYN → 192.168.64.10:445
      +---- SYN → 192.168.64.10:3389
      +---- SYN → 192.168.64.10:8080
```

The rapid multi-port behavior is substantially different from ordinary access to one expected application service.

---

### Evidence 02 - Open Port 22 SYN/ACK Response

The target's responses containing both SYN and ACK flags were isolated.

#### Wireshark Filter

```text
ip.src == 192.168.64.10 && ip.dst == 192.168.64.1 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

![Open Port 22 SYN ACK](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/02-open-port-22-syn-ack.png?raw=true)

Wireshark showed:

```text
Source:      192.168.64.10
Destination: 192.168.64.1

22 → 46205 [SYN, ACK]
```

#### Interpretation

```text
192.168.64.1:46205
        |
        | SYN
        v
192.168.64.10:22
        |
        | SYN/ACK
        v
192.168.64.1:46205
```

A `SYN/ACK` response to the SYN probe indicates that a TCP service was listening on the destination port.

```text
SYN sent to port 22
        ↓
SYN/ACK returned
        ↓
TCP service listening
        ↓
Port 22 OPEN
```

This packet-level evidence independently confirmed the Nmap result:

```text
22/tcp open ssh
```

---

### Evidence 03 - Closed-Port RST/ACK Responses

The target's reset responses were isolated using the following Wireshark filter:

```text
ip.src == 192.168.64.10 && ip.dst == 192.168.64.1 && tcp.flags.reset == 1
```

![Closed Ports RST Responses](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/03-closed-ports-rst-responses.png?raw=true)

Wireshark displayed eleven `RST/ACK` responses from:

```text
192.168.64.10
```

to:

```text
192.168.64.1
```

The responses corresponded to:

```text
8080
21
139
443
445
23
80
25
53
3389
110
```

Example behavior:

```text
8080 → 46205 [RST, ACK]
21   → 46205 [RST, ACK]
443  → 46205 [RST, ACK]
3389 → 46205 [RST, ACK]
110  → 46205 [RST, ACK]
```

#### Interpretation

```text
Scanner
   |
   | SYN
   v
Closed Target Port
   |
   | RST/ACK
   v
Scanner
```

Therefore:

```text
SYN
 ↓
RST/ACK
 ↓
Port CLOSED
```

The eleven RST/ACK responses matched the eleven ports reported as closed by Nmap.

---

### Evidence 04 - Scanner-Side RST to Open Port 22

The scanner's behavior after receiving the SYN/ACK response from TCP port 22 was examined.

#### Wireshark Filter

```text
ip.src == 192.168.64.1 && ip.dst == 192.168.64.10 && tcp.dstport == 22 && tcp.flags.reset == 1
```

![SYN Scan Half Open RST](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/04-syn-scan-half-open-rst.png?raw=true)

The packet showed:

```text
192.168.64.1 → 192.168.64.10
46205 → 22
[RST]
```

#### Interpretation

A normal TCP connection would continue:

```text
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
Connection established
```

However, the observed Nmap behavior was:

```text
SYN
 ↓
SYN/ACK
 ↓
RST
 ↓
Connection aborted
```

The scanner did not complete the normal TCP three-way handshake.

---

### Evidence 05 - Complete Port 22 Half-Open Sequence

The complete port 22 exchange was isolated to reconstruct the relevant TCP sequence.

![Port 22 Half Open Scan Sequence](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-022/05-port-22-half-open-scan-sequence.png?raw=true)

The sequence demonstrated:

```text
192.168.64.1:46205                    192.168.64.10:22
       Scanner                              Target

          SYN ───────────────────────────────►

              ◄────────────────────── SYN/ACK

          RST ───────────────────────────────►
```

This behavior is consistent with an Nmap TCP SYN scan.

The connection was not allowed to transition into a fully established TCP session.

---

## Open vs Closed Port Comparison

### Open Port

```text
Scanner → Target
       SYN
        ↓
Target → Scanner
     SYN/ACK
        ↓
      OPEN
```

Observed:

```text
22/tcp
```

### Closed Port

```text
Scanner → Target
       SYN
        ↓
Target → Scanner
     RST/ACK
        ↓
     CLOSED
```

Observed on eleven tested ports.

---

## SYN Scan vs Normal TCP Connection

### Normal TCP Three-Way Handshake

```text
Client                     Server

 SYN  ─────────────────────►

      ◄──────────────── SYN/ACK

 ACK  ─────────────────────►

      CONNECTION ESTABLISHED
```

### Nmap SYN Scan Observed in This Lab

```text
Scanner                    Target

 SYN  ─────────────────────►

      ◄──────────────── SYN/ACK

 RST  ─────────────────────►

      CONNECTION NOT COMPLETED
```

This is why `nmap -sS` is commonly described as a **SYN scan** or **half-open scan**.

---

## Analysis

The observed activity was consistent with automated TCP network service scanning.

Several indicators supported this assessment.

### 1. Single Source

The activity originated from:

```text
192.168.64.1
```

### 2. Single Target

The destination host was:

```text
192.168.64.10
```

### 3. Multiple Destination Ports

The source systematically probed twelve TCP services instead of communicating with only one expected application port.

### 4. Rapid SYN Activity

The SYN packets occurred within a very short interval.

This behavior is consistent with automated scanning rather than ordinary interactive access.

### 5. Different Responses Revealed Port State

Port 22 returned:

```text
SYN/ACK
```

indicating an open port.

The other eleven tested ports returned:

```text
RST/ACK
```

indicating closed ports.

### 6. Handshake Was Not Completed

After receiving the SYN/ACK from port 22, the scanner returned:

```text
RST
```

instead of the final:

```text
ACK
```

expected in a normal TCP connection.

This packet behavior was consistent with an Nmap `-sS` scan.

---

## What Was Observed?

A single source host generated rapid TCP SYN probes against multiple destination ports on another host.

---

## Where Was It Observed?

The activity was observed in a Wireshark packet capture collected from the laboratory network through the `bridge100` interface.

---

## Which User, Host, IP Address, Domain, or File Was Involved?

### Source / Scanner

```text
192.168.64.1
```

### Destination / Target

```text
192.168.64.10
```

### Open Service

```text
22/tcp - SSH
```

No domain, file, or user account was directly involved in this network-scanning event.

---

## Why Is It Suspicious?

In an uncontrolled production environment, rapid probing of many destination ports from a single source may indicate reconnaissance.

An attacker may perform network service scanning to identify:

- Exposed services
- Remote administration interfaces
- Vulnerable applications
- Unnecessary listening services
- Potential entry points

However, scanning behavior alone does not prove malicious intent.

Administrative vulnerability scanners, monitoring systems, penetration tests, asset-discovery tools, and authorized security assessments may generate similar traffic.

Context is therefore necessary before escalation.

---

## What Evidence Supports It?

The assessment was supported by:

- Twelve TCP SYN probes from one source toward multiple destination ports.
- A consistent source and target relationship.
- Rapid changes in destination port.
- A SYN/ACK response from TCP port 22.
- RST/ACK responses from eleven closed ports.
- Scanner-side RST behavior following the SYN/ACK response.
- Nmap output confirming one open and eleven closed TCP ports.
- Wireshark packet evidence independently validating the Nmap result.

---

## Risk

If similar activity were unexpected in a production environment, the potential risks could include:

- Unauthorized reconnaissance
- Service enumeration
- Identification of exposed attack surfaces
- Preparation for exploitation
- Discovery of remote-access services
- Targeting of SSH or other reachable services

The discovery of an open SSH service could become more significant if followed by:

```text
Network scanning
        ↓
SSH authentication attempts
        ↓
Repeated login failures
        ↓
Successful authentication
        ↓
Privilege escalation
```

Such correlation would substantially increase the severity of the event.

---

## Alternative Explanation

Network scanning is not inherently malicious.

Potential legitimate explanations include:

- Authorized penetration testing
- Vulnerability scanning
- Asset discovery
- Network administration
- Service monitoring
- Security validation
- Laboratory exercises

In this case, the scan was intentionally generated as part of the controlled SOC laboratory.

Therefore, malicious attribution would not be appropriate.

---

## Disposition

```text
Detection:       Network Service Scanning
Behavior:        TCP SYN / Half-Open Scan
Source:          192.168.64.1
Target:          192.168.64.10
Ports Scanned:   12
Open Ports:      1
Closed Ports:    11
Open Service:    SSH / TCP 22

Classification:
Benign Positive

Context:
Authorized Simulated Reconnaissance

Escalation:
No
```

---

## Recommended Next Steps

In a real SOC environment, the analyst should:

1. Determine whether the source IP belongs to an authorized scanner or administrator.
2. Identify the owner of the source system.
3. Review the total number of destination ports and systems contacted.
4. Determine whether the activity targeted one host or multiple hosts.
5. Establish the scan time window.
6. Review firewall and IDS/IPS telemetry.
7. Check whether the source generated authentication attempts after scanning.
8. Investigate activity against identified open services.
9. Review SSH authentication logs if TCP port 22 was targeted.
10. Correlate the activity with SIEM alerts and endpoint telemetry.
11. Determine whether similar reconnaissance occurred previously.
12. Escalate if the scan was unauthorized or followed by suspicious activity.

---

## Should It Be Escalated?

```text
Escalation Required: No
```

Reason:

The scanning activity was intentionally generated by the analyst inside an isolated laboratory environment.

In a production SOC environment, escalation would depend on authorization, source identity, target sensitivity, scan scope, and follow-on activity.

---

## MITRE ATT&CK Mapping

### T1046 - Network Service Scanning

The observed activity is consistent with:

```text
T1046 - Network Service Scanning
```

The mapping is supported by:

```text
One source
      ↓
Multiple destination ports
      ↓
TCP SYN probes
      ↓
Service-state discovery
      ↓
Open / closed port identification
```

The MITRE ATT&CK mapping describes the observed behavior only.

It does not imply that the activity was malicious because this event occurred in an authorized training environment.

---

## Final SOC Summary

A controlled TCP SYN scan was generated from `192.168.64.1` against `192.168.64.10` and analyzed using Wireshark. The source rapidly transmitted SYN packets toward twelve TCP destination ports, demonstrating behavior consistent with network service scanning. TCP port 22 responded with SYN/ACK, confirming that the SSH service was listening, while the remaining eleven tested ports returned RST/ACK responses indicating closed ports. After receiving the SYN/ACK from port 22, the scanner transmitted a RST rather than completing the TCP three-way handshake, confirming the half-open behavior associated with an Nmap `-sS` scan. The activity maps to MITRE ATT&CK technique `T1046 - Network Service Scanning`. Because the scan was intentionally generated within a controlled laboratory environment, the event was classified as a benign positive and did not require escalation.

---

## Lessons Learned

This lab strengthened my ability to identify network scanning directly from packet-level evidence rather than relying only on a scanning tool's output.

Key lessons included:

- A single source rapidly probing multiple destination ports is an important scanning indicator.
- TCP SYN packets can reveal systematic service-discovery activity.
- A SYN/ACK response indicates that a TCP service is listening.
- A RST/ACK response commonly indicates a closed TCP port during this type of scan.
- Nmap `-sS` does not need to complete the normal TCP three-way handshake.
- The scanner can send a RST after receiving SYN/ACK from an open port.
- Packet evidence can independently validate Nmap's classification of port states.
- Scanning behavior should not automatically be classified as malicious.
- Authorization and environmental context are necessary for correct SOC disposition.
- MITRE ATT&CK mapping describes behavior and should not be treated as proof of malicious intent.

---

## SOC English Sentences

### Observation

> A single source initiated TCP SYN packets toward multiple destination ports within a short time window.

### Open-Port Analysis

> TCP port 22 returned a SYN/ACK response, indicating that a service was listening on the destination port.

### Closed-Port Analysis

> The remaining tested ports returned RST/ACK responses, indicating that those TCP ports were closed at the time of the scan.

### Scan Identification

> The rapid multi-port probing pattern was consistent with TCP SYN-based network service scanning.

### Half-Open Analysis

> The scanner responded to the SYN/ACK packet with a TCP reset instead of completing the three-way handshake, which was consistent with a half-open SYN scan.

### Context

> Although the packet pattern was consistent with reconnaissance, the activity was intentionally generated within a controlled laboratory environment.

### Disposition

> The event was classified as a benign positive because the scanning activity was authorized and expected.

### Escalation

> No escalation was required; however, equivalent activity from an unauthorized source in a production environment would require additional investigation.

---

## Interview Explanation

### Question

**How would you detect a network scan?**

### Answer

I would first look for a source communicating with multiple destination ports or multiple hosts within a relatively short time window. For TCP SYN scanning, I would specifically examine repeated SYN packets and identify how the destination responds.

A SYN/ACK response can indicate that a TCP service is listening, while RST/ACK responses can indicate closed ports. I would also evaluate the timing, number of destination ports, source identity, target systems, and whether similar activity occurred against additional hosts.

I would not automatically classify the activity as malicious because vulnerability scanners, asset-discovery systems, administrators, and penetration testers can generate similar traffic. I would verify whether the source was authorized and then correlate the scan with subsequent activity such as authentication attempts, exploitation, or privilege escalation before deciding whether escalation was required.

In Lab 022, I generated an Nmap SYN scan against twelve ports. Wireshark showed rapid SYN probes from one source, a SYN/ACK response from open SSH port 22, RST/ACK responses from eleven closed ports, and a scanner-side RST after the SYN/ACK. This allowed me to validate the scan directly from raw packet evidence.

---
