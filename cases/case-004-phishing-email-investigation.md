# Case 004 - Suspicious Phishing Email Investigation

## Executive Summary

This case documents the investigation of a suspicious email containing tracking-style URLs associated with the `teckbe.com` domain.

The investigation included email header analysis, embedded URL identification, DNS and reverse DNS analysis, infrastructure enrichment, URLScan analysis, and VirusTotal reputation checks.

The available evidence identified tracking and redirect infrastructure but did not confirm credential harvesting, malware delivery, or other malicious activity.

Current reputation sources did not classify the investigated domain or the exact extracted URL as malicious.

The final disposition was **Suspicious / Unconfirmed**.

## Objective

The objective of this case was to investigate a suspicious email and determine whether the associated sender, URLs, domains, and infrastructure represented a confirmed phishing threat.

The investigation focused on:

- Email header consistency
- Embedded URL identification
- DNS resolution
- Reverse DNS analysis
- Hosting infrastructure
- Redirect behavior
- Domain reputation
- Exact URL reputation
- Final SOC assessment
- Escalation decision

## Environment / Data Source

Email Sample:

`email3.eml`

Environment:

TryHackMe phishing analysis environment

Tools Used:

- Mozilla Thunderbird
- dig
- nslookup
- URLScan.io
- VirusTotal

Email Date:

11 July 2021

Investigation Date:

August 2026

Note:

The email sample is historical. DNS, hosting, and reputation information collected during the current investigation represents current infrastructure and should not automatically be considered the same infrastructure used when the email was originally sent.

## Observed Activity

The email contained the following sender information:

- From: `support@teckbe.com`
- Reply-To: `support@teckbe.com`
- Recipient: `alexa@yahoo.com`
- Related Message-ID domain: `teckbe.com`

The raw HTML body contained multiple references to:

- `t.teckbe.com`
- `img.teckbe.com`

A tracking-style URL was identified:

`http://t.teckbe.com/p/?j3=EOowFcEwFHL6EOAyFcoUFVTVEchwFHLUFOo6MjL6EbTT`

The sender and Reply-To addresses were consistent, meaning no obvious sender-address mismatch was identified.

However, the embedded tracking and redirect infrastructure required further investigation.

## Evidence

### Email Header and URL Evidence

The raw email source showed consistent `teckbe.com` sender information together with tracking and image-delivery URLs.

![Email Header and URL Evidence](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/01-email-header-and-url-evidence.png)

The sender and Reply-To addresses both used the same domain.

The email also contained links associated with `t.teckbe.com` and images loaded from `img.teckbe.com`.

No obvious sender spoofing was identified from the available sender fields.

### Domain DNS Analysis

DNS analysis was performed against `teckbe.com` and `t.teckbe.com`.

The following IP addresses were identified:

- `83.167.244.201`
- `83.167.244.202`

The tracking subdomain was configured as:

`t.teckbe.com → CNAME → teckbe.com`

![Domain DNS Results](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/02-domain-dns-results.png)

Because the DNS analysis was performed in 2026 against an email from 2021, these addresses represent current infrastructure and do not prove that the same IP addresses were used during the original email campaign.

### Reverse DNS Analysis

Reverse DNS lookups were performed against the identified IP addresses.

The following PTR relationships were observed:

- `83.167.244.201` → `px01.svethostingu.cz`
- `83.167.244.202` → `px02.svethostingu.cz`

![Reverse DNS Results](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/03-reverse-dns-results.png)

The reverse DNS results provided additional hosting context but did not independently indicate malicious activity.

### URLScan Infrastructure Analysis

URLScan analysis identified `83.167.244.201` as the main IP associated with the currently analyzed domain.

Additional information included:

- ASN: `AS24971`
- Provider: MasterDC s.r.o.
- Location: Czech Republic
- 10 contacted IP addresses
- 10 contacted domains
- 130 HTTP transactions
- Multiple HTTP and HTTPS redirects

URLScan returned:

`No classification`

Google Safe Browsing also showed no classification at the time of analysis.

![URLScan Infrastructure Analysis](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/04-urlscan-infrastructure-analysis.png)

Redirect behavior was observed between HTTP, HTTPS, and `www.teckbe.com`.

However, this scan analyzed the current parent domain rather than reproducing the exact historical tracking-link behavior from the 2021 email.

### VirusTotal Domain Reputation

VirusTotal analysis was performed against:

`teckbe.com`

The result showed:

`0 / 91 security vendors flagged the domain as malicious`

![VirusTotal Domain Reputation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/05-virustotal-domain-reputation.png)

VirusTotal also showed a relationship with one detected file communicating with the domain.

This relationship was treated as contextual evidence only and does not prove that the domain itself is malicious.

