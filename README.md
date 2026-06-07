# SOC Analyst Journey

This repository documents my hands-on transition into Security Operations.

## Focus Areas
- Linux authentication log analysis
- SSH failed login investigation
- Alert triage fundamentals
- Source IP and username extraction
- Incident-style documentation
- SOC-focused English writing practice

## Portfolio Cases
- [Case 001 - SSH Failed Login Pattern Investigation](cases/case-001-ssh-failed-login-pattern-investigation.md)
- [Case 002 - Failed-to-Successful SSH Login Review](cases/case-002-failed-to-successful-ssh-login-review.md)
  
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

## Current Lab Status
The current lab environment includes a dedicated Wazuh SIEM server and a monitored Ubuntu endpoint. The current phase focuses on Wazuh agent validation, endpoint event ingestion, SSH authentication alert visibility, and SIEM-based triage practice.

## Current Status
The Linux authentication log analysis foundation has been completed through Case 001 and Case 002. These cases document SSH failed login analysis and failed-to-successful SSH login correlation using raw Linux journal logs.
The current phase focuses on Wazuh SIEM practice, including Wazuh server deployment, dashboard validation, Ubuntu agent enrollment, endpoint event ingestion, and SSH authentication alert analysis.
