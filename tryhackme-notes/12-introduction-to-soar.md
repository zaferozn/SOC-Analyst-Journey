# Introduction to SOAR

Security Operations Centers use several tools, including SIEM, EDR, firewalls, IAM, ticketing systems, and threat intelligence platforms. Traditional SOC teams often face alert fatigue, disconnected tools, repetitive manual work, undocumented procedures, communication problems, and staff shortages.

Security Orchestration, Automation, and Response (SOAR) helps address these challenges by connecting security tools, automating repeatable tasks, and organising incident response through structured workflows.

## Core SOAR Capabilities

### 1. Orchestration

Orchestration connects different security tools within one interface.

During a VPN brute-force investigation, a traditional analyst may need to:

- Review login history in the SIEM
- Check the source IP using threat intelligence
- Search for successful authentication attempts
- Disable a compromised account through IAM
- Create and update an investigation ticket

SOAR coordinates these tools through predefined workflows called **playbooks**.

### 2. Automation

Automation allows SOAR to execute playbook steps without requiring repeated manual actions.

For example, SOAR may automatically:

1. Receive an alert from the SIEM.
2. Review the user’s previous login activity.
3. Enrich the source IP using threat intelligence.
4. Search for successful logins.
5. Disable the account when compromise is suspected.
6. Create a ticket containing the investigation evidence.

Automation reduces response time and allows analysts to focus on alerts requiring human judgement.

### 3. Response

SOAR can perform containment and remediation actions through connected security tools.

Possible response actions include:

- Blocking an IP address on a firewall
- Disabling a user account through IAM
- Isolating an endpoint through EDR
- Deleting malicious emails
- Creating or updating an incident ticket
- Escalating the case to another team

## SOAR Playbooks

A playbook is a predefined workflow for investigating and responding to a recurring type of security alert.

Playbooks contain decision points. The result of one step determines the next action.

## Phishing Playbook

A phishing playbook may follow these steps:

1. Receive a suspicious email alert.
2. Create an investigation ticket.
3. Extract URLs and attachments.
4. Calculate attachment hashes.
5. Submit URLs and hashes to threat intelligence platforms.
6. Determine whether the indicators are malicious.
7. Send uncertain cases for manual sandbox analysis.
8. Delete confirmed malicious emails.
9. Notify affected users.
10. Update the ticket with collected IOCs and findings.

The playbook automates enrichment and remediation, while analysts review uncertain results and make final decisions.

## CVE Patching Playbook

A CVE patching playbook may:

1. Monitor vulnerability advisories.
2. Extract newly published CVE information.
3. Check whether the CVE was previously addressed.
4. Determine whether the vulnerability affects organisational assets.
5. Create and assign a patching ticket.
6. Identify affected systems.
7. Verify whether a patch exists.
8. Test the patch in a controlled environment.
9. Deploy the patch to affected assets.
10. Scan the systems again to confirm remediation.
11. Develop mitigation steps when no patch is available.
12. Update and close the ticket.

This workflow helps reduce vulnerability backlogs and standardises patch management.

## Do SOAR Platforms Replace Analysts?

SOAR does not replace SOC analysts. It automates repetitive actions, but analysts are still required to:

- Investigate complex incidents
- Validate automated findings
- Consider business context
- Make escalation and containment decisions
- Handle uncertain or conflicting evidence
- Design and improve playbooks

## Final Summary

SOAR combines orchestration, automation, and response to connect security tools and standardise incident handling. It reduces repetitive manual work, improves response time, and helps analysts manage alerts more consistently. Human analysts remain essential for judgement, verification, and complex investigations.
## Learning Source

This note was created while completing the **Introduction to SOAR** room in the TryHackMe learning environment. The concepts have been summarised and paraphrased in my own words for educational and portfolio documentation purposes.

Platform: TryHackMe  
Room: Introduction to SOAR
