# 07 - SOC Metrics and Objectives

## Executive Summary

This note summarizes the key SOC performance metrics covered in the TryHackMe **SOC Metrics and Objectives** room. The focus is on alert volume, false positive rate, alert escalation rate, threat detection rate, SLA, MTTD, MTTA, and MTTR.

The main lesson is that SOC work is not only about closing alerts. A SOC analyst must understand how alert handling speed, investigation quality, documentation, escalation, and false positive management affect the overall efficiency of the SOC team.

From an L1 SOC analyst perspective, these metrics are important because they measure how quickly and accurately alerts are acknowledged, triaged, escalated, and resolved.

---

## Objective

The objective of this room was to understand how SOC team performance is measured and how L1 analysts contribute to improving operational metrics.

---

## Room Information

Platform: TryHackMe  
Room: SOC Metrics and Objectives  
Focus area: SOC Operations / Alert Management / SOC Performance Metrics  
Status: Completed - Concepts reviewed and documented in analyst-style notes.

---

## Key Concepts Learned

- SOC teams protect the confidentiality, integrity, and availability of an organization’s digital assets.
- SOC analysts receive, triage, document, and escalate security alerts.
- L1 analysts are responsible for identifying true positives and escalating them to L2 analysts.
- Alert metrics help measure SOC workload, detection quality, escalation quality, and analyst performance.
- Time-based metrics such as MTTD, MTTA, and MTTR are often included in SLA requirements.
- False positives must be reduced because excessive alert noise can create analyst fatigue.
- Good documentation and fast escalation can reduce MTTR.
- L1 analysts directly affect MTTA, escalation quality, documentation quality, and false positive handling.

---

## Alert Quality and Workload Metrics

| Metric | Formula | Measures |
|---|---|---|
| Alerts Count | Total count of alerts received | Overall SOC analyst workload |
| False Positive Rate | False Positives / Total Alerts | Level of noise in alerts |
| Alert Escalation Rate | Escalated Alerts / Total Alerts | L1 analyst experience and escalation behavior |
| Threat Detection Rate | Detected Threats / Total Threats | SOC detection reliability |

---

## Alerts Count

Alerts Count measures the number of alerts received by the SOC team.

A high number of unresolved alerts may overwhelm L1 analysts and increase the risk of missing real threats. However, an unusually low alert count can also be suspicious because it may indicate SIEM visibility problems, broken detection rules, or missing log sources.

A balanced alert volume depends on company size, environment, and SOC maturity. In general, the room explains that approximately 5 to 30 alerts per day per L1 analyst may be a reasonable workload.

### Analyst Takeaway

Too many alerts can create overload.  
Too few alerts can indicate poor visibility.  
Alert volume must be evaluated together with detection quality and false positive rate.

---

## False Positive Rate

False Positive Rate measures how many alerts are later confirmed as benign activity.

A high false positive rate is dangerous because it creates alert fatigue. When analysts see too many noisy or benign alerts, they may become less vigilant and may miss real threats.

A false positive rate of 0% is unrealistic, but a rate of 80% or higher is a serious issue. This usually requires detection rule tuning, SIEM tuning, EDR tuning, or false positive remediation.

### Analyst Takeaway

Repeated benign alerts should be documented and reviewed for detection rule tuning.

High false positive rates reduce analyst attention and increase the chance of missing real attacks.

---

## Alert Escalation Rate

Alert Escalation Rate measures how often L1 analysts escalate alerts to L2 analysts.

L2 analysts rely on L1 analysts to filter noise and escalate actionable or suspicious alerts. However, an L1 analyst should not be overconfident and close alerts that are not fully understood.

A high escalation rate may show that L1 analysts are still inexperienced or lack confidence. A very low escalation rate may also be risky if real threats are being closed too early.

The room explains that alert escalation rate is usually aimed to be below 50%, and ideally below 20%, depending on the SOC environment.

### Analyst Takeaway

Escalation should be based on evidence, uncertainty, severity, and potential impact.

If the alert cannot be confidently closed as benign, it should be reviewed or escalated according to procedure.

---

## Threat Detection Rate

Threat Detection Rate measures how reliably the SOC detects real threats.

The formula is:

`Threat Detection Rate = Detected Threats / Total Threats`

