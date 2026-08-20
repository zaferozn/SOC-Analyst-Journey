
# Lab 027 - Wazuh + Suricata Integration

## Executive Summary

This lab implemented and validated an end-to-end **Suricata IDS → Wazuh SIEM integration pipeline**.

Suricata was already operating on the monitored Ubuntu endpoint and generating structured network telemetry in `/var/log/suricata/eve.json`. However, Wazuh was not initially configured to collect this data.

The Wazuh agent configuration was updated to ingest Suricata EVE JSON events. After validating the integration, a temporary controlled Suricata rule was created to detect ICMP echo requests originating from the Wazuh server (`192.168.64.9`) and targeting the monitored Ubuntu endpoint (`192.168.64.10`).

Five controlled ICMP echo requests were generated. Suricata detected all five requests under custom signature ID `1000027` and wrote the alerts to `eve.json`.

The Wazuh agent collected the events, forwarded them to the Wazuh manager, decoded the JSON telemetry, and generated Wazuh rule `86601` alerts under the `ids` and `suricata` groups.

The Wazuh Threat Hunting dashboard displayed exactly **five corresponding alerts**, confirming successful end-to-end telemetry flow.

After validation, the temporary Suricata test rule was removed, the original detection rule was preserved, the Suricata configuration was tested successfully, and both Suricata and the Wazuh agent remained active.

---

## Objective

The objective of this lab was to integrate Suricata network IDS telemetry with Wazuh SIEM and validate the complete alert pipeline from network activity to centralized analyst visibility.

The lab was designed to prove the following workflow:

```text
Network Traffic
      ↓
Suricata
      ↓
eve.json
      ↓
Wazuh Logcollector
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
JSON Decoder
      ↓
Wazuh Detection Rule
      ↓
Threat Hunting Dashboard
      ↓
SOC Analyst
```

The specific objectives were:

- Verify Suricata and the Wazuh agent were operational.
- Confirm Suricata was producing EVE JSON telemetry.
- Determine whether Wazuh was already ingesting `eve.json`.
- Configure Wazuh JSON log collection for Suricata.
- Validate Wazuh Logcollector monitoring.
- Create a deterministic Suricata test detection.
- Generate controlled network traffic.
- Verify the raw Suricata alert.
- Verify the same event reached the Wazuh manager.
- Inspect structured SIEM fields.
- Validate the alert through the Wazuh Threat Hunting dashboard.
- Remove the temporary test rule and return the environment to its normal state.

---

## Environment / Data Source

### Hosts

**Wazuh Server**

```text
Hostname: wazuh-server
IPv4: 192.168.64.9
Role: Wazuh Manager / SIEM
```

**Monitored Endpoint**

```text
Hostname: ubuntu-agent
IPv4: 192.168.64.10
Wazuh Agent ID: 001
Role: Monitored Linux endpoint / Suricata IDS sensor
```

### Tools

- Wazuh SIEM
- Wazuh Agent
- Wazuh Logcollector
- Wazuh Threat Hunting
- Suricata 7.0.3
- Suricata EVE JSON
- Linux CLI
- `grep`
- `systemctl`
- `python3`
- `ping`
- `sed`
- `suricata -T`

### Primary Log Sources

```text
/var/log/suricata/eve.json
/var/ossec/logs/ossec.log
/var/ossec/logs/alerts/alerts.json
```

### Suricata Rule Files

```text
Default rule path:
/var/lib/suricata/rules

Primary rules:
/var/lib/suricata/rules/suricata.rules

Local rules:
/var/lib/suricata/rules/local.rules
```

---

# Architecture

## Before Integration

```text
Network Traffic
      ↓
Suricata
      ↓
eve.json
      │
      X
      │
Wazuh
```

Suricata was generating network telemetry, but Wazuh was not configured to ingest `/var/log/suricata/eve.json`.

## After Integration

```text
Network Traffic
      ↓
Suricata
      ↓
eve.json
      ↓
Wazuh Logcollector
      ↓
Wazuh Agent 001
      ↓
Wazuh Manager
      ↓
JSON Decoder
      ↓
Wazuh Rule 86601
      ↓
Threat Hunting
      ↓
SOC Analyst
```

---

# Investigation and Implementation

## Phase 1 - Baseline Validation

The first step was to confirm the current state of Suricata, the Wazuh agent, and the Suricata log files.

Commands included:

```bash
hostname
hostname -I

suricata --build-info | head -10

sudo systemctl is-active suricata
sudo systemctl status suricata --no-pager

sudo systemctl is-active wazuh-agent
sudo systemctl status wazuh-agent --no-pager

sudo ls -lh /var/log/suricata/
sudo ls -lh /var/log/suricata/eve.json
```

Results confirmed:

```text
Host: ubuntu-agent
IP: 192.168.64.10

Suricata: active
Version: 7.0.3

Wazuh Agent: active

EVE JSON:
 /var/log/suricata/eve.json

EVE JSON size:
 approximately 78 MB
```

Suricata was therefore actively producing network telemetry before the Wazuh integration was configured.

### Evidence - Screenshot 01

![Suricata and Wazuh baseline](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/01-wazuh-suricata-baseline.png)

---

## Phase 2 - Pre-Integration Wazuh Configuration Check

The Wazuh agent configuration was inspected for an existing Suricata EVE JSON collection entry.

```bash
sudo grep -n -B3 -A5 \
'/var/log/suricata/eve.json' \
/var/ossec/etc/ossec.conf

sudo grep -n -B2 -A4 \
'<log_format>json</log_format>' \
/var/ossec/etc/ossec.conf
```

Neither search returned an existing configuration.

This demonstrated that Suricata and Wazuh were both installed and operational but were **not yet integrated**.

### Evidence - Screenshot 02

![Wazuh pre-integration configuration](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/02-wazuh-preintegration-config.png)

---

## Phase 3 - Configure Suricata EVE JSON Ingestion

Before modifying the Wazuh configuration, a backup was created.

```bash
sudo cp /var/ossec/etc/ossec.conf \
/var/ossec/etc/ossec.conf.lab027-backup
```

The following configuration was added to `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

The configuration instructed Wazuh Logcollector to monitor Suricata EVE JSON events as structured JSON telemetry.

The resulting configuration was verified with:

```bash
sudo grep -n -B2 -A5 \
'/var/log/suricata/eve.json' \
/var/ossec/etc/ossec.conf
```

### Evidence - Screenshot 03

![Suricata Wazuh log ingestion configuration](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/03-suricata-wazuh-log-ingestion-config.png)

---

## Phase 4 - Activate and Validate Wazuh Ingestion

The Wazuh agent was restarted to load the new configuration.

```bash
sudo systemctl restart wazuh-agent

sudo systemctl is-active wazuh-agent
sudo systemctl status wazuh-agent --no-pager
```

The service returned:

```text
active
```

The Wazuh agent log was then inspected.

A decisive entry appeared:

```text
wazuh-logcollector: INFO: (1950): Analyzing file: '/var/log/suricata/eve.json'.
```

This confirmed that the Wazuh Logcollector had successfully loaded the Suricata telemetry source.

### Evidence - Screenshot 04

![Wazuh agent post-integration status](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/04-wazuh-agent-post-integration-status.png)

---

## Phase 5 - Initial Controlled Network Activity

Twenty ICMP echo requests were initially generated from the Wazuh server toward the Ubuntu endpoint.

```bash
ping -c 20 192.168.64.10
```

The source was:

```text
wazuh-server
192.168.64.9
```

The destination was:

```text
ubuntu-agent
192.168.64.10
```

Results:

```text
20 packets transmitted
20 packets received
0% packet loss
```

### Evidence - Screenshot 05

![Initial controlled network activity](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/05-controlled-network-activity.png)

### Analysis

The traffic itself did not generate the required fresh Suricata IDS alert.

This was expected because merely observing ICMP traffic does not guarantee that an enabled Suricata detection rule will classify it as an alert.

Instead of relying on unpredictable existing signatures, a deterministic test rule was created.

---

# Controlled Detection Validation

## Phase 6 - Inspect Suricata Rule Configuration

The Suricata configuration showed:

```text
default-rule-path: /var/lib/suricata/rules

rule-files:
  - suricata.rules
  - local.rules
