# Lab 024 - DNS Threat Investigation

## Executive Summary

This lab investigated DNS activity from the monitored Ubuntu endpoint using command-line DNS utilities, packet capture, Tshark, and Wireshark.

The investigation began by establishing the host network and DNS resolver configuration. Normal DNS behavior was then reviewed through multiple record types, including `A`, `AAAA`, `NS`, `MX`, `TXT`, and `PTR`.

A controlled suspicious-looking DNS pattern was subsequently generated using intentionally non-existent authentication-themed domains. Repeated DNS queries and multiple related subdomain requests produced `NXDOMAIN` responses.

The traffic was captured with `tcpdump` and analyzed with Tshark and Wireshark. Packet-level evidence confirmed that the monitored endpoint `192.168.64.10` sent DNS requests to the upstream resolver `192.168.64.1`, which returned response code `3` (`NXDOMAIN`) for the simulated domains.

Frequency analysis showed repeated requests for the same domain as well as queries for several related hostnames.

The activity was anomalous in appearance but was generated intentionally within the controlled lab environment. Therefore, the final disposition was **Simulated Benign Activity**.

The key analytical lesson was that repeated `NXDOMAIN` responses may justify further investigation but do not, by themselves, prove malicious activity.

---

## Objective

The objective of this lab was to develop practical DNS investigation skills relevant to SOC alert triage and network threat analysis.

The investigation focused on answering the following questions:

- Which host generated the DNS queries?
- Which resolver processed the requests?
- Which domains were requested?
- Which DNS record types were observed?
- Did the DNS requests succeed?
- Which IP addresses were returned for valid queries?
- Were `NXDOMAIN` responses present?
- Were any domains queried repeatedly?
- Were multiple related hostnames observed?
- Could the DNS pattern represent suspicious activity?
- What additional evidence would be required before escalation?

---

## Environment / Data Source

**Host:** `ubuntu-agent`  
**User:** `analyst`  
**Host IPv4 Address:** `192.168.64.10/24`  
**Network Interface:** `enp0s1`  
**Local DNS Stub Resolver:** `127.0.0.53`  
**Upstream DNS Resolver:** `192.168.64.1`  
**Operating System:** Ubuntu 24.04  
**Packet Capture File:** `lab-024-dns-traffic.pcap`

### Tools Used

- `dig`
- `host`
- `nslookup`
- `tcpdump`
- `tshark`
- Wireshark
- `systemd-resolved`
- `resolvectl`

### DNS Packet Capture Time Window

Approximate DNS activity captured:

`2026-08-12 20:33:43 UTC`  
to  
`2026-08-12 20:37:48 UTC`

---

# 1. DNS Environment Identification

The investigation began by identifying the current user, hostname, network configuration, system time, and DNS resolver configuration.

Commands:

~~~bash
whoami
hostname
date -u
ip -br addr
cat /etc/resolv.conf
resolvectl status
~~~

The monitored endpoint was identified as:

~~~text
User: analyst
Hostname: ubuntu-agent
IPv4 Address: 192.168.64.10/24
Interface: enp0s1
~~~

`/etc/resolv.conf` showed:

~~~text
nameserver 127.0.0.53
~~~

However, `resolvectl status` identified the actual upstream DNS server:

~~~text
Current DNS Server: 192.168.64.1
DNS Servers: 192.168.64.1
~~~

This demonstrates the local DNS resolution architecture:

~~~text
Application
    ↓
127.0.0.53
systemd-resolved local stub
    ↓
192.168.64.1
upstream DNS resolver
~~~

The distinction is important because `127.0.0.53` is not the external DNS resolver. It is the local `systemd-resolved` stub that forwards requests to the configured upstream resolver.

### Evidence

![DNS Environment and Resolver](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/01-dns-environment-and-resolver.png?raw=true)

---

# 2. Normal A Record Resolution

A normal IPv4 DNS lookup was performed against `example.com`.

Command:

~~~bash
dig example.com A
~~~

Observed result:

~~~text
status: NOERROR

QUESTION:
example.com. IN A

ANSWERS:
104.20.23.154
172.66.147.243

Query time: 32 msec
Server: 127.0.0.53#53
Protocol: UDP
~~~

The query returned two IPv4 addresses.

An `A` record maps a domain or hostname to an IPv4 address.

