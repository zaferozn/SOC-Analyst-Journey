# Human Attack Vectors

## Overview

Human attack vectors are attack methods that target people instead of directly attacking technical systems.

Attackers target humans because employees can provide access to valuable systems, accounts, mailboxes, databases, VPNs, internal tools, and sensitive information.

A simple idea:

```text
Instead of breaking the gate, the attacker may convince the gatekeeper to open it.
Social Engineering

Social engineering is the manipulation of people into helping an attacker, either knowingly or unknowingly.

Social engineering exploits human psychology instead of technical vulnerabilities.

Successful social engineering attacks usually appear:

* Trustworthy
* Emotional
* Urgent
* Legitimate

Attackers may create fear, curiosity, urgency, pressure, greed, or confusion to make the victim act quickly.

Common Human Attack Vectors

Phishing

Phishing is a social engineering attack where attackers trick users into clicking malicious links, opening attachments, or entering credentials into fake login pages.

A phishing attack can lead to credential theft, malware execution, unauthorized login, mailbox compromise, data theft, or internal phishing.

Malware Downloads

Attackers can trick users into downloading malware through fake installers, cracked software, fake updates, fake CAPTCHAs, malicious QR codes, SEO poisoning, or fake download buttons.

Deepfakes

Deepfakes are AI-generated audio or video content used to impersonate real people.

Attackers may impersonate executives, managers, colleagues, family members, or business partners to create trust and urgency.

Deepfakes can increase the risk of fraud, social engineering, and business email compromise.

Impersonation

Impersonation means pretending to be someone else.

Attackers may impersonate IT support, managers, executives, vendors, banks, delivery companies, or trusted colleagues.

This can lead to credential theft, MFA approval abuse, remote access installation, unauthorized password resets, account compromise, or malware execution.

Other Human Attack Vectors

Other human attack vectors include:

* USB drop campaigns
* Physical attacks
* Insider threats
* Fake job offers
* Vishing
* Smishing
* QR phishing
* Business Email Compromise

Vishing means voice phishing over phone calls.

Smishing means phishing through SMS or text messages.

Business Email Compromise, or BEC, is a social engineering attack where attackers compromise or impersonate business email accounts to trick employees into sending money, data, or sensitive information.

Why Human Attack Vectors Matter

Human-targeted attacks often create the first step of a larger cyber incident.

A phishing email may lead to credential theft.

Credential theft may lead to unauthorized login.

Unauthorized login may lead to mailbox compromise.

Mailbox compromise may lead to internal phishing.

Internal phishing may lead to more compromised accounts.

A malicious download may lead to endpoint compromise.

Endpoint compromise may lead to lateral movement or ransomware.

Mitigation and Detection

Defending against human attack vectors involves two key tasks:

* Mitigation
* Detection

Mitigation means preventing an attack or reducing its chance and impact.

Detection means identifying suspicious or malicious activity when prevention fails.

No mitigation is perfect. Some attacks will bypass defenses, and this is where SOC detection and investigation skills become critical.

Mitigation Examples

Anti-Phishing Solutions

Anti-phishing tools help block phishing emails before users see them.

This reduces the number of phishing emails reaching employees and makes SOC investigation easier.

Antivirus / EDR

Antivirus and EDR tools help prevent or detect malware execution on endpoints.

Antivirus usually detects known malicious files.

EDR provides deeper endpoint visibility and response capability.

Trust But Verify

Suspicious requests should be verified through a second channel, even if they appear to come from a manager, CEO, IT department, or trusted colleague.

For example, if someone claiming to be the CEO asks for an urgent money transfer or password reset, the user should verify the request through an official channel.

Security Awareness Training

Security awareness training teaches employees how to recognize and report phishing, impersonation, suspicious attachments, fake login pages, vishing, smishing, and social engineering attempts.

Phishing simulations can help reinforce this training.

SOC Investigation Questions

When investigating human-targeted attacks, a SOC analyst should ask:

* Who was targeted?
* What access does the user have?
* What did the user click, open, download, or share?
* Was there a successful login?
* Was the login location suspicious?
* Was MFA triggered or bypassed?
* Were emails forwarded or deleted?
* Were files downloaded?
* Were other users contacted from the compromised account?
* Was malware executed on the endpoint?

Technical Evidence

A SOC analyst should connect human behavior with technical evidence.

Examples of technical evidence include:

* Email headers
* Sender domains
* URLs
* Attachment hashes
* Login logs
* MFA events
* EDR alerts
* Process execution
* DNS queries
* Network connections
* File downloads
* Mailbox forwarding rules

Example Scenarios

Suspicious Login Location

A user account logs in from an unusual location.

Possible SOC actions:

* Review authentication logs
* Check IP address, location, time, and device
* Review MFA activity
* Compare the login with normal user behavior
* Revoke sessions if needed
* Force password reset if compromise is suspected
* Escalate according to the SOC process

Phishing Email

A suspicious email is sent to employees.

Possible SOC actions:

* Review sender address and domain
* Inspect links and attachments
* Check email headers
* Identify affected recipients
* Check whether any user clicked the link
* Block malicious domains or senders
* Remove emails from inboxes if needed

Password Reset Impersonation

An attacker attempts to request a password reset on behalf of someone else.

Possible SOC actions:

* Verify the request through an official channel
* Confirm the user identity
* Check whether the request came from an authorized source
* Reject suspicious requests
* Notify the affected user or IT team
* Document the event

SOC Analyst Perspective

Human attack vectors are dangerous because attackers can bypass technical defenses by manipulating users.

As a SOC Analyst, I should investigate both the human action and the technical evidence.

I should not assume that a user action is safe just because it looks normal at first.

A suspicious login, phishing email, password reset request, or unusual communication may be the first sign of a larger attack.

The key SOC mindset is:
Trust but verify.
Never assume.
Observe → Verify → Document → Escalate
Personal Reflection

My previous work experience trained me to stay alert, question suspicious behavior, verify information, and think about what could go wrong before it happens.

These habits are directly relevant to SOC work.

In a SOC environment, this mindset helps with phishing investigations, suspicious login analysis, impersonation attempts, social engineering cases, and incident escalation.

I should turn this instinct into a professional analyst skill:
Observe → Verify → Document → Escalate
Analyst Summary

Human attack vectors are dangerous because attackers can manipulate users to gain access instead of directly exploiting technical systems.

Social engineering attacks such as phishing, impersonation, malware downloads, deepfakes, and fake support requests can lead to credential theft, malware execution, unauthorized access, and data loss.

As a SOC Analyst, I would investigate both the human behavior and the technical evidence, such as email activity, login logs, MFA events, endpoint alerts, and network connections.

Core Memory Lines

Human attack vector = attacking people to gain access.

Social engineering = manipulating people instead of exploiting technical flaws.

Phishing can lead to credential theft.

Credential theft can lead to account compromise.

Malware downloads can lead to endpoint compromise.

Impersonation can lead to unauthorized actions.

Deepfakes can make impersonation more convincing.

Mitigation reduces risk, but detection is still necessary.

A SOC Analyst should connect human behavior with technical evidence.

Trust but verify.

Never assume.

Observe, verify, document, escalate.
```text
