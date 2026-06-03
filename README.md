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
The current lab uses an Ubuntu Linux VM as an SSH target system. The current phase focuses on Linux authentication log analysis. Wazuh integration is planned as a later extension.

## Current Status
Week 1 foundation is in progress. Case 001 has been developed from SSH failed login pattern analysis using Linux journal logs. Case 002 has been added to document failed SSH login attempts followed by a successful SSH login from the same source IP address.