~~~text
Domain
   ↓
A Record
   ↓
IPv4 Address
~~~

The returned TTL was:

~~~text
24 seconds
~~~

TTL represents the period for which the DNS record may be cached before it should be refreshed.

### Evidence

![Example.com A Record](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/02-example-com-a-record.png?raw=true)

---

# 3. AAAA Record Analysis

An IPv6 lookup was performed.

Command:

~~~bash
dig example.com AAAA
~~~

Observed addresses:

~~~text
2606:4700:10::6814:179a
2606:4700:10::ac42:93f3
~~~

The result demonstrated the distinction between:

~~~text
A     → IPv4 address
AAAA  → IPv6 address
~~~

The query completed successfully with:

~~~text
status: NOERROR
~~~

---

# 4. NS Record Analysis

The authoritative name servers for `example.com` were queried.

Command:

~~~bash
dig example.com NS
~~~

Observed results:

~~~text
hera.ns.cloudflare.com.
elliott.ns.cloudflare.com.
~~~

An `NS` record identifies the authoritative name servers responsible for a DNS zone.

Conceptually:

~~~text
Domain
   ↓
NS Record
   ↓
Authoritative Name Server
~~~

NS information can provide useful infrastructure context during domain investigations.

---

# 5. MX Record Analysis

The mail exchange configuration was queried.

Command:

~~~bash
dig example.com MX
~~~

Observed result:

~~~text
example.com. 473 IN MX 0 .
~~~

The DNS query itself completed successfully:

~~~text
status: NOERROR
~~~

The result:

~~~text
MX 0 .
~~~

represents a null MX configuration indicating that the domain does not accept incoming email.

MX records are useful during phishing and infrastructure investigations because they help identify the mail infrastructure associated with a domain.

---

# 6. TXT Record Analysis

TXT records were queried.

Command:

~~~bash
dig example.com TXT
~~~

Observed values included:

~~~text
"v=spf1 -all"
"_k2n1y4vw3qtb4skdx9e7dxt97qrmmq9"
~~~

The SPF-related value:

~~~text
v=spf1 -all
~~~

indicates that the domain does not authorize systems to send email on its behalf.

The second TXT value was treated only as an observed DNS value because there was insufficient evidence to assign a specific purpose to it.

This demonstrates an important analyst principle:

~~~text
Observe
   ↓
Validate
   ↓
Interpret only what the evidence supports
   ↓
Avoid unsupported assumptions
~~~

---

# 7. DNS Tool Comparison

The same domain was reviewed using `host` and `nslookup`.

Commands:

~~~bash
host example.com
nslookup example.com
~~~

`host` returned a concise summary of:

- IPv4 addresses
- IPv6 addresses
- MX configuration

`nslookup` confirmed that the system was using:

~~~text
127.0.0.53#53
~~~

as its local DNS stub resolver.

Practical comparison:

~~~text
dig
→ detailed DNS investigation

host
→ quick DNS lookup

nslookup
→ basic DNS troubleshooting and resolution
~~~

For this investigation, `dig` provided the clearest record-level information.

---

# 8. PTR / Reverse DNS Investigation

A reverse DNS lookup was performed against:

~~~text
1.1.1.1
~~~

Command:

~~~bash
dig -x 1.1.1.1
~~~

Observed result:

~~~text
1.1.1.1.in-addr.arpa. 514 IN PTR one.one.one.one.
~~~

The lookup completed successfully:

~~~text
status: NOERROR
~~~

This demonstrates:

~~~text
A Record:
Domain → IPv4

PTR Record:
IP Address → Hostname
~~~

The result was also confirmed with:

~~~bash
host 1.1.1.1
~~~

which returned:

~~~text
1.1.1.1.in-addr.arpa domain name pointer one.one.one.one.
~~~

PTR records can provide additional infrastructure context when investigating unfamiliar IP addresses.

However, PTR information alone should not be treated as proof that an address is trusted or malicious.

### Evidence

![PTR Reverse DNS Query](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/03-ptr-reverse-dns-query.png?raw=true)

---

# 9. DNS Packet Capture

A packet capture was started on the Ubuntu endpoint.

Command:

~~~bash
sudo tcpdump -i any -nn -s0 -w lab-024-dns-traffic.pcap port 53
~~~

Command breakdown:

~~~text
-i any
Capture on all available interfaces.

