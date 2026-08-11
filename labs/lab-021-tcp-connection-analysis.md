# Lab 021 - TCP Connection Analysis

## Executive Summary

This lab investigated the lifecycle of a TCP connection using Wireshark and packet-level evidence from `lab-020.pcap`.

The analysis focused on an SSH connection between:

- Client: `192.168.64.1:53399`
- Server: `192.168.64.10:22`
- Application Protocol: SSHv2
- TCP Stream: `7`

The investigation confirmed that the connection successfully completed the TCP three-way handshake, progressed into SSHv2 communication, and later terminated through an orderly FIN/ACK sequence.

Additional analysis found no TCP reset packets and no TCP retransmissions associated with the selected session.

The observed TCP behavior was consistent with a successfully established and normally terminated SSH connection.

---

## Objective

The objective of this lab was to analyze a TCP connection using Wireshark and determine:

- How a TCP connection is initiated
- How the TCP three-way handshake works
- Whether the connection was successfully established
- Which source and destination hosts were involved
- Which ports and application protocol were used
- Whether TCP reset packets occurred
- Whether retransmissions occurred
- How the connection was terminated
- Whether the observed behavior appeared normal or suspicious

---

## Environment / Data Source

**Tool:** Wireshark  
**PCAP:** `lab-020.pcap`  
**Environment:** Local virtual security lab  
**Network Protocol:** TCP  
**Application Protocol:** SSHv2  
**TCP Stream:** `7`

### Client

**IP Address:** `192.168.64.1`  
**Source Port:** `53399`

### Server

**IP Address:** `192.168.64.10`  
**Destination Port:** `22`

---

# Investigation

## 1. TCP SYN Investigation

The following Wireshark display filter was used to identify TCP connection initiation packets:

`tcp.flags.syn == 1 && tcp.flags.ack == 0`

Several SYN packets were identified.

The SSH connection selected for deeper investigation was:

- Frame: `5748`
- Time: `273.873763`
- Source IP: `192.168.64.1`
- Destination IP: `192.168.64.10`
- Source Port: `53399`
- Destination Port: `22`
- TCP Flags: `SYN, ECE, CWR`

The important indicator was the `SYN` flag.

A SYN packet represents an attempt to initiate a TCP connection.

Observed connection attempt:

`192.168.64.1:53399 → 192.168.64.10:22`

Because destination port `22` was used, the destination service was identified as SSH.

### Evidence

![TCP SYN Packets](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/01-tcp-syn-packets.png?raw=true)

### Interpretation

The source host `192.168.64.1` attempted to initiate a TCP connection to the SSH service running on `192.168.64.10`.

The additional `ECE` and `CWR` flags were related to TCP Explicit Congestion Notification functionality and did not indicate malicious activity by themselves.

---

## 2. SYN-ACK Response Investigation

The following filter was used to identify SYN-ACK responses:

`tcp.flags.syn == 1 && tcp.flags.ack == 1`

The response corresponding to the selected SSH connection was:

- Frame: `5749`
- Time: `273.873881`
- Source IP: `192.168.64.10`
- Destination IP: `192.168.64.1`
- Source Port: `22`
- Destination Port: `53399`
- TCP Flags: `SYN, ACK, ECE`

Observed response:

`192.168.64.10:22 → 192.168.64.1:53399`

### Evidence

![TCP SYN-ACK Response](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/02-tcp-syn-ack-response.png?raw=true)

### Interpretation

The destination host received the initial TCP SYN and responded with a SYN-ACK.

This confirmed that:

- The destination host was reachable
- Port `22` was responding
- The SSH service accepted the TCP connection attempt

At this stage, two parts of the TCP handshake had been observed:

`SYN → SYN-ACK`

---

## 3. TCP Three-Way Handshake

The three handshake packets were isolated using:

`frame.number == 5748 || frame.number == 5749 || frame.number == 5750`

The following sequence was observed.

### Frame 5748

`192.168.64.1:53399 → 192.168.64.10:22`

Flags:

`SYN`

Purpose:

The client requested a TCP connection.

---

### Frame 5749

`192.168.64.10:22 → 192.168.64.1:53399`

Flags:

`SYN, ACK`

Purpose:

The server acknowledged the request and responded with its own SYN.

---

### Frame 5750

