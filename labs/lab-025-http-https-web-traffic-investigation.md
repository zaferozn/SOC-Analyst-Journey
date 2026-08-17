# Lab 025 - HTTP/HTTPS Web Traffic Investigation

## Executive Summary

This lab investigated HTTP and HTTPS web traffic captured from an Ubuntu endpoint in a controlled environment.

Traffic was generated using `curl` against `example.com` and captured with `tcpdump`. The resulting PCAP was analyzed with `tshark` to compare the visibility available in unencrypted HTTP traffic with encrypted HTTPS traffic.

The investigation confirmed that HTTP requests exposed application-layer details such as request method, host, URI, and User-Agent. HTTP responses also revealed status codes and content type.

In contrast, HTTPS traffic exposed network and TLS metadata, including destination IP addresses, TCP port 443, and Server Name Indication (SNI), while the underlying HTTP request content was not visible in the packet capture.

The lab demonstrates a core SOC investigation principle: encryption reduces direct application-layer visibility, but useful network and TLS metadata can still support investigation.

---

## Objective

The objective of this lab was to:

- capture HTTP and HTTPS web traffic
- identify HTTP requests and responses
- correlate request and response frames
- extract HTTP methods, hosts, URIs, and User-Agent fields
- inspect HTTPS/TLS traffic
- identify SNI values
- compare visibility between HTTP and HTTPS
- document the limitations of passive packet inspection when encryption is used

---

## Environment / Data Source

**Host:** `ubuntu-agent`

**Host IP:** `192.168.64.10`

**Network Interface:** `enp0s1`

**Tools:**
- `tcpdump`
- `curl`
- `tshark`

**Traffic Target:** `example.com`

**Data Source:** Packet capture

**PCAP File:** `/home/analyst/lab-025-web-traffic.pcap`

**PCAP Size:** Approximately `36K`

**Packets Captured:** `112`

**Packets Dropped:** `0`

**Time Window:** 17 August 2026

---

## Traffic Capture

The packet capture was created using:

    sudo tcpdump -i enp0s1 -nn -s 0 \
    -w ~/lab-025-web-traffic.pcap \
    'tcp port 80 or tcp port 443'

The capture successfully recorded:

    112 packets captured
    112 packets received by filter
    0 packets dropped by kernel

The resulting PCAP size was approximately:

    36K

---

## Traffic Generation

HTTP and HTTPS traffic was generated using `curl`.

Commands included:

    curl -v http://example.com/

    curl -I http://example.com/

    curl -A "SOC-Lab-Agent/1.0" -v http://example.com/

    curl -v https://example.com/

    curl -I https://example.com/

The custom User-Agent was intentionally added so that a distinctive application-layer indicator could be identified during packet analysis.

---

## Observed Activity

### HTTP Requests

The HTTP request analysis identified three requests originating from:

    192.168.64.10

Observed destination IP addresses:

    172.66.147.243
    104.20.23.154

The requests targeted:

    example.com

Observed request methods:

    GET
    HEAD
    GET

The requested URI was:

    /

Observed User-Agent values:

    curl/8.5.0
    SOC-Lab-Agent/1.0

---

## Evidence 1 - HTTP Request Analysis

The following command was used:

    tshark -r ~/lab-025-web-traffic.pcap \
    -Y "http.request" \
    -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e http.request.method \
    -e http.host \
    -e http.request.uri \
    -e http.user_agent

Observed output:

    30  192.168.64.10  172.66.147.243  GET   example.com  /  curl/8.5.0
    42  192.168.64.10  172.66.147.243  HEAD  example.com  /  curl/8.5.0
    52  192.168.64.10  104.20.23.154   GET   example.com  /  SOC-Lab-Agent/1.0

![HTTP Requests](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-025/01-http-requests.png)

### Interpretation

Because HTTP is unencrypted, application-layer information was directly visible in the PCAP.

The analyst could identify:

- source IP address
- destination IP address
- HTTP method
- hostname
- URI
- User-Agent

This level of visibility allows direct inspection of web requests during packet analysis.

---

## Evidence 2 - HTTP Response Analysis

The following command was used:

    tshark -r ~/lab-025-web-traffic.pcap \
    -Y "http.response" \
    -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e http.response.code \
    -e http.response.phrase \
    -e http.content_type

Observed output:

    33  172.66.147.243  192.168.64.10  200  OK  text/html
    44  172.66.147.243  192.168.64.10  200  OK  text/html
    55  104.20.23.154   192.168.64.10  200  OK  text/html

![HTTP Responses](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-025/02-http-responses.png)

### Interpretation

All three HTTP requests received successful server responses.

Observed response status:

    200 OK

Observed content type:

    text/html

This indicates that the server successfully processed the requests and returned HTML content.

---

## Evidence 3 - HTTP Request and Response Correlation

The following command was used:

    tshark -r ~/lab-025-web-traffic.pcap \
    -Y "http" \
    -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e http.request.method \
    -e http.host \
    -e http.request.uri \
    -e http.response.code