This metric is critical because missed threats can lead to serious consequences such as ransomware infection, data exfiltration, unauthorized access, or account compromise.

The target should be as close to 100% as possible because every missed threat can create significant business and security impact.

### Analyst Takeaway

Threat Detection Rate is one of the most important SOC metrics because it reflects whether the SOC can actually detect real attacks.

A missed threat is more dangerous than a noisy alert.

---

## SLA and Time-Based SOC Metrics

An alert by itself does not stop an attack. The SOC must detect the activity, acknowledge the alert, triage the evidence, escalate when needed, and respond before the attacker achieves their objective.

A Service Level Agreement, or SLA, defines expected time requirements for SOC operations. These requirements may include SOC availability, detection speed, acknowledgement time, and response time.

---

## SOC Timeline Logic

Malicious event occurs  
↓  
SOC receives SIEM or EDR alert  
↓  
L1 analyst starts alert triage  
↓  
Escalation and internal response workflow begins  
↓  
SOC mitigates the breach  

---

## Time-Based Metrics

| Metric | Common SLA | Description |
|---|---:|---|
| SOC Team Availability | 24/7 | Working schedule of the SOC team |
| Mean Time to Detect | 5 minutes | Average time between attack activity and detection by SOC tools |
| Mean Time to Acknowledge | 10 minutes | Average time for L1 analysts to start triage of a new alert |
| Mean Time to Respond | 60 minutes | Average time taken by the SOC to stop the breach from spreading |

---

## MTTD - Mean Time to Detect

MTTD measures the average time between the start of suspicious or malicious activity and detection by SOC tools.

A high MTTD may indicate delayed detection, poor log visibility, slow detection rules, delayed SIEM ingestion, or missing telemetry.

### L1 Analyst Relevance

MTTD is mostly affected by SIEM rules, EDR telemetry, log coverage, and detection engineering. However, an L1 analyst can still support MTTD improvement by identifying missed patterns and reporting detection gaps.

---

## MTTA - Mean Time to Acknowledge

MTTA measures how long it takes for an L1 analyst to acknowledge an alert and begin triage.

This is one of the metrics most directly affected by L1 analysts.

### L1 Analyst Relevance

An L1 analyst improves MTTA by:

- Monitoring the alert queue consistently
- Starting triage quickly
- Prioritizing alerts based on severity
- Following shift procedures
- Avoiding unnecessary delay

---

## MTTR - Mean Time to Respond / Resolve

MTTR measures how long it takes the SOC to respond to or resolve an incident after detection.

A high MTTR may indicate weak escalation, unclear playbooks, poor documentation, slow containment, or poor coordination between L1, L2, incident response, and IT teams.

### L1 Analyst Relevance

An L1 analyst can help reduce MTTR by:

- Collecting relevant evidence quickly
- Documenting findings clearly
- Escalating suspicious activity without delay
- Following playbooks
- Avoiding vague handoff notes
- Including affected users, hosts, IP addresses, domains, files, timestamps, and alert fields

---

## How L1 Analysts Can Improve SOC Metrics

Although metrics tracking is usually a SOC manager’s responsibility, L1 analysts are often the first people to notice excessive alert volume, repeated false positives, delayed triage, or unclear escalation procedures.

A technical analyst should be able to highlight these issues and communicate them clearly.

---

## Improvement Recommendations

| Issue | Problem | Recommended Action |
|---|---|---|
| False Positive Rate over 80% | Too much alert noise | Exclude trusted activity from detection rules, tune SIEM/EDR rules, automate common triage steps |
| MTTD over 30 minutes | Threats are detected too late | Review detection rule performance, check real-time log ingestion, contact SOC engineers |
| MTTA over 30 minutes | L1 analysts start triage too late | Improve real-time notifications, distribute alerts evenly, monitor the queue consistently |
| MTTR over 4 hours | SOC cannot stop the breach quickly enough | Escalate threats quickly, use documented playbooks, improve evidence collection and handoff notes |

---

## L1 Analyst Practical Actions

- Monitor the alert queue during the shift.
- Acknowledge alerts within the expected SLA window.
- Prioritize alerts based on severity, asset criticality, and evidence.
- Document why an alert is a true positive or false positive.
- Escalate suspicious activity when evidence supports concern.
- Avoid closing alerts that are not fully understood.
- Report repeated false positives for tuning.
- Use playbooks for common attack scenarios.
- Collect clear evidence before escalation.
- Write handoff notes that L2 analysts can understand quickly.