```

The active local rule file was therefore:

```text
/var/lib/suricata/rules/local.rules
```

An existing custom rule was already present:

```text
LOCAL TCP SYN Port Scan Detected
SID: 1000001
```

Before adding the Lab 027 rule, the existing rules were inspected and backed up.

```bash
sudo cp \
/var/lib/suricata/rules/local.rules \
/var/lib/suricata/rules/local.rules.lab027-backup
```

The intended Lab 027 SID was also checked:

```text
SID 1000027 is available
```

### Evidence - Screenshot 06

![Local Suricata rules backup](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/06-local-suricata-rules-backup.png)

---

## Phase 7 - Create Deterministic Suricata Detection

A temporary detection rule was created specifically for this integration test.

```text
alert icmp 192.168.64.9 any -> 192.168.64.10 any (msg:"LAB027 Controlled ICMP Test"; itype:8; classtype:network-scan; sid:1000027; rev:1;)
```

The rule conditions were:

```text
Protocol:
ICMP

Source:
192.168.64.9

Destination:
192.168.64.10

ICMP Type:
8 - Echo Request

Signature:
LAB027 Controlled ICMP Test

SID:
1000027
```

The configuration was validated before activation:

```bash
sudo suricata -T \
-c /etc/suricata/suricata.yaml
```

Result:

```text
Configuration provided was successfully loaded. Exiting.
```

### Evidence - Screenshot 07

![Lab 027 Suricata ICMP rule validation](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/07-lab027-suricata-icmp-rule-validation.png)

---

## Phase 8 - Activate the Detection Rule

Suricata was restarted.

```bash
sudo systemctl restart suricata
```

The service returned:

```text
active
```

Suricata successfully loaded the ruleset:

```text
2 rule files processed
52237 rules successfully loaded
0 rules failed
```

This confirmed that the temporary Lab 027 rule had been successfully activated.

### Evidence - Screenshot 08

![Suricata post-rule restart](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/08-suricata-post-rule-restart.png)

---

# Controlled IDS Trigger

## Phase 9 - Generate Detection Traffic

Five ICMP echo requests were generated from the Wazuh server.

```bash
ping -c 5 192.168.64.10
```

Source:

```text
Hostname: wazuh-server
IP: 192.168.64.9
```

Destination:

```text
Hostname: ubuntu-agent
IP: 192.168.64.10
```

Result:

```text
5 packets transmitted
5 packets received
0% packet loss
```

This traffic exactly matched the temporary Suricata rule.

### Evidence - Screenshot 09

![Lab 027 controlled ICMP trigger](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/09-lab027-controlled-icmp-trigger.png)

---

# Raw Suricata Alert Verification

The Suricata EVE JSON log was searched for the controlled signature.

The resulting event contained:

```text
timestamp    : 2026-08-19T16:37:43.857147+0000
event_type   : alert
src_ip       : 192.168.64.9
dest_ip      : 192.168.64.10
proto        : ICMP
icmp_type    : 8
signature_id : 1000027
signature    : LAB027 Controlled ICMP Test
category     : Detection of a Network Scan
severity     : 3
action       : allowed
```

The network detection layer was therefore confirmed:

```text
192.168.64.9
      ↓
ICMP Echo Request
      ↓
192.168.64.10
      ↓
Suricata Rule Match
      ↓
SID 1000027
      ↓
eve.json
```

### Evidence - Screenshot 10

![Suricata EVE JSON controlled alert](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/10-suricata-eve-json-controlled-alert.png)

---

# Wazuh SIEM Verification

## Wazuh Alert Search

The Wazuh manager alert store was searched for both the Suricata signature and SID.

```bash
sudo grep -F 'LAB027 Controlled ICMP Test' \
/var/ossec/logs/alerts/alerts.json

sudo grep -F '"signature_id":"1000027"' \
/var/ossec/logs/alerts/alerts.json
```

The controlled Suricata alerts were found inside:

```text
/var/ossec/logs/alerts/alerts.json
```

Wazuh generated:

```text
Rule ID:
86601

Rule level:
3

Description:
Suricata: Alert - LAB027 Controlled ICMP Test

Rule groups:
ids
suricata

Decoder:
json
```

The structured event contained:

```text
Agent ID:
001

Agent:
ubuntu-agent

Agent IP:
192.168.64.10

Manager:
wazuh-server

Event type:
alert

Source IP:
192.168.64.9

Destination IP:
192.168.64.10

Protocol:
ICMP

ICMP Type:
8

Suricata SID:
1000027

Signature:
LAB027 Controlled ICMP Test

Category:
Detection of a Network Scan

Severity:
3

Action:
allowed

