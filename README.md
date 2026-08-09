# SOC Analyst Journey

This repository documents my hands-on transition into Security Operations.

## Focus Areas

- Linux authentication log analysis
- SSH failed login investigation
- Alert triage fundamentals
- Source IP and username extraction
- Wazuh SIEM alert review
- Raw log vs SIEM alert comparison
- SSH brute-force investigation
- False positive analysis
- Email header analysis
- Phishing email investigation
- Suspicious URL and domain analysis
- DNS and reverse DNS investigation
- IOC enrichment
- Threat intelligence and reputation checks
- Incident-style documentation
- Evidence-based escalation decisions
- SOC-focused English writing practice

## Portfolio Cases

- [Case 001 - SSH Failed Login Pattern Investigation](cases/case-001-ssh-failed-login-pattern-investigation.md)
- [Case 002 - Failed-to-Successful SSH Login Review](cases/case-002-failed-to-successful-ssh-login-review.md)
- [Case 003 - Wazuh SSH Brute-Force Investigation](cases/case-003-wazuh-ssh-brute-force-investigation.md)
- [Case 004 - Suspicious Phishing Email Investigation](cases/case-004-phishing-email-investigation.md)

## Lab Notes

- [Lab 001 - SSH Log Observation](labs/lab-001-ssh-log-observation.md)
- [Lab 002 - Fresh SSH Authentication Baseline](labs/lab-002-fresh-ssh-authentication-baseline.md)
- [Lab 003 - Authentication Triage Scenarios](labs/lab-003-authentication-triage-scenarios.md)
- [Lab 004 - Wazuh Environment Validation](labs/lab-004-wazuh-environment-validation.md)
- [Lab 005 - Raw Log to Alert Logic Bridge](labs/lab-005-raw-log-to-alert-logic-bridge.md)
- [Lab 006 - Wazuh SIEM Installation, Validation, and Dashboard Access Troubleshooting](labs/lab-006-wazuh-siem-installation-validation-and-dashboard-access-troubleshooting.md)
- [Lab 007 - Wazuh Agent Enrollment and First Event Validation](labs/lab-007-wazuh-agent-enrollment-and-first-event-validation.md)
- [Lab 008 - Wazuh Authentication Event Review and First Alert Triage](labs/lab-008-wazuh-authentication-event-review-and-first-alert-triage.md)
- [Lab 009 - Wazuh Alert Field Extraction and Raw Log Comparison](labs/lab-009-wazuh-alert-field-extraction-and-raw-log-comparison.md)
- [Lab 010 - Repeated Failed SSH Login Triage from Same Source IP](labs/lab-010-repeated-failed-ssh-login-triage.md)
- [Lab 011 - Wazuh SSH Brute-Force Alert Deep Triage](labs/lab-011-wazuh-ssh-brute-force-alert-deep-triage.md)
- [Lab 012 - Wazuh SSH False Positive Review](labs/lab-012-wazuh-ssh-false-positive-review.md)
- [Lab 013 - Email Structure and Header Analysis](labs/lab-013-Email-Structure-and-Header-Analysis.md)
- [Lab 014 - Suspicious URL Analysis](labs/lab-014-suspicious-url-analysis.md)
- [Lab 015 - Suspicious URL and Domain Analysis](labs/lab-015-suspicious%20URL%20and%20domain%20analysis.md)
- [Lab 016 - False Positive Review and Alert Disposition](labs/lab-016-false-positive-review-and-alert-disposition.md)

## Supporting Notes

- [Linux Foundation Notes](labs/linux-foundation-notes.md)
- [Alert vs Raw Log Notes](labs/alert-vs-raw-log-notes.md)

## Analyst Communication Practice

- [Basic Analyst Sentence Patterns](english-drills/basic-analyst-sentence-patterns.md)
- [SSH Log Summary Practice](english-drills/ssh-log-summary-practice.md)
- [Failed-to-Successful Login Pattern](english-drills/failed-to-successful-login-pattern.md)
- [Case 001 Defense Drill](english-drills/case-001-defense-drill.md)
- [Case 002 Defense Drill](english-drills/case-002-failed-to-successful-login-drill.md)

## TryHackMe Notes

- [TryHackMe Notes](tryhackme-notes/)

## Current Lab Environment

The current lab environment includes a dedicated Wazuh SIEM server and a monitored Ubuntu endpoint.

- Wazuh server: dedicated SIEM server
- Monitored endpoint: Ubuntu agent
- Main endpoint log source: Linux authentication logs
- SIEM focus: Wazuh authentication alerts, rule fields, alert severity, source IP review, and evidence-based triage
- Phishing analysis focus: email headers, embedded URLs, domains, DNS records, infrastructure, redirects, and reputation data
- Investigation tools: Wazuh, Linux command-line tools, VirusTotal, URLScan.io, DNS lookup, and reverse DNS analysis

## Current Status

The Linux authentication investigation foundation has been developed through repeated failed SSH login analysis, failed-to-successful authentication correlation, Wazuh alert review, raw log comparison, brute-force triage, and false positive analysis.

Cases 001–003 document progressively more advanced authentication investigations, moving from raw Linux log analysis to SIEM-assisted SSH brute-force triage and evidence-based escalation decisions.

The portfolio has now expanded into phishing investigation and basic threat intelligence workflows. Labs 013–015 cover email structure and header analysis, suspicious URL investigation, DNS and reverse DNS analysis, hosting infrastructure review, redirect analysis, and reputation checking using external threat intelligence sources.

Case 004 combines these skills into an end-to-end suspicious phishing email investigation. The case evaluates email metadata, embedded URLs, associated domains and infrastructure, redirect behavior, and reputation evidence before reaching a final disposition of Suspicious / Unconfirmed.

The current portfolio direction is focused on practical SOC Analyst work: Linux log analysis, Wazuh SIEM alert triage, authentication investigation, false positive review, phishing analysis, IOC enrichment, threat intelligence support, evidence collection, incident documentation, escalation decisions, and professional analyst communication.
