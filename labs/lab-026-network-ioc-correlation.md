# Lab 026 - Network IOC Correlation

## Executive Summary

This lab investigated how multiple network indicators can be correlated into a single analyst assessment rather than evaluated independently.

A controlled connection to `example.com` was captured from the Ubuntu endpoint `192.168.64.10`. The investigation correlated DNS responses, observed HTTP and HTTPS connections, TLS Server Name Indication (SNI), Suricata telemetry, IP ownership, ASN information, TLS certificate metadata, and external threat-intelligence reputation.

DNS resolution returned multiple IPv4 and IPv6 addresses for `example.com`. The endpoint subsequently communicated specifically with `172.66.147.243` over TCP ports 80 and 443.

HTTP telemetry identified `example.com` as the requested host, while TLS telemetry independently confirmed the same hostname through SNI.

Suricata reconstructed DNS, HTTP, TLS, and flow telemetry from the same PCAP without generating an alert.

WHOIS identified the destination infrastructure as Cloudflare, while VirusTotal associated the observed IP address with `AS13335`, Cloudflare, Inc.

TLS certificate metadata was consistent with `example.com`, and VirusTotal reported no malicious detections for either the domain or the observed destination IP.

The combined evidence supported a final disposition of benign expected lab traffic.

---

## Objective

The objective of this lab was to practice network IOC correlation by connecting multiple indicators and telemetry sources into one investigation.

The investigation focused on:

- Domain
- IP address
- Hostname / URL context
- Port
- Protocol
- DNS activity
- HTTP activity
- TLS SNI
- Suricata telemetry
- ASN
- Network ownership
- TLS certificate
- Threat-intelligence reputation

The goal was not simply to perform an IP lookup on VirusTotal.

The goal was to determine whether multiple independent evidence sources described the same network activity.

---

## Environment / Data Source

**Host:** Ubuntu Agent  
**Host IP:** `192.168.64.10`  
**DNS Server:** `192.168.64.1`  
**Primary Domain:** `example.com`  
**Observed Destination IP:** `172.66.147.243`  
**Additional Resolved IPv4:** `104.20.23.154`  
**Tools:** tcpdump, TShark, Suricata, jq, WHOIS, OpenSSL, VirusTotal  
**Evidence Sources:** PCAP, Suricata `eve.json`, WHOIS, TLS certificate, VirusTotal  
**PCAP:** `lab-026-network-ioc-correlation.pcap`  
**Time Window:** 18 August 2026

---

## Investigation Workflow

    example.com
          ↓
    DNS resolution
          ↓
    172.66.147.243
    104.20.23.154
          ↓
    Observed destination
    172.66.147.243
          ↓
    TCP/80
    HTTP Host: example.com
          ↓
    TCP/443
    TLS SNI: example.com
          ↓
    Suricata telemetry
          ↓
    WHOIS / Network Ownership
          ↓
    ASN
          ↓
    TLS Certificate
          ↓
    Threat Intelligence
          ↓
    Final Disposition

---

# Step 1 - Packet Capture

Network traffic was captured using tcpdump.

Command:

    sudo tcpdump -i enp0s1 -nn -s0 \
    -w ~/lab-026-network-ioc-correlation.pcap \
    'port 53 or port 80 or port 443'

Traffic was generated using:

    nslookup example.com

    curl -I http://example.com

    curl -I https://example.com

Capture result:

    39 packets captured
    39 packets received by filter
    0 packets dropped by kernel

PCAP size:

    9.1K

The capture contained DNS, HTTP, and HTTPS traffic required for the correlation investigation.

---

# Step 2 - DNS Resolution Analysis

TShark was used to identify DNS activity associated with `example.com`.

Command:

    tshark -r ~/lab-026-network-ioc-correlation.pcap \
    -Y 'dns.qry.name == "example.com"' \
    -T fields \
    -e frame.number \
    -e frame.time \
    -e ip.src \
    -e ip.dst \
    -e dns.flags.response \
    -e dns.qry.name \
    -e dns.a \
    -e dns.aaaa

Observed IPv4 addresses:

    172.66.147.243
    104.20.23.154

Observed IPv6 addresses:

    2606:4700:10::6814:179a
    2606:4700:10::ac42:93f3

The endpoint:

    192.168.64.10

