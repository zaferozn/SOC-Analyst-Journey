# Case 005 - Suspicious URL Investigation

## Executive Summary

This case documents the investigation of a suspicious URL identified in a package-delivery-themed email within a controlled TryHackMe training environment.

The email displayed package-tracking language while directing the recipient to a URL hosted on `devret[.]xyz`. Analysis of the email source also identified a remote tracking pixel associated with the same domain.

The investigation included URL analysis, DNS resolution, WHOIS infrastructure review, VirusTotal reputation analysis, passive DNS review, and examination of existing urlscan.io records.

The domain resolved to two Amazon-associated IP addresses. VirusTotal showed no engines classifying the domain as malicious in the primary detection ratio, although some vendors categorized the domain as suspicious. Existing urlscan.io records confirmed that the exact URL observed in the email had previously been scanned.

The historical scan showed an HTTP-to-HTTPS redirect on the same domain followed by an HTTP `404 Not Found` response. No credential-harvesting page, malware delivery, external malicious redirect, or confirmed compromise was demonstrated by the available evidence.

**Final Disposition: Suspicious / Unconfirmed**

---

## Objective

The objective of this investigation was to determine:

- What domain the suspicious hyperlink actually used
- Whether the visible text matched the technical destination
- Whether tracking mechanisms were present
- Which IP addresses the domain resolved to
- Who owned the associated infrastructure
- Whether reputation sources identified malicious or suspicious activity
- Whether historical web-scan evidence existed
- Whether the URL redirected to another domain
- Whether credential harvesting or malware delivery could be confirmed
- Whether escalation was justified

---

## Environment / Data Source

**Investigation Type:** Suspicious URL / phishing-link investigation

**Training Source:** TryHackMe - Phishing Emails in Action

**Analysis Environment:** TryHackMe AttackBox

**Tools Used:**

- Linux `dig`
- Linux `whois`
- VirusTotal
- urlscan.io
- TryHackMe email-source evidence

**Primary Domain:**

`devret[.]xyz`

**Observed URL Pattern:**

`hxxp://devret[.]xyz/4833mt11254939vf6888zq22032si1269du1508rr`

### Investigation Safety

The suspicious hyperlink was not opened directly.

The full URL was not submitted as a new public urlscan.io scan.

Existing historical scan results were searched instead.

VirusTotal was used for passive domain lookup.

---

## Observed Activity

The investigated email used package-delivery language designed to encourage recipient interaction.

The visible message included package-tracking text similar to:

`Track your package`

However, inspection of the underlying HTML showed that the hyperlink destination pointed to:

`devret[.]xyz`

The source also contained a remote tracking image:

`hxxp://devret[.]xyz/Creatives/Tracking.png`

This created a mismatch between the user-facing package-tracking message and the underlying technical destination.

---

## Evidence

### 1. Suspicious URL and Tracking-Pixel Source

The email source revealed:

- Package-tracking display text
- Hyperlink destination hosted on `devret[.]xyz`
- A long unique-looking URL path
- A remote tracking pixel
- Additional resources hosted on the same domain

![Suspicious URL Source](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/01-suspicious-url-source.png?raw=true)

### Analyst Interpretation

The visible message encouraged the recipient to interact with a package-tracking link, while the underlying URL used an unrelated domain.

The tracking pixel provided additional evidence that the email was designed to interact with remote infrastructure.

These observations justified further investigation but did not independently prove malicious activity.

---

## DNS Analysis

The following queries were performed:

    dig devret.xyz A +short
    dig devret.xyz MX +short
    dig devret.xyz NS +short

Observed A records:

    15.197.225.128
    3.33.251.168

Observed nameservers:

    ns17.domaincontrol.com
    ns18.domaincontrol.com

No MX record was returned.

![DNS Resolution](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/02-dns-resolution.png?raw=true)

### Analyst Interpretation

The domain was actively resolving at the time of the investigation.

The lack of an MX record was recorded but was not considered suspicious by itself because a web domain does not require an MX record unless it is intended to receive email.

---

## Infrastructure Analysis

### 3. WHOIS - 15.197.225.128