`192.168.64.1:53399 → 192.168.64.10:22`

Flags:

`ACK`

Purpose:

The client acknowledged the server response.

---

### Complete Handshake

`SYN → SYN-ACK → ACK`

### Evidence

![TCP Three-Way Handshake](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/03-tcp-three-way-handshake.png?raw=true)

---

## TCP Handshake Timeline

| Frame | Time | Source | Destination | Flags | Interpretation |
|---|---:|---|---|---|---|
| 5748 | 273.873763 | `192.168.64.1:53399` | `192.168.64.10:22` | SYN | Connection initiated |
| 5749 | 273.873881 | `192.168.64.10:22` | `192.168.64.1:53399` | SYN, ACK | Server responded |
| 5750 | 273.874255 | `192.168.64.1:53399` | `192.168.64.10:22` | ACK | Connection established |

The complete TCP three-way handshake occurred within approximately:

`0.492 milliseconds`

This fast response was consistent with communication occurring inside the local virtual lab network.

### Analyst Assessment

The presence of:

`SYN → SYN-ACK → ACK`

confirmed that the TCP connection was successfully established.

---

# 4. SSHv2 Application Traffic

After completion of the TCP handshake, Wireshark identified SSHv2 traffic.

Observed application-layer activity included:

- SSH protocol negotiation
- SSH client identification
- SSH server identification
- Key exchange initialization
- Diffie-Hellman key exchange
- New Keys messages
- Encrypted SSH packets

Examples observed in the TCP conversation included protocol identification similar to:

`Client: Protocol (SSH-2.0-OpenSSH_10.2)`

and:

`Server: Protocol (SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.18)`

After the SSH key exchange process, application communication appeared as encrypted SSH packets.

### Interpretation

The presence of SSHv2 traffic demonstrated that the connection progressed beyond the TCP handshake into active application-layer communication.

Because SSH provides encrypted remote communication, the contents of the later application traffic were not directly readable from the PCAP.

However, network metadata still provided useful evidence including:

- Source IP
- Destination IP
- Source port
- Destination port
- Connection timing
- Packet direction
- TCP flags
- Session establishment
- Session termination

---

# 5. TCP Stream Investigation

The connection was initially isolated using:

`tcp.port == 53399`

The TCP packet details identified:

`Stream index: 7`

The exact connection was then isolated using:

`tcp.stream eq 7`

Wireshark identified the selected conversation as:

`Conversation completeness: Complete, WITH_DATA`

### TCP Stream Details

**Stream:** `7`

**Client:**  
`192.168.64.1:53399`

**Server:**  
`192.168.64.10:22`

**Application:**  
SSHv2

### Interpretation

Using the TCP stream index allowed the investigation to focus on one specific TCP conversation rather than all traffic involving port `53399`.

Wireshark's `Complete, WITH_DATA` classification indicated that the capture contained a complete TCP conversation with application data.

---

# 6. TCP FIN Investigation

The following filter was used to identify TCP FIN packets:

`tcp.flags.fin == 1`

The SSH session was then isolated using:

`tcp.port == 53399 && tcp.flags.fin == 1`

Two FIN/ACK packets associated with the selected session were identified.

### Frame 6801

`192.168.64.1:53399 → 192.168.64.10:22`

Flags:

`FIN, ACK`

This indicated that the client initiated connection termination.

---

### Frame 6803

`192.168.64.10:22 → 192.168.64.1:53399`

Flags:

`FIN, ACK`

This indicated that the server also completed its side of the connection termination process.

### Evidence

![TCP FIN Connection Termination](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/04-tcp-fin-connection-termination.png?raw=true)

---

# 7. Complete TCP Termination Sequence

The termination packets were reconstructed using:

`frame.number == 6801 || frame.number == 6802 || frame.number == 6803 || frame.number == 6804`

The following sequence was observed.

### Frame 6801

`192.168.64.1:53399 → 192.168.64.10:22`

`FIN, ACK`

The client initiated connection termination.

---

### Frame 6802

`192.168.64.10:22 → 192.168.64.1:53399`

`ACK`

The server acknowledged the client's FIN.

---

### Frame 6803

`192.168.64.10:22 → 192.168.64.1:53399`

`FIN, ACK`

The server initiated termination of its side of the TCP connection.

---

### Frame 6804