-nn
Disable hostname and service-name resolution.

-s0
Capture complete packets.

-w lab-024-dns-traffic.pcap
Write packets to a PCAP file.

port 53
Capture traditional DNS traffic using port 53.
~~~

The resulting capture file was:

~~~text
/home/analyst/lab-024-dns-traffic.pcap
~~~

File verification:

~~~bash
ls -lh ~/lab-024-dns-traffic.pcap
~~~

Observed:

~~~text
-rw-r--r-- 1 tcpdump tcpdump 12K Aug 12 20:44 /home/analyst/lab-024-dns-traffic.pcap
~~~

### Evidence

![DNS Packet Capture Running](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/04-dns-packet-capture-running.png?raw=true)

---

# 10. Normal DNS Baseline Generation

While `tcpdump` was running, normal DNS traffic was generated.

Commands:

~~~bash
dig example.com A
dig example.com AAAA
dig example.com NS
dig example.com TXT
~~~

This created a normal baseline that could later be compared against the simulated suspicious DNS activity.

The investigation model was:

~~~text
Normal domain
    ↓
DNS query
    ↓
Resolver
    ↓
Successful response
~~~

---

# 11. Simulated Suspicious DNS Activity

Several intentionally non-existent authentication-themed domains were queried.

Initial queries:

~~~bash
dig secure-login-verification.test A
dig auth.secure-login-verification.test A
dig api.secure-login-verification.test A
~~~

The DNS server returned:

~~~text
status: NXDOMAIN
~~~

Additional repeated queries were generated:

~~~bash
for i in {1..5}; do
    dig +time=2 +tries=1 secure-login-verification.test A
    sleep 1
done
~~~

Multiple related subdomains were then generated:

~~~bash
for sub in login auth api portal account verify; do
    dig +time=2 +tries=1 "${sub}.secure-login-verification.test" A
    sleep 1
done
~~~

Domains included:

~~~text
secure-login-verification.test
login.secure-login-verification.test
auth.secure-login-verification.test
api.secure-login-verification.test
portal.secure-login-verification.test
account.secure-login-verification.test
verify.secure-login-verification.test
~~~

All of the simulated domains returned:

~~~text
NXDOMAIN
~~~

Key DNS fields included:

~~~text
QUERY: 1
ANSWER: 0
AUTHORITY: 1
~~~

Interpretation:

~~~text
QUERY: 1
One DNS question was submitted.

ANSWER: 0
No requested A record was returned.

NXDOMAIN
The requested DNS name did not exist.
~~~

### Evidence

![Suspicious NXDOMAIN Query](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/05-suspicious-nxdomain-query.png?raw=true)

---

# 12. Tshark DNS Field Extraction

Tshark was used to analyze the packet capture.

Command:

~~~bash
tshark -r ~/lab-024-dns-traffic.pcap \
-Y "dns" \
-T fields \
-e frame.time \
-e ip.src \
-e ip.dst \
-e dns.flags.response \
-e dns.qry.name \
-e dns.qry.type \
-e dns.flags.rcode
~~~

The command extracted:

- timestamp
- source IP
- destination IP
- DNS query/response flag
- queried domain
- DNS query type
- DNS response code

Important values:

~~~text
dns.flags.response = 0
DNS query

dns.flags.response = 1
DNS response

dns.flags.rcode = 0
NOERROR

dns.flags.rcode = 3
NXDOMAIN
~~~

---

# 13. Local Stub vs Upstream DNS Traffic

The initial Tshark analysis revealed two stages of DNS communication.

Example:

~~~text
127.0.0.1
    ↓
127.0.0.53

192.168.64.10
    ↓
192.168.64.1
~~~

A single logical DNS request could therefore appear twice because the packet capture was performed on:

~~~text
-i any
~~~

The complete path was:

~~~text
Application / dig
       ↓
127.0.0.53
systemd-resolved
       ↓
192.168.64.10
       ↓
192.168.64.1
Upstream DNS resolver
~~~

This created a potential counting issue.

Counting every observed DNS query would have counted both the local stub transaction and the upstream transaction.

To avoid this, the investigation isolated only endpoint-to-upstream-resolver traffic.

Command:

~~~bash
tshark -r ~/lab-024-dns-traffic.pcap \
-Y 'dns && ((ip.src == 192.168.64.10 && ip.dst == 192.168.64.1) || (ip.src == 192.168.64.1 && ip.dst == 192.168.64.10))' \
-T fields \
-e frame.time \
-e ip.src \
-e ip.dst \
-e dns.flags.response \
-e dns.qry.name \
-e dns.qry.type \
-e dns.flags.rcode
~~~

This produced the cleaner relationship:

~~~text
192.168.64.10 → 192.168.64.1
DNS query

192.168.64.1 → 192.168.64.10
DNS response
~~~

This filtering step prevented duplicate logical requests from influencing the analysis.

### Evidence

![DNS Tshark Field Analysis](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/06-dns-tshark-field-analysis.png?raw=true)

---

# 14. NXDOMAIN Response Isolation

Tshark was used to isolate only NXDOMAIN responses from the upstream DNS resolver.

Command:

~~~bash
tshark -r ~/lab-024-dns-traffic.pcap \
-Y 'dns.flags.response == 1 && dns.flags.rcode == 3 && ip.src == 192.168.64.1 && ip.dst == 192.168.64.10' \
-T fields \
-e frame.time \
-e ip.src \
-e ip.dst \
-e dns.qry.name \
-e dns.qry.type \
-e dns.flags.rcode
~~~

The filter showed:

~~~text
Source:       192.168.64.1
Destination:  192.168.64.10
Query Type:   1
RCODE:        3
~~~

DNS query type:

~~~text
1 = A
~~~

DNS response code:

~~~text
3 = NXDOMAIN
~~~

Observed domains included:

~~~text
secure-login-verification.test
auth.secure-login-verification.test
api.secure-login-verification.test
login.secure-login-verification.test
portal.secure-login-verification.test
account.secure-login-verification.test
verify.secure-login-verification.test
~~~

This confirmed that the upstream DNS resolver returned NXDOMAIN responses to the monitored endpoint.

### Evidence

![NXDOMAIN Response Analysis](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/07-nxdomain-response-analysis.png?raw=true)

---

# 15. DNS Query Frequency Analysis

The investigation next measured how frequently each suspicious-looking domain was queried.

Command:

~~~bash
tshark -r ~/lab-024-dns-traffic.pcap \
-Y 'dns.flags.response == 0 && ip.src == 192.168.64.10 && ip.dst == 192.168.64.1 && dns.qry.name contains "secure-login-verification"' \
-T fields \
-e dns.qry.name | sort | uniq -c | sort -nr
~~~

Observed frequency:

~~~text
6 secure-login-verification.test
2 auth.secure-login-verification.test
2 api.secure-login-verification.test
1 verify.secure-login-verification.test
1 portal.secure-login-verification.test
1 login.secure-login-verification.test
1 account.secure-login-verification.test
~~~

The base domain was observed six times because it was queried once initially and five additional times during the repeated-query loop.

`auth` and `api` were each observed twice because they were queried individually and later queried again during the subdomain-generation loop.

The resulting behavioral pattern was:

~~~text
Unusual domain observed
        ↓
NXDOMAIN confirmed
        ↓
Repeated resolution attempts
        ↓
Multiple related hostnames
        ↓
Frequency measured
        ↓
Behavioral anomaly established
~~~

Importantly:

~~~text
Anomaly ≠ Confirmed Compromise
~~~

### Evidence

![Repeated DNS Query Count](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/08-repeated-dns-query-count.png?raw=true)

---

# 16. Wireshark DNS Investigation

The PCAP was transferred to the Mac host and opened in Wireshark.

The initial filter used was:

~~~text
dns.qry.name contains "secure-login-verification"
~~~

A more precise filter was then applied:

~~~text
dns.qry.name contains "secure-login-verification" && ((ip.src == 192.168.64.10 && ip.dst == 192.168.64.1) || (ip.src == 192.168.64.1 && ip.dst == 192.168.64.10))
~~~

This removed the local loopback DNS traffic and displayed only communication between:

~~~text
Monitored endpoint:
192.168.64.10

Upstream DNS resolver:
192.168.64.1
~~~

The packet list showed repeated pairs of:

~~~text
192.168.64.10 → 192.168.64.1
Standard query A <domain>

192.168.64.1 → 192.168.64.10
Standard query response, No such name
~~~

### Evidence

![Wireshark DNS Overview](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/09-wireshark-dns-overview.png?raw=true)