WHOIS analysis identified:

    IP: 15.197.225.128
    NetName: AT-88-Z
    Organization: Amazon Technologies Inc.
    NetType: Direct Allocation

![First IP WHOIS Analysis](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/03-ip-whois-amazon.png?raw=true)

The IP address was allocated to Amazon Technologies Inc.

This identified the infrastructure provider but did not determine whether the associated activity was legitimate or malicious.

Cloud infrastructure can support both legitimate and abusive activity.

---

### 4. WHOIS - 3.33.251.168

The second A record was also investigated.

Observed information included:

    IP: 3.33.251.168
    NetName: AT-88-Z
    Organization: Amazon Technologies Inc.
    Country: US

![Second IP WHOIS Analysis](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/04-second-ip-whois-amazon.png?raw=true)

Both current A records were therefore associated with Amazon Technologies Inc.

The WHOIS country information was treated as registration information rather than proof of the physical server location.

---

## VirusTotal Analysis

### 5. Domain Reputation

VirusTotal was searched for:

`devret.xyz`

The reviewed result showed:

    Malicious detections: 0/91
    Registrar: GoDaddy.com, LLC

The primary detection ratio showed no engines classifying the domain as malicious.

However, vendors including `alphaMountain.ai` and `Forcepoint ThreatSeeker` categorized the domain as suspicious.

![VirusTotal Domain Detection](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/05-virustotal-domain-detection.png?raw=true)

### Analyst Interpretation

A `0/91` malicious detection ratio does not mean that a domain is automatically safe.

The absence of malicious detections was considered alongside the suspicious vendor categorizations and the context in which the URL appeared.

---

### 6. VirusTotal DNS Details

VirusTotal showed the same current A records identified through manual DNS analysis:

    15.197.225.128
    3.33.251.168

The nameservers also matched the manual results:

    ns17.domaincontrol.com
    ns18.domaincontrol.com

![VirusTotal DNS Details](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/06-virustotal-dns-details.png?raw=true)

This provided cross-source validation of the current DNS infrastructure.

---

### 7. Passive DNS History

VirusTotal displayed multiple historical IP addresses associated with `devret.xyz`.

Examples included:

    15.197.225.128
    3.33.251.168
    15.197.148.33
    3.33.130.190
    45.56.79.23
    45.79.19.196
    45.33.30.197
    45.33.20.235
    198.58.118.167

Some historical IP addresses had small numbers of security-vendor detections.

![VirusTotal Passive DNS History](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/07-virustotal-passive-dns-history.png?raw=true)

### Analyst Interpretation

Historical IP detections were not treated as direct evidence that `devret.xyz` was malicious.

Shared infrastructure can host multiple unrelated domains, so historical IP reputation requires context.

---

## urlscan.io Analysis

### 8. Existing Scan Search

Instead of creating a new scan, existing urlscan.io records were searched using:

`page.domain:devret.xyz`

The search returned multiple historical scans, including results containing the same long URL path observed in the original email.

![urlscan Existing Scan Search](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/08-urlscan-existing-scan-search.png?raw=true)

### Analyst Interpretation

Existing public scans allowed the suspicious URL to be investigated without directly visiting the domain or creating an additional public submission.

---

### 9. Exact URL Historical Scan

An existing urlscan.io result matching the URL from the email was reviewed.

Observed information included:

    Submitted URL:
    hxxp://devret[.]xyz/4833mt11254939vf6888zq22032si1269du1508rr

    Effective URL:
    hxxps://devret[.]xyz/4833mt11254939vf6888zq22032si1269du1508rr

    Main IP:
    15.197.225.128

    Infrastructure:
    Amazon-associated

    urlscan.io Verdict:
    No classification

No webpage screenshot was available.

![urlscan Exact URL Summary](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/09-urlscan-exact-url-summary.png?raw=true)

### Analyst Interpretation

The exact URL path from the email existed in historical urlscan.io records.

However, no rendered webpage was available, preventing visual confirmation of credential-harvesting or impersonation content.

---

### 10. Redirect Analysis