queried:

    192.168.64.1

for:

    example.com

This established the first relationship:

    example.com
          ↓
    DNS Resolution
          ↓
    172.66.147.243
    104.20.23.154

---

# Step 3 - TCP Connection Analysis

TCP traffic to ports 80 and 443 was extracted.

Command:

    tshark -r ~/lab-026-network-ioc-correlation.pcap \
    -Y 'tcp.port == 80 || tcp.port == 443' \
    -T fields \
    -e frame.number \
    -e frame.time \
    -e ip.src \
    -e ip.dst \
    -e tcp.srcport \
    -e tcp.dstport

The endpoint communicated with:

    172.66.147.243:80
    172.66.147.243:443

Although DNS also returned:

    104.20.23.154

no TCP connection to that address was observed in the captured traffic.

DNS resolution alone does not prove that every returned IP address was contacted.

---

## Initial TCP SYN Evidence

Only initial TCP SYN packets were extracted to identify connection attempts clearly.

Command:

    tshark -r ~/lab-026-network-ioc-correlation.pcap \
    -Y 'tcp.flags.syn == 1 && tcp.flags.ack == 0' \
    -T fields \
    -e frame.number \
    -e frame.time \
    -e ip.src \
    -e ip.dst \
    -e tcp.srcport \
    -e tcp.dstport

Observed connections:

    Frame 5
    192.168.64.10 → 172.66.147.243
    Source Port: 51706
    Destination Port: 80

    Frame 15
    192.168.64.10 → 172.66.147.243
    Source Port: 34366
    Destination Port: 443

This proved that the endpoint actively initiated both HTTP and HTTPS connections to `172.66.147.243`.

---

# Step 4 - HTTP Correlation

HTTP requests were extracted from the PCAP.

Command:

    tshark -r ~/lab-026-network-ioc-correlation.pcap \
    -Y 'http.request' \
    -T fields \
    -e frame.number \
    -e frame.time \
    -e ip.src \
    -e ip.dst \
    -e http.request.method \
    -e http.host \
    -e http.request.uri

Observed request:

    Frame: 8
    Source: 192.168.64.10
    Destination: 172.66.147.243
    Method: HEAD
    Host: example.com
    URI: /

This created a direct correlation:

    example.com
          ↓ DNS
    172.66.147.243
          ↓ TCP/80
    HTTP Host: example.com

---

# Step 5 - TLS / HTTPS Correlation

TLS SNI metadata was extracted from the HTTPS connection.

Command:

    tshark -r ~/lab-026-network-ioc-correlation.pcap \
    -Y 'tls.handshake.extensions_server_name' \
    -T fields \
    -e frame.number \
    -e frame.time \
    -e ip.src \
    -e ip.dst \
    -e tcp.srcport \
    -e tcp.dstport \
    -e tls.handshake.extensions_server_name

Observed TLS Client Hello:

    Frame: 18
    Source IP: 192.168.64.10
    Destination IP: 172.66.147.243
    Source Port: 34366
    Destination Port: 443
    SNI: example.com

This independently confirmed that the HTTPS connection to `172.66.147.243` was associated with `example.com`.

Correlation:

    example.com
          ↓
    DNS
          ↓
    172.66.147.243
          ↓
    TCP/443
          ↓
    TLS SNI: example.com

---

# Step 6 - Suricata PCAP Analysis

Suricata version:

    Suricata 7.0.3 RELEASE

A clean output directory was created:

    rm -rf ~/lab-026-suricata-output
    mkdir ~/lab-026-suricata-output

The PCAP was processed using Suricata:

    sudo suricata \
    -r ~/lab-026-network-ioc-correlation.pcap \
    -c /etc/suricata/suricata.yaml \
    -l ~/lab-026-suricata-output

Suricata processed:

    39 packets
    8630 bytes

Generated files:

    eve.json
    fast.log
    stats.log
    suricata.log

`fast.log` was empty, meaning no IDS alert was generated.

However, Suricata still produced useful:

- DNS telemetry
- HTTP telemetry
- TLS telemetry
- Flow telemetry

This demonstrates that Suricata can provide investigative telemetry even when no alert signature fires.

---

# Suricata DNS Correlation