`192.168.64.1:53399 → 192.168.64.10:22`

`ACK`

The client acknowledged the server's FIN.

---

## TCP Termination Timeline

| Frame | Time | Source | Destination | Flags | Interpretation |
|---|---:|---|---|---|---|
| 6801 | 293.383269 | `192.168.64.1:53399` | `192.168.64.10:22` | FIN, ACK | Client initiated termination |
| 6802 | 293.383443 | `192.168.64.10:22` | `192.168.64.1:53399` | ACK | Server acknowledged FIN |
| 6803 | 293.396547 | `192.168.64.10:22` | `192.168.64.1:53399` | FIN, ACK | Server initiated termination |
| 6804 | 293.397951 | `192.168.64.1:53399` | `192.168.64.10:22` | ACK | Client acknowledged final FIN |

The termination sequence was therefore:

`FIN-ACK → ACK → FIN-ACK → ACK`

### Interpretation

This represented an orderly TCP shutdown.

The connection was not abruptly terminated.

---

# 8. TCP Reset Analysis

TCP reset packets were investigated using:

`tcp.port == 53399 && tcp.flags.reset == 1`

The filter returned:

`Displayed: 0`

No TCP RST packets were identified for the selected SSH session.

### Evidence

![No TCP Reset Observed](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/05-no-tcp-reset-observed.png?raw=true)

### Interpretation

TCP RST packets represent abrupt connection termination.

Possible causes of RST traffic can include:

- Closed ports
- Service rejection
- Application failure
- Firewall behavior
- Unexpected TCP state
- Forced connection termination
- Some types of scanning activity

No RST packets were observed in this SSH session.

Combined with the complete FIN/ACK termination sequence, this supported the assessment that the connection ended normally.

---

# 9. TCP Retransmission Analysis

The following Wireshark filter was used:

`tcp.port == 53399 && tcp.analysis.retransmission`

The result was:

`Displayed: 0`

No TCP retransmissions were detected for the selected SSH connection.

### Evidence

![No TCP Retransmissions Observed](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/06-no-tcp-retransmissions-observed.png?raw=true)

### Interpretation

TCP retransmissions can occur when a sender does not receive the expected acknowledgment and sends packet data again.

Possible causes include:

- Packet loss
- Network congestion
- Latency
- Unstable network connectivity
- Receiver-side communication problems

No retransmissions were identified in the analyzed session.

The absence of retransmissions did not by itself prove that the network was healthy, but no retransmission-related anomalies were identified for this connection.

---

# 10. Complete TCP Connection Lifecycle

The most important packets from the connection establishment and termination phases were isolated using:

`frame.number == 5748 || frame.number == 5749 || frame.number == 5750 || frame.number == 6801 || frame.number == 6802 || frame.number == 6803 || frame.number == 6804`

Seven packets represented the major TCP lifecycle events.

### Connection Establishment

`5748 — SYN`

`5749 — SYN, ACK`

`5750 — ACK`

### Application Communication

`SSHv2 communication`

### Connection Termination

`6801 — FIN, ACK`

`6802 — ACK`

`6803 — FIN, ACK`

`6804 — ACK`

### Evidence

![TCP Connection Lifecycle](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-021/07-tcp-connection-lifecycle.png?raw=true)

---

# TCP Lifecycle Reconstruction

    192.168.64.1:53399                     192.168.64.10:22
            Client                               SSH Server

                SYN
                -------------------------------->

                            SYN, ACK
                <--------------------------------

                ACK
                -------------------------------->

                    TCP CONNECTION ESTABLISHED

                                ↓

                          SSHv2 TRAFFIC

                                ↓

                FIN, ACK
                -------------------------------->

                            ACK
                <--------------------------------

                            FIN, ACK
                <--------------------------------

                ACK
                -------------------------------->

                    TCP CONNECTION CLOSED

---

# Observed Activity

A complete TCP connection was observed between:

`192.168.64.1:53399`

and:

`192.168.64.10:22`

The connection:

1. Began with a TCP SYN
2. Received a SYN-ACK response
3. Completed the handshake with ACK
4. Progressed into SSHv2 communication
5. Contained encrypted SSH traffic
6. Showed no TCP resets
7. Showed no TCP retransmissions
8. Terminated through an orderly FIN/ACK sequence

---

# What Was Observed?