The zero-detection result reduced the strength of the malicious hypothesis but was not treated as proof that the historical email was benign.

### Exact URL Reputation

The exact tracking URL extracted from the email was submitted to VirusTotal:

`http://t.teckbe.com/p/?j3=EOowFcEwFHL6EOAyFcoUFVTVEchwFHLUFOo6MjL6EbTT`

VirusTotal showed that no security vendors flagged the URL as malicious.

The analysis also showed:

- Network Requests: 0
- Domains: 0
- IP Addresses: 0
- Response Size: 0 B
- HTTP Status: Not returned
- Content Type: Not returned

![Exact URL Reputation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/case-004/06-exact-url-reputation.png)

The URL also returned:

`ERR_CONNECTION_RESET`

during a current connectivity test.

Because the historical URL could not be successfully retrieved, its original landing-page behavior could not be verified.

The absence of detections therefore cannot be treated as proof that the historical URL was benign.

## Analysis

The investigated email contained tracking-style URLs associated with the same domain family used by the sender.

The From and Reply-To addresses were consistent, reducing the likelihood of simple sender-address spoofing.

The email also contained dedicated tracking and image-delivery subdomains.

Tracking and redirect infrastructure can be used by legitimate marketing campaigns, but similar infrastructure can also be used during phishing attacks to obscure the final destination of a link.

Current DNS and reverse DNS analysis identified the infrastructure associated with the domain.

URLScan showed hosting information and redirect behavior, but no malicious classification was returned.

VirusTotal returned `0/91` detections for the parent domain.

The exact URL extracted from the email was also not flagged as malicious.

However, the exact URL could not be successfully retrieved or dynamically analyzed during the current investigation.

Therefore, the original behavior of the historical tracking URL could not be confirmed.

The available evidence is insufficient to confirm phishing, credential harvesting, malware delivery, or other malicious activity.

An alternative explanation is that the observed infrastructure belonged to a legitimate email marketing or tracking system.

## Risk

If the embedded URL had historically redirected users to malicious infrastructure, potential risks could have included:

- Credential theft
- Malicious redirection
- Malware delivery
- User tracking
- Additional social engineering

However, none of these impacts were confirmed during the investigation.

The inability to reproduce the historical landing-page behavior significantly limits the confidence of a malicious classification.

## Recommended Next Steps

In an operational SOC environment:

- Preserve the original email and extracted indicators
- Search the email gateway for additional messages using `teckbe.com`
- Search for additional messages containing `t.teckbe.com`
- Determine whether other recipients received the same campaign
- Search DNS and proxy logs for connections to the identified domains
- Review endpoint telemetry for activity following user interaction
- Determine whether credentials were submitted
- Determine whether files were downloaded
- Correlate the indicators with additional threat-intelligence sources
- Reassess the case if additional suspicious activity is identified

## Escalation Decision

Disposition:

**Suspicious / Unconfirmed**

Confidence:

**Medium**

Confirmed Malicious Activity:

**No**

Credential Harvesting Confirmed:

**No**

Malware Delivery Confirmed:

**No**

Immediate Escalation Required:

**No**

Based on the currently available evidence, immediate incident escalation is not justified.

The case should be reassessed if additional telemetry identifies successful user interaction, credential submission, malware execution, multiple affected recipients, or stronger threat-intelligence indicators.

## MITRE ATT&CK Mapping

Potential technique:

**T1566.002 - Phishing: Spearphishing Link**

This mapping represents the investigated phishing hypothesis and does not confirm that a successful phishing attack occurred.

## Final SOC Summary

A suspicious historical email containing tracking-style URLs associated with `teckbe.com` was investigated using email header analysis, DNS enrichment, reverse DNS analysis, URLScan, and VirusTotal.

The sender and Reply-To information were consistent, while current reputation sources did not classify the parent domain or exact extracted URL as malicious.

The historical tracking URL could not currently be retrieved, preventing verification of its original landing-page behavior.

Based on the available evidence, the activity was classified as **Suspicious / Unconfirmed**.

No immediate escalation was recommended, although the identified indicators should be correlated with email, DNS, proxy, and endpoint telemetry if available.

## Lessons Learned

This case demonstrated that suspicious indicators should not automatically be classified as malicious.

Key lessons included:

- Verify sender consistency before claiming spoofing
- Tracking URLs require investigation but are not inherently malicious
- VirusTotal results should not be treated as absolute verdicts
- A `0/91` detection result does not prove that an indicator is safe
- Current DNS infrastructure may differ from historical infrastructure
- Parent-domain analysis and exact-URL analysis provide different evidence
- Historical URLs may no longer reproduce their original behavior
- SOC conclusions should separate confirmed facts from assumptions
- Escalation decisions should be based on correlated evidence rather than a single suspicious indicator