---

# 17. Packet-Level NXDOMAIN Validation

One DNS response packet was inspected in detail.

Observed packet relationship:

~~~text
Source:
192.168.64.1

Destination:
192.168.64.10

Protocol:
DNS over UDP

Source Port:
53
~~~

Wireshark displayed:

~~~text
Standard query response, No such name
~~~

DNS details included:

~~~text
Transaction ID: 0x7575

Questions: 1

Answer RRs: 0

Authority RRs: 1

Additional RRs: 1
~~~

The requested domain was:

~~~text
secure-login-verification.test
~~~

The query type was:

~~~text
A
~~~

The class was:

~~~text
IN
~~~

Wireshark also correlated the response with:

~~~text
Request In: 18
~~~

and reported an approximate response time of:

~~~text
83.322 ms
~~~

The packet-level evidence therefore confirmed:

~~~text
Client:
192.168.64.10

        ↓

DNS A query:
secure-login-verification.test

        ↓

Resolver:
192.168.64.1

        ↓

DNS response:
NXDOMAIN / No such name

        ↓

Answer RRs:
0
~~~

### Evidence

![Wireshark NXDOMAIN Investigation](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/10-wireshark-nxdomain-investigation.png?raw=true)

---

# 18. Observed Activity

The monitored Ubuntu endpoint generated both normal and intentionally suspicious-looking DNS traffic.

Normal DNS activity included successful queries for:

~~~text
example.com
~~~

using several DNS record types.

The simulated anomalous activity involved repeated A-record requests for:

~~~text
secure-login-verification.test
~~~

and multiple related hostnames:

~~~text
login.secure-login-verification.test
auth.secure-login-verification.test
api.secure-login-verification.test
portal.secure-login-verification.test
account.secure-login-verification.test
verify.secure-login-verification.test
~~~

The upstream resolver returned:

~~~text
NXDOMAIN
~~~

for all simulated suspicious-looking names.

---

# 19. Evidence Summary

The investigation established the following evidence:

| Evidence | Finding |
|---|---|
| Monitored endpoint | `192.168.64.10` |
| Host | `ubuntu-agent` |
| Local DNS stub | `127.0.0.53` |
| Upstream DNS resolver | `192.168.64.1` |
| Suspicious query type | `A` |
| DNS response code | `3` |
| Response meaning | `NXDOMAIN` |
| Base domain query count | `6` |
| Multiple related subdomains | Yes |
| DNS response answers | `0` for NXDOMAIN queries |
| Packet capture available | Yes |
| Tshark validation | Yes |
| Wireshark validation | Yes |

---

# 20. Analysis

The DNS evidence demonstrated a pattern that could appear suspicious if discovered unexpectedly on a production endpoint.

The monitored system repeatedly attempted to resolve a non-existent domain and also queried several related authentication-themed hostnames.

Examples included:

~~~text
secure-login-verification.test
login.secure-login-verification.test
auth.secure-login-verification.test
api.secure-login-verification.test
portal.secure-login-verification.test
account.secure-login-verification.test
verify.secure-login-verification.test
~~~

The upstream DNS resolver consistently returned:

~~~text
RCODE 3
NXDOMAIN
~~~

The repeated resolution attempts and related hostname pattern represent an anomaly that would justify further investigation in a real SOC environment.

Possible suspicious explanations could include:

- malware attempting to locate command-and-control infrastructure;
- automated discovery of external infrastructure;
- dynamically generated domain activity;
- unauthorized software;
- scripted network activity.

However, several benign explanations could produce similar DNS behavior:

- application misconfiguration;
- incorrect domain configuration;
- stale application settings;
- broken scripts;
- browser extensions;
- software update mechanisms;
- user typing mistakes;
- testing activity.

Therefore, DNS evidence alone is insufficient to classify the endpoint as compromised.

---

# 21. Why the Activity Is Suspicious

The activity would be considered suspicious enough for additional investigation because:

1. The same non-existent domain was queried repeatedly.
2. Multiple related authentication-themed hostnames were queried.
3. All queries returned NXDOMAIN.
4. The requests occurred within a short time window.
5. The activity followed a recognizable automated pattern rather than a single isolated lookup.

However, none of these characteristics independently establish malicious intent.

---

# 22. Alternative Explanation