A complete TCP connection lifecycle associated with an SSH session was observed.

The connection successfully completed the TCP three-way handshake and subsequently exchanged SSHv2 application traffic.

The connection later terminated normally using FIN/ACK packets.

---

# Where Was It Observed?

The activity was observed in:

`lab-020.pcap`

using Wireshark.

---

# Which User, Host, IP Address, Domain, or File Was Involved?

## Client Host

IP:

`192.168.64.1`

Source Port:

`53399`

## Server Host

IP:

`192.168.64.10`

Destination Port:

`22`

## Application Protocol

`SSHv2`

## TCP Stream

`7`

No username was identified from the packet-level evidence alone.

---

# Why Is It Suspicious?

The selected TCP connection was not determined to be suspicious based on the available network evidence.

The connection demonstrated expected TCP behavior:

- Standard three-way handshake
- Expected SSHv2 communication
- No TCP resets
- No retransmissions
- Normal FIN/ACK termination

Individual TCP connections should not be classified as malicious solely because SSH or encrypted traffic is present.

Security context would be required to determine whether the SSH activity itself was authorized.

---

# What Evidence Supports the Assessment?

The assessment was supported by:

- Frame `5748` — SYN
- Frame `5749` — SYN-ACK
- Frame `5750` — ACK
- SSHv2 protocol negotiation
- SSH encrypted communication
- TCP Stream `7`
- `Complete, WITH_DATA` conversation status
- Zero TCP RST packets
- Zero TCP retransmissions
- Frame `6801` — FIN, ACK
- Frame `6802` — ACK
- Frame `6803` — FIN, ACK
- Frame `6804` — ACK

---

# Analysis

The selected TCP stream represented a complete SSH connection from `192.168.64.1:53399` to `192.168.64.10:22`.

The session began with a standard TCP three-way handshake.

The client initiated communication by sending a SYN packet.

The server responded with a SYN-ACK, confirming that the destination host was reachable and that TCP port `22` was accepting the connection.

The client then transmitted the final ACK, completing the connection establishment process.

Following the handshake, SSHv2 protocol communication was observed.

The traffic included protocol negotiation, key exchange activity, and encrypted SSH packets.

This confirmed that the connection progressed into active application communication rather than ending immediately after TCP establishment.

The session was identified as TCP stream `7`.

Wireshark reported the conversation as:

`Complete, WITH_DATA`

TCP reset analysis returned zero results.

TCP retransmission analysis also returned zero results.

The connection later terminated through:

`FIN-ACK → ACK → FIN-ACK → ACK`

This sequence represented an orderly shutdown rather than an abrupt termination.

Based on the packet-level evidence, the selected TCP session demonstrated normal connection establishment, application communication, and termination behavior.

---

# Risk

No immediate network-level threat was identified in this specific TCP session.

However, SSH connections can become security-relevant when associated with:

- Unauthorized remote access
- Brute-force authentication
- Compromised credentials
- Suspicious source IP addresses
- Unexpected administrative access
- Privilege escalation
- Lateral movement
- Persistence activity

Therefore, network evidence should be correlated with authentication logs, endpoint activity, and SIEM alerts when investigating potentially suspicious SSH sessions.

---

# Recommended Next Steps

For a real SOC investigation, the following actions would be appropriate:

1. Correlate the SSH connection with Linux authentication logs.
2. Identify the user account involved.
3. Determine whether the source IP was expected.
4. Review failed authentication attempts before the successful connection.
5. Check whether privileged commands were executed.
6. Review sudo activity.
7. Correlate the connection with Wazuh alerts.
8. Check for file modifications after login.
9. Review the duration and frequency of SSH sessions.
10. Investigate whether the same source contacted additional ports.
11. Look for repeated SYN packets across different services.
12. Correlate network and endpoint evidence into a common incident timeline.

---

# Should It Be Escalated?

**Current disposition: No escalation based on TCP behavior alone.**

The selected connection demonstrated normal TCP lifecycle behavior.

Escalation would become appropriate if additional evidence showed:

- Unauthorized authentication
- Suspicious source activity
- Multiple failed login attempts
- Unexpected privileged access
- Malicious file modification
- Network scanning
- Lateral movement
- Other correlated security alerts

---

# MITRE ATT&CK Mapping