Log source:
/var/log/suricata/eve.json
```

The final event showed:

```text
Fired times: 5
```

This correlated directly with the five ICMP echo requests generated during the controlled test.

### Evidence - Screenshot 11

![Wazuh Suricata SIEM alert](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/11-wazuh-suricata-siem-alert.png)

---

# Threat Hunting Dashboard Verification

The Wazuh Threat Hunting interface was filtered using:

```text
rule.groups:suricata
```

The dashboard displayed:

```text
5 hits
```

All five events contained:

```text
Agent:
ubuntu-agent

Description:
Suricata: Alert - LAB027 Controlled ICMP Test

Rule level:
3

Rule ID:
86601
```

This provided visual confirmation that the alerts had successfully reached the centralized SOC interface.

### Evidence - Screenshot 12

![Wazuh dashboard Suricata alert](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/12-wazuh-dashboard-suricata-alert.png)

---

# Structured Alert Field Investigation

One of the Wazuh events was expanded in the Threat Hunting interface.

The document details showed:

```text
data.alert.signature:
LAB027 Controlled ICMP Test

data.alert.signature_id:
1000027

data.dest_ip:
192.168.64.10

data.event_type:
alert

data.flow.src_ip:
192.168.64.9

data.icmp_type:
8

data.proto:
ICMP

data.src_ip:
192.168.64.9

decoder.name:
json
```

This confirmed that the original Suricata EVE fields remained available to the SOC analyst after ingestion and processing by Wazuh.

### Evidence - Screenshot 13

![Wazuh dashboard Suricata alert details](https://raw.githubusercontent.com/zaferozn/SOC-Analyst-Journey/main/screenshots/lab-027/13-wazuh-dashboard-suricata-alert-details.png)

---

# End-to-End Correlation

The final validated telemetry path was:

```text
wazuh-server
192.168.64.9
      │
      │ 5 ICMP Echo Requests
      ↓
ubuntu-agent
192.168.64.10
      │
      ↓
Suricata 7.0.3
      │
      ↓
Custom SID 1000027
      │
      ↓
eve.json
      │
      ↓
Wazuh Logcollector
      │
      ↓
Wazuh Agent 001
      │
      ↓
Wazuh Manager
      │
      ↓
JSON Decoder
      │
      ↓
Wazuh Rule 86601
      │
      ↓
5 Wazuh Alerts
      │
      ↓
Threat Hunting Dashboard
      │
      ↓
Structured Field Investigation
      │
      ↓
SOC Analyst
```

---

# What Was Observed?

Five controlled ICMP echo requests were sent from the Wazuh server to the monitored Ubuntu endpoint.

Suricata matched each request against temporary custom signature `1000027` and generated structured EVE JSON alert events.

Wazuh successfully ingested those events and generated corresponding SIEM alerts under Wazuh rule `86601`.

Exactly five alerts appeared in the Wazuh Threat Hunting dashboard.

---

# Where Was It Observed?

The activity was observed at multiple layers.

### Network Detection Layer

```text
/var/log/suricata/eve.json
```

### Wazuh Agent Collection Layer

```text
/var/ossec/logs/ossec.log
```

Evidence:

```text
wazuh-logcollector:
Analyzing file: '/var/log/suricata/eve.json'
```

### Wazuh Manager Alert Layer

```text
/var/ossec/logs/alerts/alerts.json
```

### Analyst Layer

```text
Wazuh Threat Hunting Dashboard
```

---

# Which Hosts and IP Addresses Were Involved?

### Source

```text
Host: wazuh-server
IP: 192.168.64.9
```

### Destination

```text
Host: ubuntu-agent
IP: 192.168.64.10
Agent ID: 001
```

### Log Source

```text
/var/log/suricata/eve.json
```

---

# Why Was the Activity Suspicious?

The activity was not genuinely malicious.

It was deliberately generated as a **controlled IDS validation event**.

A temporary Suricata rule was configured to classify ICMP echo requests from the test source as an alert so that the complete Suricata-to-Wazuh telemetry pipeline could be validated deterministically.

Therefore:

```text
Detection status:
True Positive for the test rule

Security disposition:
Benign / Controlled Test