The following command was used:

    jq -c '
    select(.event_type=="dns" and .dns.rrname=="example.com") |
    {
      timestamp,
      event_type,
      src_ip,
      dest_ip,
      rrname: .dns.rrname,
      rrtype: .dns.rrtype,
      answers: .dns.answers
    }' ~/lab-026-suricata-output/eve.json

Observed DNS telemetry:

    Source IP: 192.168.64.10
    DNS Server: 192.168.64.1
    Domain: example.com

A records:

    172.66.147.243
    104.20.23.154

AAAA records:

    2606:4700:0010:0000:0000:0000:6814:179a
    2606:4700:0010:0000:0000:0000:ac42:93f3

### Evidence Screenshot

![Suricata DNS Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/01-suricata-dns-correlation.png)

---

# Suricata HTTP Correlation

The following command was used:

    jq -c '
    select(.event_type=="http" and .http.hostname=="example.com") |
    {
      timestamp,
      event_type,
      src_ip,
      dest_ip,
      dest_port,
      hostname: .http.hostname,
      method: .http.http_method,
      status: .http.status
    }' ~/lab-026-suricata-output/eve.json

Observed telemetry:

    event_type: http
    src_ip: 192.168.64.10
    dest_ip: 172.66.147.243
    dest_port: 80
    hostname: example.com
    method: HEAD
    status: 200

This independently confirmed that HTTP traffic to `172.66.147.243` was associated with `example.com`.

### Evidence Screenshot

![Suricata HTTP Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/02-suricata-http-correlation.png)

---

# Suricata TLS Correlation

The following command was used:

    jq -c '
    select(.event_type=="tls" and .tls.sni=="example.com") |
    {
      timestamp,
      event_type,
      src_ip,
      dest_ip,
      dest_port,
      sni: .tls.sni,
      tls_version: .tls.version
    }' ~/lab-026-suricata-output/eve.json

Observed telemetry:

    event_type: tls
    src_ip: 192.168.64.10
    dest_ip: 172.66.147.243
    dest_port: 443
    sni: example.com
    tls_version: TLS 1.3

Suricata also produced TLS fingerprint information.

JA3:

    0149f47eabf9a20d0893e2a44e5a6323

JA3S:

    907bf3ecef1c987c889946b737b43de8

These fingerprints were treated as correlation metadata rather than malicious indicators.

### Evidence Screenshot

![Suricata TLS Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/03-suricata-tls-correlation.png)

---

# Suricata Flow Telemetry

Suricata recorded the HTTP flow:

    Source: 192.168.64.10
    Destination: 172.66.147.243
    Destination Port: 80
    Application Protocol: HTTP
    State: closed
    Alerted: false

Suricata also recorded the TLS flow:

    Source: 192.168.64.10
    Destination: 172.66.147.243
    Destination Port: 443
    Application Protocol: TLS
    State: closed
    Alerted: false

This demonstrated that useful investigative telemetry can exist without an IDS alert.

---

# Step 7 - IP Ownership / WHOIS Enrichment

WHOIS enrichment was performed against:

    172.66.147.243

Command:

    whois 172.66.147.243 | \
    grep -Ei 'netname|orgname|organization|descr|country|origin|originas'

Observed information:

    NetName: CLOUDFLARENET
    Organization: Cloudflare, Inc.
    OrgName: Cloudflare, Inc.
    Country: US

This associated the observed destination with Cloudflare infrastructure.

### Evidence Screenshot

![IP Ownership and ASN](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/04-ip-asn-ownership.png)

---

# Step 8 - TLS Certificate Correlation

The TLS certificate was inspected using OpenSSL.

Command:

    echo | openssl s_client \
    -connect example.com:443 \
    -servername example.com 2>/dev/null | \
    openssl x509 -noout \
    -subject \
    -issuer \
    -dates \
    -ext subjectAltName

Observed subject:

    CN = example.com

Observed issuer:

    C = US
    O = SSL Corporation
    CN = Cloudflare TLS Issuing ECC CA 3

Certificate validity:

    notBefore = Jul 29 22:10:08 2026 GMT
    notAfter  = Oct 27 22:17:21 2026 GMT

Subject Alternative Name:

    DNS:example.com
    DNS:*.example.com

The certificate metadata was consistent with the hostname observed through:

- DNS
- HTTP Host
- TLS SNI