No MITRE ATT&CK technique was assigned to this TCP session because the observed connection itself did not demonstrate adversary behavior.

Normal TCP and SSH traffic should not be forced into an ATT&CK mapping.

If future analysis identifies repeated connection attempts across multiple ports or network services, the behavior may potentially map to:

**T1046 - Network Service Scanning**

Mapping should only be applied when the observed behavior supports the technique.

---

# Final SOC Summary

Wireshark analysis identified a complete SSH TCP connection from `192.168.64.1:53399` to `192.168.64.10:22`. The session successfully completed the standard TCP three-way handshake using SYN, SYN-ACK, and ACK packets before progressing to SSHv2 application communication. The connection was identified as TCP stream `7`, and Wireshark reported the conversation as complete with application data. No TCP reset packets or retransmissions were identified. The session subsequently terminated through an orderly FIN/ACK sequence in both directions. Based on the available packet-level evidence, the analyzed connection demonstrated normal TCP lifecycle behavior and no network-level indicators requiring escalation were identified.

---

# Lessons Learned

This lab demonstrated how TCP connection behavior can be reconstructed using packet-level evidence.

Key lessons included:

- SYN initiates a TCP connection.
- SYN-ACK indicates that the destination received and responded to the connection request.
- ACK completes the TCP three-way handshake.
- A completed `SYN → SYN-ACK → ACK` sequence confirms successful TCP connection establishment.
- Source and destination ports help identify network services.
- Port `22` is commonly associated with SSH.
- TCP streams allow a specific conversation to be isolated.
- `tcp.stream eq X` is more precise than filtering only by port.
- SSH protocol negotiation is visible before communication becomes fully encrypted.
- Encrypted traffic can still provide useful metadata.
- FIN normally represents orderly connection termination.
- RST represents abrupt connection termination.
- RST traffic is not automatically malicious.
- TCP retransmissions may indicate packet-loss or network-delivery problems.
- Retransmissions are not automatically malicious.
- Negative evidence such as the absence of RST or retransmissions can support an analyst assessment.
- TCP behavior should always be interpreted within context.
- Network evidence can be correlated with endpoint and SIEM logs during incident investigations.

---

# SOC English Practice

## Connection Establishment

"The TCP three-way handshake was successfully completed between the source and destination hosts."

"The source host initiated the connection by transmitting a SYN packet to the destination SSH service."

"The destination responded with a SYN-ACK, confirming that the service was reachable and accepting the TCP connection request."

"The client transmitted the final ACK, completing TCP connection establishment."

---

## Application Traffic

"Following the successful handshake, SSHv2 application traffic was observed."

"The session progressed from TCP connection establishment into SSH protocol negotiation and encrypted communication."

"The packet capture exposed connection metadata even though the SSH application payload was encrypted."

---

## Reset Analysis

"No TCP reset packets were identified for the analyzed SSH session."

"The absence of RST traffic indicates that the session was not terminated through an abrupt TCP reset."

---

## Retransmission Analysis

"No TCP retransmissions were identified during the observed SSH session."

"Wireshark did not detect packets requiring retransmission within the selected TCP stream."

---

## Connection Termination

"The connection terminated through an orderly FIN/ACK exchange."

"The complete termination sequence indicated a controlled TCP shutdown rather than an abrupt reset."

---

## Analyst Assessment

"Based on the packet-level evidence, the connection demonstrated normal TCP lifecycle behavior."

"No network-level indicators requiring escalation were identified in the selected session."

"Additional endpoint and authentication evidence would be required to determine whether the SSH access itself was authorized."

---

# Interview Explanation

## Question

**How do you determine whether a TCP connection was successfully established?**

## Answer

I would first examine the packet sequence for the TCP three-way handshake.

The initiating host sends a SYN packet to the destination.

If the destination service is reachable and accepts the connection attempt, it responds with a SYN-ACK.

The initiating host then returns an ACK.

Therefore, when I observe:

`SYN → SYN-ACK → ACK`

I can confirm that the TCP connection was successfully established.

I would then examine the subsequent application traffic and determine how the session terminated.

For example, a normal TCP session may terminate through a FIN/ACK exchange, while an RST indicates abrupt termination.

I would also investigate retransmissions, repeated connection attempts, source and destination ports, connection timing, and surrounding network activity before deciding whether the behavior was suspicious.

---