The strongest alternative explanation is legitimate automated or testing behavior.

In this lab, that explanation is confirmed because the queries were deliberately generated by the analyst.

The simulated activity was created using:

~~~bash
for i in {1..5}; do
    dig +time=2 +tries=1 secure-login-verification.test A
    sleep 1
done
~~~

and:

~~~bash
for sub in login auth api portal account verify; do
    dig +time=2 +tries=1 "${sub}.secure-login-verification.test" A
    sleep 1
done
~~~

The DNS anomaly therefore resulted from controlled lab activity rather than unauthorized software.

---

# 23. Risk

In this controlled environment:

~~~text
Risk: None / Simulated
~~~

In a production environment, unexplained repeated NXDOMAIN activity could indicate:

- malware activity;
- command-and-control discovery attempts;
- unwanted software;
- endpoint misconfiguration;
- compromised scripts;
- unauthorized automated activity.

The actual severity would depend on additional context, including:

- the endpoint involved;
- the user account;
- frequency and duration of the DNS activity;
- the process generating the requests;
- domain reputation;
- successful connections following DNS resolution;
- endpoint alerts;
- network IDS alerts;
- SIEM correlation.

---

# 24. Recommended Next Steps

If this activity were observed unexpectedly in production, the following investigation steps would be appropriate:

1. Identify the endpoint responsible for the DNS queries.
2. Identify the process that generated the DNS requests.
3. Review parent/child process relationships.
4. Determine whether the activity is expected for the affected host.
5. Review the query frequency and timing.
6. Search for additional related domains.
7. Investigate any successfully resolved infrastructure.
8. Review outbound connections associated with the querying process.
9. Correlate the DNS activity with endpoint telemetry.
10. Review SIEM alerts for the same host and time window.
11. Review IDS/network telemetry for related activity.
12. Perform IOC enrichment if real external domains or IP addresses are involved.
13. Escalate if additional evidence indicates unauthorized or malicious behavior.

---

# 25. Escalation Decision

### Controlled Lab Disposition

~~~text
Disposition: Simulated Benign Activity
Escalation: No
~~~

The DNS queries were intentionally generated during the lab.

Therefore, no incident escalation is required.

### Production Scenario

If the same behavior occurred without an authorized explanation:

~~~text
Disposition:
Suspicious / Requires Investigation

Escalation:
Conditional
~~~

Escalation would depend on supporting evidence from endpoint, network, SIEM, IDS, process, and threat-intelligence sources.

---

# 26. MITRE ATT&CK Mapping

No MITRE ATT&CK technique is assigned based solely on the NXDOMAIN behavior observed in this lab.

Repeated DNS queries and NXDOMAIN responses may occur during malicious activity, but the DNS evidence alone does not demonstrate a specific adversary technique.

ATT&CK mapping should only be added when the underlying behavior and supporting evidence justify the mapping.

This avoids forcing ATT&CK techniques into an investigation simply for presentation purposes.

---

# 27. Final Analyst Disposition

~~~text
Classification: Simulated Benign Activity
Confidence: High
Escalation Required: No
~~~

The activity appeared anomalous from a DNS perspective because repeated resolution attempts were made for multiple non-existent related domains.

However, the activity was intentionally generated as part of the controlled investigation.

No evidence indicated an actual compromise.

---

# 28. Final SOC Summary

DNS traffic from the monitored Ubuntu endpoint `192.168.64.10` was captured and analyzed using tcpdump, Tshark, and Wireshark. Normal DNS behavior was compared with intentionally generated requests for multiple non-existent authentication-themed domains.

The upstream resolver `192.168.64.1` returned `NXDOMAIN` responses for the simulated suspicious domains. Frequency analysis identified six queries for `secure-login-verification.test`, two queries each for the `auth` and `api` subdomains, and individual requests for several additional related hostnames.

Packet-level analysis confirmed A-record requests from the monitored endpoint and corresponding DNS response code `3` from the resolver. No answer resource records were returned for the NXDOMAIN responses.

Although repeated NXDOMAIN activity and related hostname patterns may justify further investigation in a production environment, DNS evidence alone is insufficient to confirm malicious activity. In this lab, the behavior was intentionally generated and was therefore classified as simulated benign activity with no escalation required.

---

# 29. Key DNS Concepts Learned

