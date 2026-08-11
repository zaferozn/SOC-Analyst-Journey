# Lab 020 - Wireshark and PCAP Fundamentals

## Executive Summary

This lab focused on foundational network packet analysis using `tcpdump` and Wireshark.

A controlled packet capture was collected from the Ubuntu security agent while ICMP, DNS, HTTP, HTTPS/TLS, and SSH traffic was intentionally generated.

The resulting PCAP contained **6,833 packets**.

Wireshark was used to identify:

- Network endpoints
- Host-to-host conversations
- Protocol distribution
- DNS resolution
- TCP connection attempts
- HTTP communication
- TLS-encrypted traffic
- SSH communication
- ICMP connectivity testing
- Chronological network activity

The captured activity was correlated with known lab actions and expected background communication.

No evidence was identified that justified classification as malicious or escalation.

---

## Objective

The analyst question was:

> **What network activity occurred inside this packet capture, and which traffic deserves further investigation?**

The investigation focused on:

- Identifying communicating hosts
- Identifying protocols and ports
- Reviewing DNS queries and responses
- Identifying TCP connection initiators
- Comparing HTTP and HTTPS visibility
- Isolating an SSH session
- Reviewing ICMP traffic
- Reconstructing activity chronologically
- Determining whether observed traffic required escalation

---

## Environment / Data Source

**Host:** `ubuntu-agent`  
**Ubuntu Agent IP:** `192.168.64.10`  
**Mac Client IP:** `192.168.64.1`  
**Wazuh Server IP:** `192.168.64.9`  
**Interface:** `enp0s1`  
**Capture Tool:** `tcpdump`  
**Analysis Tool:** Wireshark  
**Capture File:** `lab-020.pcap`  
**Capture Size:** approximately `2.4 MB`  
**Packets Captured:** `6,833`  
**Packets Dropped:** `0`

---

## Tools Used

- Ubuntu Linux
- tcpdump
- Wireshark
- ping
- nslookup
- curl
- SSH

---

# 1. Packet Capture Creation

The active network interface was identified with:

```bash
ip addr
```

The Ubuntu agent used:

```text
enp0s1
192.168.64.10/24
```

Packet capture was started with:

```bash
sudo tcpdump -i enp0s1 -w lab-020.pcap
```

Controlled traffic was then generated.

### ICMP

```bash
ping -c 4 8.8.8.8
```

Result:

```text
4 packets transmitted
0 received
100% packet loss
```

### DNS

```bash
nslookup example.com
```

DNS response:

```text
104.20.23.154
172.66.147.243
```

### HTTP

```bash
curl -I http://example.com
```

Result:

```text
HTTP/1.1 200 OK
```

### HTTPS

```bash
curl -I https://example.com
```

Result:

```text
HTTP/2 200
```

### SSH

From the Mac host:

```bash
ssh analyst@192.168.64.10
```

The session was successfully established and later terminated with:

```bash
exit
```

---

## Evidence 01 - PCAP Overview

![PCAP Overview](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/01-pcap-overview.png?raw=true)

The capture completed with:

```text
6833 packets captured
6834 packets received by filter
0 packets dropped by kernel
```

The PCAP was approximately:

```text
2.4 MB
```

The capture was verified using:

```bash
sudo tcpdump -nn -r lab-020.pcap | head -20
```

### Finding

The PCAP was successfully collected and contained readable network traffic suitable for Wireshark analysis.

---

# 2. Endpoint Analysis

Wireshark endpoint statistics were reviewed through:

```text
Statistics → Endpoints → IPv4
```

## Evidence 02 - IPv4 Endpoints

![IPv4 Endpoints](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/02-ip-endpoints.png?raw=true)

Important IPv4 endpoints included:

| IP Address | Context |
|---|---|
| `192.168.64.10` | Ubuntu agent |
| `192.168.64.1` | Mac / SSH client |
| `192.168.64.9` | Wazuh server |
| `8.8.8.8` | ICMP destination |
| `104.20.23.154` | `example.com` communication |
| `172.66.147.243` | DNS result for `example.com` |
| `91.189.91.103` | Ubuntu infrastructure |
| `91.189.91.104` | Ubuntu infrastructure |

### Finding

The endpoint view answered:

> **Which systems appeared inside the packet capture?**