Malicious activity:
No
```

The `network-scan` classification was used for controlled laboratory validation and should not be interpreted as evidence that five ICMP requests alone represent a real network scan.

---

# What Evidence Supports the Finding?

Evidence included:

1. Suricata service active on `ubuntu-agent`.
2. Wazuh Agent active on `ubuntu-agent`.
3. Existing `/var/log/suricata/eve.json` telemetry.
4. No initial Suricata EVE JSON entry in Wazuh configuration.
5. Successful addition of JSON log collection.
6. Wazuh Logcollector confirming active monitoring of `eve.json`.
7. Custom Suricata SID `1000027`.
8. Successful Suricata configuration validation.
9. `52237` Suricata rules successfully loaded with `0` failures.
10. Five controlled ICMP requests from `192.168.64.9`.
11. Five Suricata alerts containing SID `1000027`.
12. Wazuh rule `86601` processing the Suricata events.
13. JSON decoder successfully preserving structured fields.
14. Five corresponding Wazuh Threat Hunting events.
15. Expanded Wazuh event showing the original Suricata fields.

---

# Risk

The controlled test traffic represented no genuine security threat.

However, the lab demonstrates an important operational security principle.

Without IDS-to-SIEM integration, Suricata detections would remain isolated in a local endpoint log:

```text
Suricata
   ↓
eve.json
```

An analyst would need to inspect the endpoint manually.

After integration:

```text
Suricata
   ↓
eve.json
   ↓
Wazuh
   ↓
Centralized Detection
   ↓
SOC Analyst
```

This improves centralized visibility and makes network IDS telemetry available for triage, correlation, investigation, and escalation.

---

# Recommended Next Steps

In a production environment, the following actions would be appropriate:

1. Continue collecting Suricata EVE JSON telemetry centrally.
2. Monitor `rule.groups:suricata` for IDS detections.
3. Prioritize alerts based on signature severity and context.
4. Correlate Suricata network events with endpoint and authentication activity.
5. Investigate repeated source IPs across different telemetry sources.
6. Tune noisy or informational Suricata signatures.
7. Avoid treating every IDS alert as automatically malicious.
8. Validate source, destination, protocol, signature, severity, and surrounding activity.
9. Maintain change-control procedures when adding custom detection rules.
10. Remove temporary test rules after validation.

---

# Escalation Decision

```text
Escalation Required: No
Disposition: Benign / Controlled Test
```

The events were deliberately generated as part of the lab.

In a real SOC environment, escalation would depend on:

- source reputation,
- destination asset criticality,
- signature confidence,
- alert severity,
- repeated activity,
- endpoint telemetry,
- authentication evidence,
- additional network indicators,
- and whether the behavior could be explained by authorized activity.

---

# False Positive / Contextual Analysis

An additional observation occurred during Wazuh alert searches.

A broad search for:

```text
LAB027 Controlled ICMP Test
```

also returned a Wazuh sudo event.

The reason was that the analyst had previously executed:

```bash
sudo grep -F '"signature":"LAB027 Controlled ICMP Test"' \
/var/log/suricata/eve.json
```

Wazuh recorded the sudo command itself, including the search string.

Therefore, a simple keyword search matched both:

```text
Actual Suricata alert
```

and:

```text
Unrelated sudo activity containing the same text
```

This demonstrates an important SOC principle:

> A keyword match is not equivalent to event correlation.

The analyst must validate:

- rule ID,
- decoder,
- log source,
- event type,
- source IP,
- destination IP,
- signature ID,
- timestamp,
- and surrounding context.

---

# Cleanup and Change Control

After successful validation, the temporary Lab 027 Suricata rule was removed.

```bash
sudo sed -i '/sid:1000027/d' \
/var/lib/suricata/rules/local.rules
```

Verification returned:

```text
LAB027 temporary rule successfully removed
```

The original local detection remained:

```text
LOCAL TCP SYN Port Scan Detected
SID: 1000001
```

Suricata configuration was then tested again:

```bash
sudo suricata -T \
-c /etc/suricata/suricata.yaml
```

Result:

```text
Configuration provided was successfully loaded. Exiting.
```

Both security services remained operational:

```text
Suricata:
active