Correlation:

    example.com
          ↓
    172.66.147.243
          ↓
    TCP/443
          ↓
    TLS SNI: example.com
          ↓
    Certificate CN: example.com
          ↓
    SAN: example.com
    SAN: *.example.com

### Evidence Screenshot

![TLS Certificate Correlation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/05-tls-certificate-correlation.png)

---

# Step 9 - VirusTotal Domain Reputation

The domain:

    example.com

was reviewed in VirusTotal.

Observed result:

    0 / 91 security vendors flagged the domain

VirusTotal also showed a positive community score.

Contextual relationships involving files communicating with the domain were visible, but those relationships were not treated as proof that the domain itself was malicious.

Threat-intelligence data must be interpreted together with local telemetry.

### Evidence Screenshot

![VirusTotal Domain Reputation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/06-virustotal-domain-reputation.png)

---

# Step 10 - VirusTotal IP Reputation

The destination IP:

    172.66.147.243

was reviewed in VirusTotal.

Observed information:

    IP: 172.66.147.243
    Network: 172.66.128.0/19
    ASN: AS13335
    Organization: Cloudflare, Inc.

Detection result:

    0 / 91 security vendors flagged the IP

The ASN information reinforced the WHOIS ownership evidence.

### Evidence Screenshot

![VirusTotal IP Reputation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-026/07-virustotal-ip-reputation.png)

---

# IOC Correlation Table

| IOC / Evidence | Observation | Correlation Value |
|---|---|---|
| Domain | `example.com` | Primary hostname investigated |
| Source Host | `192.168.64.10` | Endpoint initiating activity |
| DNS Server | `192.168.64.1` | DNS resolver |
| DNS A Record | `172.66.147.243` | Resolved and subsequently contacted |
| DNS A Record | `104.20.23.154` | Resolved but not contacted |
| DNS AAAA | `2606:4700:10::6814:179a` | IPv6 resolution evidence |
| DNS AAAA | `2606:4700:10::ac42:93f3` | IPv6 resolution evidence |
| HTTP Destination | `172.66.147.243:80` | Confirmed TCP/HTTP connection |
| HTTPS Destination | `172.66.147.243:443` | Confirmed TCP/TLS connection |
| HTTP Host | `example.com` | Application hostname confirmation |
| HTTP Method | `HEAD` | Request observed |
| HTTP Status | `200` | Successful response |
| TLS SNI | `example.com` | HTTPS hostname confirmation |
| TLS Version | `TLS 1.3` | TLS session metadata |
| WHOIS | Cloudflare, Inc. | Infrastructure ownership |
| ASN | `AS13335` | Cloudflare network |
| Certificate CN | `example.com` | Certificate hostname confirmation |
| Certificate SAN | `example.com`, `*.example.com` | Certificate relationship |
| VirusTotal Domain | `0/91` | No malicious vendor detections |
| VirusTotal IP | `0/91` | No malicious vendor detections |
| Suricata Alert | None | No IDS signature triggered |

---

# Correlation Chain

    192.168.64.10
          ↓
    DNS Query
          ↓
    example.com
          ↓
    DNS Response
          ↓
    172.66.147.243
    104.20.23.154
          ↓
    Actual Connection Observed
          ↓
    172.66.147.243
          ↓
    TCP/80
          ↓
    HTTP Host: example.com
          ↓
    TCP/443
          ↓
    TLS SNI: example.com
          ↓
    TLS 1.3
          ↓
    Suricata DNS / HTTP / TLS / Flow Telemetry
          ↓
    WHOIS: Cloudflare, Inc.
          ↓
    ASN: AS13335
          ↓
    Certificate CN: example.com
          ↓
    VirusTotal Domain: 0/91
          ↓
    VirusTotal IP: 0/91
          ↓
    Benign / Expected Lab Traffic

---

# What Was Observed?

The endpoint resolved `example.com` to multiple IP addresses and subsequently communicated with one of those addresses over HTTP and HTTPS.

The same hostname was independently identified across:

- DNS
- HTTP
- TLS SNI
- Suricata
- TLS certificate metadata

---

# Where Was It Observed?

Evidence was observed through:

- tcpdump PCAP
- TShark
- Suricata `eve.json`
- WHOIS
- OpenSSL
- VirusTotal

---

# Which User, Host, IP Address, Domain, or File Was Involved?