---

## SOC Analyst Notes

### What was observed?

The room explained common SOC performance indicators, including Alerts Count, False Positive Rate, Alert Escalation Rate, Threat Detection Rate, SLA, MTTD, MTTA, and MTTR.

### Where was it observed?

The concepts were observed in the TryHackMe SOC Metrics and Objectives room.

### Which process was involved?

SOC alert handling, incident triage, escalation workflow, false positive management, detection quality, SLA compliance, and performance measurement were involved.

### Why is it important?

These metrics help evaluate whether a SOC team can detect, acknowledge, investigate, escalate, and respond to security events efficiently.

### What evidence supports it?

The room covered alert volume, false positive rate, alert escalation rate, threat detection rate, SLA requirements, MTTD, MTTA, MTTR, and practical recommendations for improving SOC metrics.

### What is the possible risk?

If these metrics are ignored, real incidents may be delayed, false positives may increase, alert fatigue may develop, escalation quality may decrease, and serious attacks may remain undetected.

### What should be done next?

An L1 analyst should acknowledge alerts quickly, review evidence carefully, document findings clearly, escalate suspicious activity according to procedure, and report repeated false positives or detection gaps for tuning.

### Should it be escalated?

This room is not an incident case. However, repeated SLA breaches, unresolved alert queues, excessive false positives, delayed response, or missed threats should be reviewed by SOC leads or SOC engineers.

---

## SOC English Sentences

- The alert was acknowledged within the expected SLA window.
- The alert queue should be monitored continuously during the shift.
- The incident should be reviewed to determine whether the response time met the required SLA.
- A high false positive rate may increase alert fatigue and reduce analyst efficiency.
- Repeated benign alerts should be reviewed for possible detection rule tuning.
- The alert requires further triage before a final verdict can be assigned.
- The finding should be escalated if additional evidence confirms suspicious activity.
- Clear documentation can help reduce MTTR by allowing the next analyst to understand the investigation quickly.
- Delayed alert acknowledgement may increase the risk of missed or late incident response.
- The available evidence does not currently confirm malicious activity.
- Further investigation is required to determine whether the alert represents a true positive.
- The detection rule should be reviewed if the same benign activity repeatedly generates alerts.
- Delayed escalation may increase the risk of further compromise.
- The affected host or account should be contained according to procedure if active compromise is confirmed.

---

## CV Bullet

Completed a TryHackMe SOC operations room focused on SLA, MTTD, MTTA, MTTR, alert volume, false positive rate, alert escalation rate, threat detection rate, alert handling efficiency, and the L1 analyst’s impact on SOC performance metrics.

---

## Interview Explanation

In this room, I learned that SOC performance is measured not only by the number of alerts closed, but also by how quickly and accurately alerts are detected, acknowledged, investigated, escalated, and resolved.

As an L1 SOC analyst, I can improve SOC metrics by monitoring the alert queue, acknowledging alerts on time, documenting evidence clearly, escalating suspicious activity correctly, and identifying repeated false positives that may require detection tuning.

I also learned that a high alert volume can overwhelm analysts, a high false positive rate can create alert fatigue, a high MTTA can show delayed triage, and a high MTTR can indicate weak escalation or unclear response procedures.

---

## Personal SOC Relevance

My previous security operations experience is relevant to this topic because SOC metrics require monitoring discipline, anomaly observation, procedure-based escalation, operational record tracking, and accurate incident documentation.

The connection is clear:

- Monitoring and surveillance observation → alert queue monitoring
- Anomaly observation → suspicious activity recognition
- Operational record tracking → SLA, MTTA, and MTTR awareness
- Incident documentation → clear evidence notes and escalation handoff
- Procedure-based reporting → structured SOC escalation workflow
- Shift discipline → consistent alert acknowledgement and queue review

---

## Lessons Learned

This room helped me understand that SOC performance depends on both technical detection and analyst behavior. L1 analysts play an important role in improving MTTA, MTTR, escalation quality, and false positive handling.

The most important lesson is that fast alert closure is not enough. A good SOC analyst must acknowledge alerts quickly, investigate carefully, document clearly, escalate correctly, and support continuous improvement by reporting repeated false positives and detection gaps.