Wazuh Agent:
active
```

This ensured the environment was returned to a clean operational state after the test.

---

# MITRE ATT&CK Mapping

No specific MITRE ATT&CK technique was assigned to the controlled ICMP test.

The activity was designed to validate telemetry ingestion and SIEM integration rather than emulate a specific adversary technique.

Forcing an ATT&CK mapping would therefore provide little analytical value.

---

# Final SOC Summary

Suricata network IDS telemetry was successfully integrated with the Wazuh SIEM by configuring the Wazuh agent to ingest `/var/log/suricata/eve.json` using JSON log collection.

A temporary Suricata detection rule was created to provide deterministic validation of the integration. Five ICMP echo requests were generated from `wazuh-server` (`192.168.64.9`) toward `ubuntu-agent` (`192.168.64.10`). Suricata detected each request under signature ID `1000027`, generated structured EVE JSON alerts, and recorded the source, destination, protocol, signature, category, and severity.

The Wazuh Logcollector successfully monitored the Suricata EVE JSON file and forwarded the events to the Wazuh manager. The events were decoded as JSON and processed by Wazuh rule `86601` under the `ids` and `suricata` groups.

Five corresponding alerts were confirmed in both `/var/ossec/logs/alerts/alerts.json` and the Wazuh Threat Hunting dashboard. Individual event inspection confirmed that the original Suricata fields remained available for analyst investigation.

The temporary detection rule was removed after validation, the original local Suricata rule was preserved, and both Suricata and the Wazuh agent remained active.

The test therefore confirmed a complete and operational **network IDS → SIEM → SOC analyst telemetry pipeline**.

---

# Lessons Learned

## 1. Installing two security tools does not mean they are integrated

Suricata and Wazuh were both operational before this lab.

However, Wazuh had no configuration instructing it to collect Suricata telemetry.

Integration required an explicit telemetry path:

```text
Suricata
↓
eve.json
↓
Wazuh Logcollector
```

---

## 2. Raw telemetry and SIEM alerts are different layers

Suricata produced:

```text
event_type: alert
signature_id: 1000027
```

Wazuh then transformed this telemetry into:

```text
rule.id: 86601
rule.groups: ids, suricata
decoder.name: json
```

Understanding both layers is important during SOC investigations.

---

## 3. Deterministic testing is better than hoping an alert fires

The first ICMP test did not produce the desired IDS alert.

Instead of repeatedly generating random traffic, a controlled rule was created with a known:

```text
source
destination
protocol
signature
SID
```

This produced predictable evidence and allowed the integration to be validated properly.

---

## 4. A successful service restart is not proof of full integration

After modifying `ossec.conf`, the Wazuh agent restarted successfully.

That only proved that the configuration was accepted.

The integration was not fully proven until the same event could be traced through:

```text
eve.json
→ Wazuh Manager
→ Wazuh rule
→ Threat Hunting
```

---

## 5. Structured fields improve analyst investigation

The Wazuh dashboard preserved fields including:

```text
data.src_ip
data.dest_ip
data.proto
data.icmp_type
data.alert.signature
data.alert.signature_id
```

This is significantly more useful than treating the event as an unstructured text string.

---

## 6. IDS detection does not necessarily mean blocking

The Suricata alert showed:

```text
action: allowed
```

This did not indicate a detection failure.

Suricata was operating as an IDS sensor and successfully detected and logged the traffic without blocking it.

---

## 7. Search results must be validated contextually

A keyword search also matched an unrelated sudo event because the search term itself appeared inside a logged command.

This reinforced the need to validate structured fields rather than relying only on text matching.

---

## 8. Temporary detection logic should be removed after testing

After validation, SID `1000027` was removed.

This prevented unnecessary alert generation and restored the lab environment to its intended state.

---

# Skills Demonstrated

- Suricata IDS administration
- Suricata EVE JSON analysis
- Wazuh SIEM administration
- Wazuh Agent configuration
- Wazuh Logcollector configuration
- IDS-to-SIEM integration
- JSON log ingestion
- Structured alert analysis
- Custom IDS rule creation
- Detection engineering fundamentals
- Rule syntax validation
- Controlled alert generation
- Network telemetry analysis
- Source/destination correlation
- SIEM alert verification
- Threat Hunting
- Alert field extraction
- False-match analysis
- Change-control discipline
- Configuration backup
- Rollback and cleanup
- Evidence-based SOC reporting
- Converged network and endpoint monitoring

---

# SOC English Sentences

### Detection

> Suricata detected controlled ICMP traffic originating from `192.168.64.9` and targeting the monitored endpoint at `192.168.64.10`.

### Telemetry

> The detection was recorded as a structured EVE JSON event under Suricata signature ID `1000027`.

### SIEM Integration

> The Wazuh agent was configured to ingest Suricata EVE JSON telemetry and forward the resulting events to the Wazuh manager.

### Alert Processing

> Wazuh decoded the Suricata event as JSON and generated rule `86601` under the `ids` and `suricata` rule groups.

### Correlation

> Five controlled ICMP requests resulted in five Suricata detections and five corresponding Wazuh alerts.

### Investigation

> The analyst validated the source IP, destination IP, protocol, signature ID, decoder, rule ID, and original log source before determining the final disposition.

### Disposition

> The activity was classified as benign because it was generated intentionally as part of a controlled IDS-to-SIEM integration test.

### Cleanup

> Following successful validation, the temporary detection rule was removed and the Suricata configuration was revalidated before returning the environment to normal operation.

---

# CV Bullet

**Integrated Suricata IDS telemetry with Wazuh SIEM by configuring EVE JSON ingestion, creating and validating a controlled detection rule, tracing network alerts through Wazuh's JSON decoder and rule engine, and verifying structured events in the Threat Hunting dashboard.**

Alternative shorter version:

**Built and validated an end-to-end Suricata-to-Wazuh IDS/SIEM pipeline, correlating controlled network detections from EVE JSON through centralized Wazuh alerting and Threat Hunting.**

---

# Interview Explanation

## Question

**Have you integrated an IDS with a SIEM?**

## Answer

Yes. In my home SOC environment, I integrated Suricata with Wazuh.

Suricata was already running on my monitored Ubuntu endpoint and producing structured telemetry in `eve.json`, but Wazuh was not collecting that data.

I first verified both services and confirmed the missing integration. I then configured the Wazuh agent to collect `/var/log/suricata/eve.json` using JSON log ingestion and verified through the Wazuh agent logs that Logcollector was actively monitoring the file.

To validate the integration deterministically, I created a temporary Suricata rule that detected ICMP echo requests from my Wazuh server to the monitored endpoint. I validated the Suricata configuration before restarting the service and then generated five controlled ICMP requests.

Suricata generated five alerts under signature ID `1000027`. I verified the raw events in EVE JSON and then traced the same events into the Wazuh manager.

Wazuh decoded the events as JSON and generated rule `86601` alerts under the `ids` and `suricata` groups. The Wazuh Threat Hunting dashboard displayed exactly five alerts, corresponding to the five test packets.

I also inspected the structured fields including source IP, destination IP, protocol, Suricata signature, Wazuh rule ID, decoder, and log source.

Finally, I removed the temporary rule, validated the Suricata configuration again, and confirmed that both Suricata and the Wazuh agent remained operational.

The lab gave me practical experience with IDS-to-SIEM integration, structured telemetry, detection validation, centralized monitoring, and analyst-level alert investigation.

---

# Short Interview Version

> I integrated Suricata with Wazuh by configuring the Wazuh agent to ingest EVE JSON telemetry. I created a temporary controlled ICMP detection, generated five test events, confirmed the raw Suricata alerts, traced them through the Wazuh JSON decoder and rule `86601`, and verified five corresponding alerts in Threat Hunting. I then removed the temporary rule and revalidated the environment.

---

# Portfolio Value

This lab demonstrates more than independent familiarity with Wazuh or Suricata.

It demonstrates the ability to connect security technologies into a working monitoring architecture:

```text
Network Detection
+
Structured Logging
+
SIEM Ingestion
+
Alert Processing
+
Centralized Monitoring
+
Analyst Investigation
```

The primary portfolio value is therefore not:

```text
"I installed Suricata."
```

or:

```text
"I used Wazuh."
```

It is:

```text
"I built, tested, investigated, and validated an end-to-end IDS-to-SIEM telemetry pipeline."
```

---

# Final Lab Status

```text
LAB 027 - WAZUH + SURICATA INTEGRATION

Suricata operational                     ✅
Wazuh Agent operational                  ✅
EVE JSON telemetry verified              ✅
Pre-integration gap identified            ✅
Wazuh JSON ingestion configured           ✅
Logcollector monitoring verified          ✅
Existing rules protected                  ✅
Temporary detection rule created          ✅
Suricata configuration validated          ✅
Controlled traffic generated              ✅
Suricata SID 1000027 triggered            ✅
Raw EVE JSON alert verified               ✅
Wazuh Rule 86601 verified                 ✅
JSON decoder verified                     ✅
Five Wazuh alerts correlated              ✅
Threat Hunting visualization verified     ✅
Structured field investigation completed  ✅
Temporary rule removed                    ✅
Original rule preserved                   ✅
Final configuration validated             ✅