**Host:**

    192.168.64.10

**Domain:**

    example.com

**Observed Destination IP:**

    172.66.147.243

**Additional Resolved IPv4:**

    104.20.23.154

**Ports:**

    53
    80
    443

**Protocols:**

    UDP
    DNS
    TCP
    HTTP
    TLS / HTTPS

---

# Why Is It Suspicious?

The activity was not inherently suspicious.

This was controlled lab traffic designed to demonstrate how a potentially suspicious IOC should be investigated.

The purpose was to show that an analyst should not classify an IP or domain based only on one reputation source.

---

# What Evidence Supports the Assessment?

The assessment was supported by:

1. DNS query evidence
2. DNS response evidence
3. TCP SYN evidence
4. HTTP Host metadata
5. TLS SNI metadata
6. Suricata DNS telemetry
7. Suricata HTTP telemetry
8. Suricata TLS telemetry
9. Suricata flow metadata
10. WHOIS infrastructure ownership
11. ASN information
12. TLS certificate metadata
13. VirusTotal domain reputation
14. VirusTotal IP reputation

---

# Analysis

DNS analysis showed that `example.com` resolved to:

    172.66.147.243
    104.20.23.154

However, subsequent TCP analysis showed that only:

    172.66.147.243

was actually contacted during the captured activity.

This distinction prevented an inaccurate conclusion that both DNS-returned addresses participated in the observed communication.

The endpoint initiated:

    TCP/80
    TCP/443

connections to `172.66.147.243`.

The HTTP request contained:

    Host: example.com

The TLS Client Hello contained:

    SNI: example.com

Suricata independently reconstructed the same relationship through DNS, HTTP, TLS, and flow telemetry.

WHOIS identified the destination network as Cloudflare infrastructure.

VirusTotal identified the observed address as:

    AS13335
    Cloudflare, Inc.

Certificate analysis showed:

    CN = example.com
    SAN = example.com
    SAN = *.example.com

The infrastructure, hostname, network connection, and certificate evidence were therefore internally consistent.

VirusTotal returned:

    Domain: 0/91
    IP: 0/91

No evidence supported a malicious classification.

The investigation was therefore closed as benign expected lab traffic.

---

# Important Analyst Distinction

DNS resolution returned:

    172.66.147.243
    104.20.23.154

But only:

    172.66.147.243

was observed in subsequent network communication.

The correct analyst statement is:

> DNS resolution returned multiple IPv4 addresses, but the captured endpoint activity was specifically associated with `172.66.147.243`.

The incorrect statement would be:

> The endpoint communicated with both IP addresses.

The evidence does not support that conclusion.

---

# Threat Intelligence Interpretation

VirusTotal was used as an enrichment source rather than as the sole decision-making mechanism.

A result such as:

    0/91

does not mathematically prove that an IOC is safe.

Likewise, contextual relationships involving suspicious files do not automatically make a shared domain or IP malicious.

Threat-intelligence findings should be correlated with:

- Internal telemetry
- DNS activity
- Network connections
- HTTP metadata
- TLS metadata
- IDS telemetry
- Infrastructure ownership
- Certificate information
- Historical behavior
- Endpoint evidence

---

# Risk

No significant security risk was identified in this controlled activity.

The observed evidence was consistent with expected infrastructure.

No Suricata alert was generated.

The domain and observed IP received no malicious detections from the VirusTotal security vendors shown during the investigation.

Certificate metadata was consistent with the requested hostname.

---

# Recommended Next Steps

For this controlled lab:

1. Preserve the PCAP.
2. Preserve relevant Suricata telemetry.
3. Record the domain-to-IP relationship.
4. Record the HTTP and TLS correlation.
5. Record WHOIS and ASN information.
6. Record certificate metadata.
7. Document VirusTotal reputation results.
8. Close the activity as benign expected lab traffic.

In a production SOC investigation, additional actions could include:

1. Search firewall logs for the IP.
2. Search proxy logs for the domain.
3. Identify the endpoint process responsible for the connection.
4. Review EDR telemetry.
5. Search historical DNS activity.
6. Search SIEM data for the same IOC across additional endpoints.
7. Review connection frequency.
8. Look for beaconing patterns.
9. Check additional threat-intelligence feeds.
10. Escalate if independent evidence supports malicious activity.

---