| DNS Concept | Practical Meaning |
|---|---|
| `A` | Domain → IPv4 |
| `AAAA` | Domain → IPv6 |
| `NS` | Authoritative DNS servers |
| `MX` | Mail receiving infrastructure |
| `TXT` | Text-based DNS metadata and policies |
| `PTR` | IP → hostname |
| `NXDOMAIN` | Requested DNS name does not exist |
| `TTL` | DNS cache lifetime |
| `RCODE 0` | NOERROR |
| `RCODE 3` | NXDOMAIN |
| Query | Client asks resolver for information |
| Response | Resolver returns the DNS result |

---

# 30. Investigation Lessons Learned

This lab demonstrated that DNS investigation requires more than checking whether a domain resolves.

A SOC analyst should examine:

~~~text
Source host
    ↓
DNS resolver
    ↓
Queried domain
    ↓
Record type
    ↓
Response code
    ↓
Resolved infrastructure
    ↓
Frequency
    ↓
Timing
    ↓
Related hostnames
    ↓
Endpoint/network context
    ↓
Disposition
~~~

A major lesson was the importance of avoiding duplicate counting.

Because the capture was performed across all interfaces, the same logical DNS request appeared both:

~~~text
127.0.0.1 ↔ 127.0.0.53
~~~

and:

~~~text
192.168.64.10 ↔ 192.168.64.1
~~~

Filtering specifically for the monitored endpoint and upstream DNS resolver prevented the same logical request from being counted twice.

Another important lesson was:

~~~text
NXDOMAIN ≠ Malware
~~~

Repeated NXDOMAIN responses may be anomalous and may justify investigation, but additional endpoint and network context is required before malicious intent can be established.

---

# 31. SOC English Practice

### Observation

> Multiple DNS queries were observed from the monitored endpoint.

### NXDOMAIN

> The DNS resolver returned NXDOMAIN responses for the requested domains.

### Frequency

> Repeated resolution attempts were identified for the same domain.

### Pattern

> Multiple related authentication-themed hostnames were queried within a short time window.

### Analyst Judgment

> The observed DNS pattern is anomalous but is not sufficient to confirm malicious activity.

### Additional Investigation

> Additional endpoint telemetry would be required to determine which process generated the DNS requests.

### Escalation

> Escalation would be appropriate only if additional evidence supported unauthorized or malicious activity.

### Disposition

> The activity was intentionally generated within the controlled lab environment and was classified as simulated benign activity.

---

# 32. CV Bullet

- Investigated DNS traffic using `dig`, `tcpdump`, Tshark, and Wireshark, analyzing DNS record types, NXDOMAIN responses, repeated query behavior, resolver communication, query frequency, and packet-level evidence for SOC triage.

---

# 33. Interview Explanation

**Question: How would you investigate suspicious DNS activity?**

I would first identify the endpoint responsible for the DNS activity and determine which DNS resolver processed the requests. I would examine the queried domain, DNS record type, response code, timestamps, and frequency of the activity.

I would determine whether the domain resolved successfully and inspect any returned IP addresses or related infrastructure. Repeated NXDOMAIN responses, unusual domain patterns, or multiple related subdomains could justify additional investigation, but I would not classify the activity as malicious based on DNS telemetry alone.

I would then correlate the activity with endpoint process information, network connections, SIEM alerts, IDS telemetry, and threat-intelligence information. The escalation decision would depend on whether the additional evidence supported unauthorized or malicious activity.

---

# 34. Evidence Index

## Screenshot 01 - DNS Environment and Resolver

![01-dns-environment-and-resolver.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/01-dns-environment-and-resolver.png?raw=true)

Purpose:

- Confirms host identity
- Confirms endpoint IP
- Shows local DNS stub
- Shows upstream resolver
- Establishes DNS environment

---

## Screenshot 02 - Example.com A Record

![02-example-com-a-record.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/02-example-com-a-record.png?raw=true)

Purpose:

- Demonstrates successful DNS resolution
- Shows `NOERROR`
- Shows A-record query
- Shows returned IPv4 addresses
- Shows TTL
- Shows DNS resolver

---

## Screenshot 03 - PTR Reverse DNS Query

![03-ptr-reverse-dns-query.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/03-ptr-reverse-dns-query.png?raw=true)

Purpose:

- Demonstrates reverse DNS
- Shows PTR record
- Maps `1.1.1.1` to `one.one.one.one`

---

## Screenshot 04 - DNS Packet Capture Running

![04-dns-packet-capture-running.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/04-dns-packet-capture-running.png?raw=true)

Purpose:

- Demonstrates packet capture
- Shows tcpdump collecting port 53 traffic
- Shows simultaneous DNS traffic generation

---

## Screenshot 05 - Suspicious NXDOMAIN Query

![05-suspicious-nxdomain-query.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/05-suspicious-nxdomain-query.png?raw=true)

Purpose:

- Shows authentication-themed domain queries
- Shows `NXDOMAIN`
- Demonstrates failed DNS resolution

---

## Screenshot 06 - Tshark DNS Field Analysis

![06-dns-tshark-field-analysis.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/06-dns-tshark-field-analysis.png?raw=true)

Purpose:

- Shows endpoint-to-resolver communication
- Displays timestamps
- Displays source/destination IPs
- Displays queried domains
- Displays query/response information

---

## Screenshot 07 - NXDOMAIN Response Analysis

![07-nxdomain-response-analysis.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/07-nxdomain-response-analysis.png?raw=true)

Purpose:

- Isolates DNS responses
- Shows RCODE `3`
- Confirms NXDOMAIN activity
- Shows resolver-to-endpoint responses

---

## Screenshot 08 - Repeated DNS Query Count

![08-repeated-dns-query-count.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/08-repeated-dns-query-count.png?raw=true)

Purpose:

- Demonstrates query-frequency analysis
- Confirms repeated base-domain requests
- Shows multiple related subdomains

---

## Screenshot 09 - Wireshark DNS Overview

![09-wireshark-dns-overview.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/09-wireshark-dns-overview.png?raw=true)

Purpose:

- Shows endpoint-to-resolver DNS traffic
- Shows query/response pairs
- Shows repeated suspicious-looking DNS activity
- Shows `No such name` responses

---

## Screenshot 10 - Wireshark NXDOMAIN Investigation

![10-wireshark-nxdomain-investigation.png](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-024/10-wireshark-nxdomain-investigation.png?raw=true)

Purpose:

- Provides packet-level NXDOMAIN evidence
- Shows client and resolver IP addresses
- Shows A-record query
- Shows `Answer RRs: 0`
- Shows `No such name`
- Correlates request and response packets

---

# 35. Skills Demonstrated

This lab demonstrates practical experience with:

- DNS investigation
- DNS record analysis
- A records
- AAAA records
- NS records
- MX records
- TXT records
- PTR records
- DNS TTL interpretation
- DNS response codes
- NXDOMAIN analysis
- Linux DNS configuration
- `systemd-resolved`
- `dig`
- `host`
- `nslookup`
- tcpdump
- PCAP collection
- Tshark
- Wireshark
- DNS packet filtering
- query frequency analysis
- anomaly identification
- evidence correlation
- false-positive awareness
- SOC analyst disposition
- evidence-based escalation decisions

---

# 36. Lab Progression

~~~text
DNS environment identification
        ↓
Normal DNS resolution
        ↓
DNS record analysis
        ↓
Reverse DNS
        ↓
Packet capture
        ↓
Simulated suspicious DNS activity
        ↓
NXDOMAIN responses
        ↓
Tshark field extraction
        ↓
Endpoint/resolver filtering
        ↓
Query-frequency analysis
        ↓
Wireshark validation
        ↓
Packet-level investigation
        ↓
Analyst assessment
        ↓
Disposition
~~~

---

# Conclusion

Lab 024 demonstrated how DNS telemetry can be investigated from both command-line and packet-level perspectives.

The investigation progressed beyond simply performing DNS lookups by capturing DNS traffic, identifying the monitored endpoint and upstream resolver, isolating NXDOMAIN responses, measuring repeated-query behavior, and validating the findings directly within Wireshark.

The most important analytical conclusion is that unusual DNS behavior should be treated as evidence requiring context rather than automatic proof of compromise.

Repeated NXDOMAIN responses can indicate activity worth investigating, but the analyst must correlate DNS evidence with endpoint, network, SIEM, IDS, process, and threat-intelligence context before reaching a malicious disposition.

In this controlled lab, the suspicious-looking DNS pattern was intentionally generated and was therefore classified as simulated benign activity.

---