The scan showed the following redirect sequence:

    hxxp://devret[.]xyz/<unique-path>
            ↓
        HTTP 307
            ↓
    hxxps://devret[.]xyz/<same-path>

![urlscan HTTP to HTTPS Redirect](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/10-urlscan-http-to-https-redirect.png?raw=true)

### Analyst Interpretation

The redirect upgraded the connection from HTTP to HTTPS.

The domain and path remained unchanged.

No redirect to a different external domain was observed.

---

### 11. HTTP Response Analysis

The HTTP transaction showed:

    Method: GET
    Status: 404
    Protocol: HTTP/2
    IP: 15.197.225.128
    MIME type: text/plain

![urlscan HTTP 404 Response](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/11-urlscan-http-404-response.png?raw=true)

### Analyst Interpretation

At the time of the historical scan, the exact HTTPS path returned:

`404 Not Found`

This does not prove that the URL was historically harmless.

Possible explanations include:

- The content had expired
- The content had been removed
- The URL required a specific tracking condition
- The resource was unavailable at the time of scanning
- The historical behavior of the URL may have differed

No active phishing page could therefore be confirmed.

---

### 12. Extracted Indicators

urlscan.io extracted several indicators from the stored scan.

These included:

    Domain:
    devret.xyz

    IP:
    15.197.225.128

    Additional indicators:
    SHA-256 resource hashes

![urlscan Extracted Indicators](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-005/12-urlscan-extracted-indicators.png?raw=true)

### Analyst Interpretation

Extracted indicators were treated as investigative artifacts rather than automatic indicators of maliciousness.

The presence of a domain, IP address, or hash in an analysis platform does not independently establish malicious activity.

---

## IOC Summary

| Indicator | Type | Assessment |
|---|---|---|
| `devret[.]xyz` | Domain | Suspicious / Unconfirmed |
| `15.197.225.128` | IPv4 | Amazon infrastructure; maliciousness not established |
| `3.33.251.168` | IPv4 | Amazon infrastructure; maliciousness not established |
| `/4833mt11254939vf6888zq22032si1269du1508rr` | URL Path | Observed in the suspicious email and historical urlscan records |
| `/Creatives/Tracking.png` | Remote Resource | Tracking-pixel behavior observed |

---

## Analysis

The investigated email used a package-tracking theme to encourage recipient interaction.

Inspection of the HTML source showed that the underlying destination was `devret[.]xyz`, rather than an identifiable postal or delivery-service domain.

The same source contained a remote tracking pixel hosted on the investigated infrastructure.

DNS analysis confirmed that the domain remained active and resolved to two IP addresses associated with Amazon Technologies Inc.

VirusTotal showed no malicious detections in the primary domain detection ratio, although some vendors categorized the domain as suspicious.

Passive DNS records demonstrated that the domain had used several IP addresses historically.

Existing urlscan.io results showed that the exact URL path from the email had previously been scanned.

The historical request sequence showed an HTTP `307` redirect from HTTP to HTTPS while remaining on the same domain.

The final request returned an HTTP `404 Not Found` response.

The available evidence did not demonstrate credential harvesting, malware delivery, successful exploitation, or endpoint compromise.

The most appropriate conclusion is therefore that the URL was suspicious but not conclusively malicious based on the collected evidence.

---

## Confirmed Facts

The investigation confirmed:

- The email used a package-tracking theme.
- The underlying hyperlink pointed to `devret[.]xyz`.
- The email contained a remote tracking pixel.
- The domain resolved to `15.197.225.128` and `3.33.251.168`.
- Both current IP addresses were associated with Amazon Technologies Inc.
- VirusTotal showed `0/91` malicious detections in the primary domain result.
- Some VirusTotal vendors categorized the domain as suspicious.
- Historical passive DNS relationships existed.
- Multiple historical urlscan.io records existed.
- The exact suspicious URL path appeared in historical scans.
- The observed redirect changed HTTP to HTTPS.
- No external-domain redirect was observed.
- The final HTTPS request returned HTTP `404 Not Found`.

---

## Unconfirmed / Unknown

The investigation did not establish:

- Whether the URL previously hosted a phishing page
- Whether credentials were requested
- Whether credentials were submitted
- Whether malware was delivered
- Whether the recipient clicked the URL
- Whether an endpoint contacted the infrastructure
- Whether compromise occurred
- Whether the URL behaved differently at an earlier time

These unknowns prevent classification as confirmed malicious activity.

---

## Risk

The main risk is social engineering designed to persuade recipients to interact with an unrelated external domain.

If the URL had previously hosted malicious content, possible consequences could include:

- Credential theft
- Account compromise
- Malware delivery
- User tracking
- Additional phishing activity
- Unauthorized access

These outcomes were not demonstrated by the available evidence.

**Risk Rating:** Medium

**Confidence:** Medium

---

## Recommended Next Steps

In a real SOC environment:

1. Verify whether the recipient clicked the suspicious URL.
2. Search DNS, proxy, firewall, and endpoint logs for `devret[.]xyz`.
3. Search the email environment for additional messages containing the same domain.
4. Review browser and endpoint telemetry if user interaction occurred.
5. Check for authentication anomalies involving the affected user.
6. Preserve the original email and extracted indicators.
7. Monitor or block the domain according to organizational policy and threat confidence.
8. Escalate if evidence shows credential submission, malware execution, suspicious downloads, or additional malicious activity.

---

## Escalation Decision

**Conditional escalation recommended.**

The URL itself does not prove that compromise occurred.

Escalation becomes more strongly justified if additional evidence identifies:

- User interaction
- Credential submission
- File download
- Endpoint execution
- Suspicious network activity
- Authentication anomalies
- Additional confirmed malicious indicators

Without such evidence, the appropriate disposition remains:

**Suspicious / Unconfirmed**

---

## MITRE ATT&CK Mapping

| Technique | Name | Relevance |
|---|---|---|
| T1566.002 | Phishing: Spearphishing Link | The email attempted to encourage user interaction through a package-themed hyperlink pointing to an external domain. |

No additional MITRE ATT&CK techniques were mapped because credential submission, malware execution, or successful exploitation was not demonstrated.

---

## Final SOC Summary

A suspicious package-tracking URL associated with `devret[.]xyz` was investigated using static email analysis, DNS queries, WHOIS data, VirusTotal, passive DNS information, and existing urlscan.io records.

The email contained a package-themed hyperlink and a remote tracking pixel associated with the same domain. Current DNS records resolved the domain to Amazon-associated infrastructure.

VirusTotal showed no malicious-engine detections in the primary domain result, although some vendors classified the domain as suspicious.

Historical urlscan.io evidence confirmed that the exact URL observed in the email had previously been scanned. The recorded activity showed an HTTP-to-HTTPS redirect on the same domain followed by an HTTP `404 Not Found` response.

No credential-harvesting page, malware delivery, external malicious redirect, or confirmed compromise was demonstrated.

The final disposition was **Suspicious / Unconfirmed**, with further escalation dependent on evidence of user interaction or downstream malicious activity.

---

## Lessons Learned

This investigation reinforced several SOC principles:

- Visible hyperlink text should not be trusted without inspecting the technical destination.
- Tracking pixels can provide useful investigative context.
- Infrastructure ownership alone does not establish maliciousness.
- A `0/91` VirusTotal result does not automatically mean a domain is safe.
- Historical IP detections require careful interpretation when shared infrastructure is involved.
- Existing sandbox and urlscan.io records can provide evidence without directly visiting suspicious infrastructure.
- HTTP redirects should be interpreted precisely.
- An HTTP `404` response does not prove that a URL was historically benign.
- Suspicious indicators must be separated from confirmed malicious behavior.
- SOC conclusions should clearly distinguish confirmed facts from assumptions and unknown information.

---

## Final Disposition

    Classification: Suspicious / Unconfirmed
    Risk: Medium
    Confidence: Medium
    Credential Harvesting: Not confirmed
    Malware Delivery: Not confirmed
    External Redirect: Not observed
    User Interaction: Unknown
    Compromise: Not confirmed
    Escalation: Conditional