# Should It Be Escalated?

**Escalation:** No

Reasons:

- No Suricata alert was generated.
- HTTP and TLS metadata were internally consistent.
- WHOIS and ASN information matched expected infrastructure.
- Certificate metadata matched the domain.
- VirusTotal returned no malicious vendor detections.
- The traffic was intentionally generated in a controlled lab.

---

# Final Disposition

    Disposition: Benign / Expected Lab Traffic
    Confidence: High
    Escalation: No

---

# MITRE ATT&CK Mapping

No MITRE ATT&CK technique was mapped.

This lab focused on network IOC correlation rather than confirmed adversary behavior.

MITRE ATT&CK mapping was therefore not forced.

---

# Final SOC Summary

Network telemetry from host `192.168.64.10` showed DNS resolution of `example.com` followed by HTTP and HTTPS connections to `172.66.147.243`. DNS returned multiple IPv4 addresses, but only `172.66.147.243` was observed in subsequent TCP communication. HTTP Host metadata and TLS SNI independently associated the network sessions with `example.com`. Suricata reconstructed consistent DNS, HTTP, TLS, and flow telemetry without generating an alert. WHOIS identified the destination as Cloudflare infrastructure, while VirusTotal associated the IP with `AS13335`, Cloudflare, Inc. TLS certificate metadata was also consistent with `example.com`. VirusTotal reported zero malicious vendor detections for both the domain and the observed IP. Based on correlation across all available evidence sources, the traffic was assessed as benign expected lab activity and did not require escalation.

---

# Lessons Learned

This lab demonstrated that IOC investigation should focus on relationships rather than isolated indicators.

The main lesson was:

    IOC Lookup ≠ IOC Correlation

A single VirusTotal lookup cannot explain how an indicator was involved in endpoint activity.

A stronger investigation follows the chain:

    Domain
       ↓
    DNS Resolution
       ↓
    IP Address
       ↓
    Actual Network Connection
       ↓
    Port / Protocol
       ↓
    HTTP / TLS Metadata
       ↓
    Suricata Telemetry
       ↓
    Infrastructure Ownership
       ↓
    ASN
       ↓
    Certificate
       ↓
    Threat Intelligence
       ↓
    Analyst Disposition

The lab also demonstrated that a DNS response can contain multiple addresses without proving that all of them were contacted.

Actual communication must be verified through network telemetry.

Suricata can provide useful investigation data even when no IDS alert fires.

Finally, external threat-intelligence results should support an investigation rather than replace analyst reasoning.

---

# SOC English

> The endpoint resolved the domain to multiple IP addresses, but only one of the returned addresses was observed in subsequent network communication.

> The DNS-resolved infrastructure was subsequently contacted by the endpoint over both HTTP and HTTPS.

> HTTP metadata identified the requested hostname as `example.com`.

> TLS SNI independently confirmed that the HTTPS session was associated with the same hostname.

> Suricata reconstructed DNS, HTTP, TLS, and flow telemetry without generating an IDS alert.

> WHOIS and ASN enrichment associated the destination IP with Cloudflare infrastructure.

> Certificate metadata was consistent with the hostname observed in network telemetry.

> External reputation data did not identify the domain or destination IP as malicious.

> No single indicator was considered sufficient to determine maliciousness.

> The final disposition was based on correlation across multiple independent evidence sources.

---

# CV Bullet

- Correlated DNS, HTTP, TLS, Suricata, ASN, certificate, PCAP, and threat-intelligence evidence to trace domain-to-IP network activity and produce an evidence-based analyst disposition.

---

# Interview Explanation

**Question: How do you investigate a suspicious network IOC?**

> I avoid relying on a single reputation lookup. I first determine where the IOC appeared in internal telemetry and then correlate it across multiple evidence sources. In my Network IOC Correlation lab, I started with a domain observed in DNS traffic, identified the returned IP addresses, and verified which IP was actually contacted using PCAP evidence. I correlated that connection with HTTP Host metadata and TLS SNI, then compared the activity with Suricata DNS, HTTP, TLS, and flow telemetry. I enriched the destination using WHOIS and ASN information, validated the hostname relationship through TLS certificate metadata, and finally reviewed the domain and IP in VirusTotal. The final disposition was based on the consistency of the complete evidence chain rather than any single indicator.

---