Observed sequence:

    Frame 30  → GET  example.com /
    Frame 33  → 200

    Frame 42  → HEAD example.com /
    Frame 44  → 200

    Frame 52  → GET  example.com /
    Frame 55  → 200

![HTTP Request Response Sequence](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-025/03-http-request-response-sequence.png)

### Interpretation

The frame sequence allowed individual HTTP requests to be correlated with their corresponding responses.

The activity can be summarized as:

    GET  → 200 OK
    HEAD → 200 OK
    GET  → 200 OK

This demonstrates how an analyst can reconstruct basic web transaction activity directly from packet capture evidence.

---

## HTTPS / TLS Analysis

HTTPS traffic was also present in the PCAP.

Observed destination IP addresses:

    104.20.23.154
    172.66.147.243

Observed destination port:

    443

TLS traffic exposed Server Name Indication values corresponding to:

    example.com

---

## Evidence 4 - TLS Server Name Indication

The following command was used:

    tshark -r ~/lab-025-web-traffic.pcap \
    -Y 'tls.handshake.extensions_server_name == "example.com"' \
    -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e tcp.dstport \
    -e tls.handshake.extensions_server_name

Observed output:

    64  192.168.64.10  104.20.23.154   443  example.com
    91  192.168.64.10  172.66.147.243  443  example.com

![HTTPS TLS SNI](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-025/04-https-tls-sni.png)

### Interpretation

Although HTTPS encrypts application-layer content, some TLS metadata remained visible.

The packet capture revealed:

- client IP address
- destination IP address
- destination port
- TLS activity
- SNI hostname

The SNI field exposed:

    example.com

This allowed the destination hostname to be identified without decrypting the underlying HTTPS application data.

---

## Evidence 5 - No Visible HTTP Request Within HTTPS

The following command was used:

    tshark -r ~/lab-025-web-traffic.pcap \
    -Y 'tcp.port == 443 && http.request' \
    -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e http.request.method \
    -e http.host

The command returned no matching packets.

![HTTPS No Visible HTTP Request](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-025/05-https-no-visible-http-request.png)

### Interpretation

The absence of visible HTTP request fields within the TCP/443 traffic supports the conclusion that the application-layer HTTP content was encrypted.

The analyst could still observe:

    TCP connection
    TLS handshake activity
    destination IP address
    destination port
    packet timing
    packet size
    SNI

However, fields such as the following were not directly visible:

    HTTP method
    HTTP headers
    URI
    User-Agent
    response body
    request body

This demonstrates the practical visibility limitation introduced by HTTPS encryption.

---

## HTTP vs HTTPS Comparison

### HTTP

Visible fields included:

    GET
    HEAD
    Host
    URI
    User-Agent
    Response Code
    Content-Type

Example:

    GET /
    Host: example.com
    User-Agent: SOC-Lab-Agent/1.0

### HTTPS

Visible evidence included:

    Source IP
    Destination IP
    TCP port 443
    TLS handshake metadata
    SNI
    Packet timing
    Packet size

The underlying HTTP request itself was not directly visible.

---

## Analysis

The captured activity was consistent with intentionally generated web traffic from the laboratory host.

The HTTP portion of the capture provided direct application-layer visibility.

The analyst was able to identify:

- request methods
- requested host
- requested URI
- User-Agent values
- HTTP response codes
- content type

The HTTPS portion provided significantly less application-layer visibility.

The analyst could still identify the remote IP addresses and determine that TLS metadata contained the hostname:

    example.com

However, the underlying HTTP request information was not directly recoverable from the packet capture without decryption material or an additional visibility source.

This distinction is operationally important because SOC analysts frequently investigate encrypted traffic using metadata rather than packet payload content.

---

## Background Traffic

The PCAP also contained TLS traffic associated with:

    changelogs.ubuntu.com

Observed destination:

    185.125.190.48

This traffic was not part of the intentionally generated `example.com` activity.

It represented unrelated background system traffic originating from the Ubuntu endpoint.

### Analyst Interpretation

A packet capture may contain legitimate background activity unrelated to the event being investigated.

Analysts must therefore separate relevant traffic from unrelated system-generated activity before reaching conclusions.

Within the context of this lab, the `changelogs.ubuntu.com` traffic was not treated as suspicious.

---

## Risk

No malicious activity was identified during this controlled lab.

However, the investigation demonstrates several risks relevant to real SOC environments.

### Unencrypted HTTP Risk

HTTP can expose sensitive information in plaintext, potentially including:

- authentication information
- session identifiers
- requested resources
- form data
- application headers
- internal hostnames
- User-Agent information

### HTTPS Investigation Limitation

HTTPS protects application-layer content from passive inspection.

While this improves confidentiality, network analysts may need to rely on:

- TLS metadata
- DNS evidence
- proxy logs
- endpoint telemetry
- firewall logs
- SIEM correlation
- certificate information
- network behavior