The Ubuntu agent `192.168.64.10` was the primary system generating and receiving network traffic.

---

# 3. Conversation Analysis

Wireshark conversation statistics were reviewed through:

```text
Statistics → Conversations → IPv4
```

## Evidence 03 - IPv4 Conversations

![IPv4 Conversations](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/03-ip-conversations.png?raw=true)

Important conversations included:

```text
192.168.64.10 ↔ 192.168.64.1
```

This included SSH communication.

```text
192.168.64.10 ↔ 192.168.64.9
```

This represented expected internal communication with the Wazuh server.

```text
192.168.64.10 → 8.8.8.8
```

Four packets were transmitted without replies.

```text
192.168.64.10 ↔ 104.20.23.154
```

This communication was associated with the controlled HTTP and HTTPS activity.

### Finding

Endpoints show:

> **Which systems exist?**

Conversations show:

> **Which systems communicated with each other?**

This distinction is useful during initial PCAP triage.

---

# 4. Protocol Analysis

The Protocol Hierarchy was reviewed through:

```text
Statistics → Protocol Hierarchy
```

## Evidence 04 - Protocol Analysis

![Protocol Analysis](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/04-protocol-analysis.png?raw=true)

Observed protocols included:

- IPv4
- IPv6
- TCP
- UDP
- SSH
- HTTP
- TLS
- DNS
- ICMP
- ARP
- DHCP
- NTP

Important statistics:

```text
Total packets: 6833
IPv4 packets: 6767
TCP packets: 6755
SSH packets: 3087
HTTP packets: 13
TLS packets: 11
ICMP packets: 4
```

### Finding

TCP represented the majority of the captured traffic.

SSH represented a significant portion because SSH communication occurred during the capture window.

The Protocol Hierarchy provided a high-level overview before individual packet analysis.

---

# 5. DNS Analysis

DNS traffic was isolated using:

```wireshark
dns
```

## Evidence 05 - DNS Filter

![DNS Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/05-dns-filter.png?raw=true)

A DNS query for:

```text
example.com
```

was observed.

The response returned:

```text
104.20.23.154
172.66.147.243
```

Additional Ubuntu-related DNS activity was also visible.

### Finding

The activity could be correlated as:

```text
example.com
↓
DNS Resolution
↓
104.20.23.154
172.66.147.243
```

Subsequent HTTP and HTTPS communication was observed with `104.20.23.154`.

This demonstrated how DNS evidence can provide context for later network connections.

---

# 6. TCP Connection Analysis

Initial TCP connections were isolated using:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

## Evidence 06 - TCP SYN Filter

![TCP SYN Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/06-tcp-syn-filter.png?raw=true)

Relevant connection attempts included:

```text
192.168.64.10 → 104.20.23.154:80
192.168.64.10 → 104.20.23.154:443
192.168.64.1  → 192.168.64.10:22
```

Relevant ports:

| Port | Service |
|---|---|
| `22` | SSH |
| `80` | HTTP |
| `443` | HTTPS/TLS |

### Finding

TCP SYN packets identified which system initiated each connection.

For example:

```text
192.168.64.1 → 192.168.64.10:22
```

showed that the Mac host initiated the SSH connection toward the Ubuntu agent.

---

# 7. SSH Analysis

The controlled SSH session used client source port:

```text
53399
```

The specific session was isolated using:

```wireshark
tcp.port == 53399
```

## Evidence 07 - SSH Filter

![SSH Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/07-ssh-filter.png?raw=true)

The SSH connection was:

```text
192.168.64.1:53399 → 192.168.64.10:22
```

The sequence included:

```text
SYN
↓
SYN/ACK
↓
ACK
↓
SSH Protocol Negotiation
↓
Key Exchange
↓
Encrypted SSH Traffic
```

### Finding

The session exposed useful network metadata:

- Client IP
- Server IP
- Client source port
- Server destination port
- TCP handshake
- SSH negotiation
- Key exchange
- Encrypted bidirectional traffic

SSH commands and credentials were not readable because the application traffic was encrypted.

However, the connection itself could still be reconstructed from packet metadata.

---

# 8. HTTP Analysis

HTTP traffic was isolated using:

```wireshark
http
```

## Evidence 08 - HTTP Filter

![HTTP Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/08-http-filter.png?raw=true)

The controlled request showed:

```text
192.168.64.10 → 104.20.23.154
HEAD / HTTP/1.1
```

The response was:

```text
104.20.23.154 → 192.168.64.10
HTTP/1.1 200 OK
```

This corresponded to:

```bash
curl -I http://example.com
```

### Finding

Because HTTP was unencrypted, application-layer information was directly visible.

This included:

- HTTP method
- Request path
- Response status
- Header information

Additional Ubuntu HTTP traffic was consistent with package activity generated during the lab.

---

# 9. HTTPS / TLS Analysis

TLS traffic associated with the controlled HTTPS connection was isolated using:

```wireshark
tls && ip.addr == 104.20.23.154
```

## Evidence 09 - TLS Filter

![TLS Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/09-tls-filter.png?raw=true)

The session showed:

```text
192.168.64.10 → 104.20.23.154:443
Client Hello
SNI = example.com
```

Encrypted application traffic followed.

### HTTP vs HTTPS

HTTP:

```text
HEAD / HTTP/1.1
HTTP/1.1 200 OK
```

HTTPS:

```text
Client Hello
SNI = example.com
Server Hello
Encrypted Application Data
```

### Finding

TLS protected the application content.

However, useful metadata remained visible:

- Source IP
- Destination IP
- Port `443`
- TLS handshake
- SNI
- Packet timing
- Communication direction

This demonstrated that encrypted traffic can still provide valuable SOC evidence.

---

# 10. ICMP Analysis

ICMP traffic was isolated using:

```wireshark
icmp
```

## Evidence 10 - ICMP Filter

![ICMP Filter](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/10-icmp-filter.png?raw=true)

Four ICMP Echo Requests were observed:

```text
192.168.64.10 → 8.8.8.8
```

No corresponding Echo Replies were present.

This matched:

```text
4 packets transmitted
0 received
100% packet loss
```

### Finding

The absence of replies did not independently indicate malicious activity.

Possible explanations include:

- ICMP filtering
- Firewall policy
- Routing restrictions
- Network configuration

The traffic was intentionally generated during the lab.

---

# 11. Final Network Timeline

Representative packets were isolated using:

```wireshark
frame.number == 4335 || frame.number == 4731 || frame.number == 4747 || frame.number == 4994 || frame.number == 5020 || frame.number == 5233 || frame.number == 5748
```

## Evidence 11 - Final Network Timeline

![Final Network Timeline](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-020/11-final-network-timeline.png?raw=true)

The activity was reconstructed chronologically:

```text
1. ICMP Connectivity Test
192.168.64.10 → 8.8.8.8
Echo Request
No Response

↓

2. DNS Query
example.com

↓

3. DNS Response
104.20.23.154
172.66.147.243

↓

4. HTTP Connection
192.168.64.10 → 104.20.23.154:80
HEAD / HTTP/1.1

↓

5. HTTP Response
HTTP/1.1 200 OK

↓

6. HTTPS/TLS Connection
192.168.64.10 → 104.20.23.154:443
Client Hello
SNI = example.com

↓

7. Encrypted TLS Application Traffic

↓

8. SSH Connection
192.168.64.1:53399 → 192.168.64.10:22

↓

9. SSH Negotiation and Encrypted Session
```

---

# Analysis

The PCAP contained both deliberately generated activity and expected background traffic.

The main findings were:

- DNS resolution preceded communication with `example.com`.
- HTTP traffic exposed readable application information.
- HTTPS/TLS protected application content while leaving useful metadata visible.
- TCP SYN packets identified connection initiators.
- A specific SSH session was successfully isolated and reconstructed.
- ICMP requests received no responses.
- Ubuntu and Wazuh-related background communication was consistent with the lab environment.

No individual packet, endpoint, conversation, or protocol provided sufficient evidence of malicious activity.

---

# Risk

**Risk Level:** Low

The observed traffic was consistent with:

- Controlled lab actions
- Expected Ubuntu activity
- Internal Wazuh communication
- Normal encrypted network communication

Traffic should not be considered malicious solely because it involves:

- External IP addresses
- SSH
- TLS encryption
- High packet counts
- Failed ICMP responses

Operational context is required.

---

# Recommended Next Steps

In a production SOC investigation, the analyst should:

1. Enrich unfamiliar external IP addresses and domains.
2. Compare destinations against approved network baselines.
3. Correlate SSH connections with Linux authentication logs.
4. Correlate PCAP timestamps with SIEM alerts.
5. Confirm whether remote access was authorized.
6. Investigate unexpected outbound connections.
7. Review unusual ports or communication volumes.
8. Escalate only when additional evidence supports suspicious activity.

---

# Should It Be Escalated?

**No.**

The observed network activity was consistent with known lab actions and expected system communication.

There was insufficient evidence to classify the traffic as malicious or escalate it as a security incident.

---

# MITRE ATT&CK Mapping

No MITRE ATT&CK technique was assigned.

This lab focused on network-analysis fundamentals rather than confirmed adversary behavior.

Mapping normal or intentionally generated traffic to ATT&CK without evidence of malicious intent would reduce analytical accuracy.

---

# Final SOC Summary

A PCAP containing **6,833 packets** was analyzed using Wireshark to identify endpoints, conversations, protocols, ports, DNS resolution, TCP connections, HTTP/TLS traffic, SSH activity, and ICMP connectivity testing.

The Ubuntu agent `192.168.64.10` communicated with internal systems including `192.168.64.1` and `192.168.64.9`, as well as external infrastructure.

DNS resolution for `example.com` returned `104.20.23.154` and `172.66.147.243`. Subsequent HTTP and HTTPS communication was observed with `104.20.23.154`.

The HTTP request was directly readable, while the HTTPS session exposed TLS metadata and encrypted application traffic.

An SSH connection from `192.168.64.1:53399` to `192.168.64.10:22` was reconstructed from the TCP handshake through SSH negotiation and encrypted communication.

Four ICMP Echo Requests toward `8.8.8.8` were also observed without corresponding replies.

No evidence within the capture was sufficient to classify the observed activity as malicious or require escalation.

---

# Lessons Learned

This lab reinforced that:

- Endpoints identify systems present in a PCAP.
- Conversations identify communicating host pairs.
- Protocol Hierarchy provides an overview of traffic composition.
- TCP SYN packets identify connection initiators.
- Ports provide service context.
- DNS can be correlated with later IP communication.
- HTTP can expose readable application data.
- TLS protects content while preserving useful metadata.
- SSH traffic remains identifiable despite encryption.
- ICMP failure does not automatically indicate malicious activity.
- Packet volume alone does not establish malicious behavior.
- Network evidence must be interpreted within operational context.

---

# Useful Wireshark Filters

```wireshark
ip.addr == 192.168.64.10
ip.src == 192.168.64.10
ip.dst == 192.168.64.10
```

```wireshark
tcp
udp
dns
http
tls
icmp
```

```wireshark
tcp.port == 22
tcp.port == 53399
```

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

```wireshark
tls && ip.addr == 104.20.23.154
```

---

# SOC English

> The packet capture was reviewed to identify communicating hosts, protocols, destination services, and the chronological sequence of network activity.

> Endpoint and conversation statistics were used to identify systems and communication relationships within the capture.

> DNS activity was correlated with subsequent HTTP and HTTPS connections to resolved infrastructure.

> TCP SYN packets were reviewed to identify connection initiators and destination services.

> The HTTP request was directly visible because the communication was unencrypted.

> TLS metadata remained visible although the application content was encrypted.

> The SSH session was reconstructed from TCP connection establishment through encrypted session traffic.

> No evidence within the packet capture was sufficient to classify the observed activity as malicious.
---
# Interview Explanation

During this lab, I created a controlled packet capture using `tcpdump` and analyzed approximately 6,800 packets in Wireshark.

I began with endpoint, conversation, and protocol analysis to understand the overall network activity.

I then used display filters to investigate DNS, TCP connection attempts, HTTP, TLS, SSH, and ICMP traffic.

I correlated DNS resolution for `example.com` with subsequent HTTP and HTTPS connections to one of the returned IP addresses.

I also compared HTTP and HTTPS visibility. HTTP exposed readable application data, while TLS encrypted the application content but still exposed useful metadata such as IP addresses, ports, handshake information, and SNI.

Finally, I isolated a specific SSH session and reconstructed it from the TCP handshake through SSH negotiation and encrypted traffic.

The key lesson was that packet evidence must be evaluated within operational context before deciding whether network activity is suspicious or requires escalation.