to investigate suspicious encrypted connections.

---

## Recommended Next Steps

If similar traffic appeared during a real SOC investigation:

1. Identify the originating host and user.
2. Confirm whether the destination hostname is expected.
3. Review DNS resolution history for the hostname.
4. Check the destination IP against threat intelligence sources.
5. Review endpoint process telemetry to determine which application initiated the connection.
6. Correlate the network activity with proxy, firewall, EDR, or SIEM logs.
7. Review TLS metadata for unusual characteristics.
8. Investigate abnormal User-Agent strings.
9. Compare requested destinations against known or approved services.
10. Escalate if the destination, process, timing, or surrounding activity cannot be explained.

---

## MITRE ATT&CK Mapping

No direct MITRE ATT&CK mapping is required for this lab because the captured activity was intentionally generated benign traffic.

In a real incident, suspicious web communication could potentially relate to:

    T1071.001 - Application Layer Protocol: Web Protocols

However, no malicious behavior was established in this laboratory activity.

---

## Final SOC Summary

A packet capture from host `192.168.64.10` was analyzed to compare HTTP and HTTPS network visibility. HTTP traffic to `example.com` exposed application-layer details including GET and HEAD methods, URI `/`, User-Agent values, and `200 OK` responses. HTTPS traffic to the same destination exposed network and TLS metadata, including destination IP addresses, TCP port 443, and SNI values identifying `example.com`, while the underlying HTTP request content was not visible due to encryption. Additional TLS traffic associated with `changelogs.ubuntu.com` was identified as unrelated background system activity. No malicious behavior was observed. The investigation demonstrated the importance of correlating packet metadata, application-layer fields, and contextual evidence when analyzing web traffic in SOC environments.

---

## Lessons Learned

### 1. HTTP Provides Direct Application-Layer Visibility

Unencrypted HTTP allowed direct inspection of:

    method
    host
    URI
    User-Agent
    response code
    content type

### 2. HTTPS Changes the Investigation Approach

With HTTPS, the analyst could not directly inspect the HTTP request.

Instead, the investigation relied on:

    IP addresses
    ports
    TLS metadata
    SNI
    packet behavior

### 3. Empty Results Can Still Be Meaningful Evidence

The filter:

    tcp.port == 443 && http.request

returned no results.

This was not an analysis failure.

The absence of HTTP request fields supported the conclusion that the application-layer request was encrypted.

### 4. Request and Response Frames Can Be Correlated

Example:

    Frame 30 → GET
    Frame 33 → 200 OK

### 5. User-Agent Values Can Be Useful Investigation Indicators

The custom value:

    SOC-Lab-Agent/1.0

was clearly visible in the HTTP traffic.

In real investigations, unusual User-Agent values may help identify:

- scripts
- automated tools
- malware
- scanners
- non-browser communication

### 6. Packet Captures Contain Background Noise

The presence of:

    changelogs.ubuntu.com

demonstrated that not every connection in a PCAP belongs to the event being investigated.

Relevant evidence must be separated from legitimate background activity.

### 7. Encryption Does Not Eliminate Network Visibility

HTTPS hides application content, but useful investigative metadata can remain available.

An analyst may still determine:

    who connected
    where they connected
    when they connected
    which port was used
    which hostname may have been requested
    how the connection behaved

---

## SOC English Practice

### Analyst Sentence 1

The packet capture revealed three unencrypted HTTP requests from the investigated host to `example.com`.

### Analyst Sentence 2

The HTTP traffic exposed request methods, destination hostnames, URIs, and User-Agent values.

### Analyst Sentence 3

All observed HTTP requests received successful `200 OK` responses.

### Analyst Sentence 4

TLS analysis identified `example.com` through the Server Name Indication field.

### Analyst Sentence 5

The underlying HTTP request content was not directly visible within the HTTPS traffic because the application data was encrypted.

### Analyst Sentence 6

Additional TLS traffic to `changelogs.ubuntu.com` was assessed as unrelated background system activity.

### Analyst Sentence 7

No malicious behavior was identified in the analyzed traffic.

### Analyst Sentence 8

The investigation demonstrated how encrypted traffic can still be analyzed using network and TLS metadata.

---

## Interview Explanation

In this lab, I captured both HTTP and HTTPS traffic from an Ubuntu endpoint and analyzed the resulting PCAP with `tshark`.

For HTTP, I was able to directly inspect the request methods, hostname, URI, User-Agent, and response codes because the application traffic was unencrypted.

I then compared this with HTTPS traffic. I could still observe the destination IP addresses, TCP port 443, TLS traffic, and the SNI value for `example.com`, but the actual HTTP request fields were no longer visible because the application-layer traffic was encrypted.

I also identified unrelated background TLS traffic to `changelogs.ubuntu.com`, which reinforced the importance of distinguishing relevant evidence from normal system activity during network investigations.

The main lesson was that encryption changes the type of evidence available to an analyst, but it does not eliminate the ability to investigate network behavior.

