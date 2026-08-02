Copy everything below and paste it into your **Lab 012 Markdown file**:

````markdown
# Lab 012 - Wazuh SSH False-Positive Review and Alert Disposition

## Executive Summary

This lab investigated repeated failed SSH authentication attempts against the legitimate Linux user account `analyst` on the monitored `ubuntu-agent` endpoint.

Three failed password attempts were generated from the Mac host at `192.168.64.1`. Wazuh detected the individual authentication failures and generated a correlated level 10 alert under Rule ID `2502`.

A successful SSH login was then completed from the same source IP address against the same user account. The Wazuh alerts were compared with the original Linux SSH authentication logs to confirm the complete event sequence.

Because the source IP belonged to an authorized internal device, only one legitimate account was involved, and the failed attempts were followed by a successful login from the same source, the activity was classified as a benign true positive. Escalation was not required.

## Objective

The objective of this lab was to investigate repeated SSH authentication failures and determine whether the resulting Wazuh alert represented malicious brute-force activity or legitimate user behavior.

The investigation focused on:

- Validating the Wazuh alert against the original Linux logs
- Identifying the source IP and destination account
- Reviewing the failed-to-successful authentication sequence
- Assessing the possible security risk
- Assigning an appropriate alert disposition
- Determining whether escalation was required

## Environment / Data Source

Host: `ubuntu-agent`  
Wazuh manager: `wazuh-server`  
Wazuh agent ID: `001`  
Wazuh agent IP: `192.168.64.10`  
Source host: MacBook  
Source IP: `192.168.64.1`  
User account: `analyst`  
Protocol: SSH  
Log source: Linux `journald` SSH authentication logs  
Tool: Wazuh SIEM  
Investigation date: August 1, 2026  

## Lab Architecture

```text
MacBook
192.168.64.1
    |
    | SSH authentication attempts
    v
ubuntu-agent
192.168.64.10
    |
    | Wazuh agent forwards security events
    v
wazuh-server
192.168.64.9
    |
    | Alerts reviewed through the dashboard
    v
Wazuh Threat Hunting
```

## Activity Generation

The following command was executed from the Mac Terminal:

```bash
ssh analyst@192.168.64.10
```

An incorrect password was entered three times to generate repeated authentication failures.

After the failed attempts, the same SSH command was executed again:

```bash
ssh analyst@192.168.64.10
```

The correct password was then entered, resulting in a successful SSH login.

## Observed Activity

Multiple failed SSH authentication attempts were observed against the legitimate user account `analyst`.

The attempts originated from the Mac host at `192.168.64.1` and targeted the monitored Ubuntu endpoint at `192.168.64.10`.

Wazuh generated individual authentication-failure alerts and a higher-severity correlated alert after the password was entered incorrectly multiple times.

A successful SSH authentication was subsequently observed from the same source IP address against the same user account.

The following entities were involved:

```text
Source IP: 192.168.64.1
Destination host: ubuntu-agent
Destination IP: 192.168.64.10
Destination user: analyst
Protocol: SSH
```

## Evidence

### Evidence 1 - Correlated Authentication-Failure Alert

Wazuh generated a correlated alert after multiple failed password attempts.

```text
Rule ID: 2502
Rule level: 10
Rule description: syslog: User missed the password more than one time
Agent name: ubuntu-agent
Agent ID: 001
Agent IP: 192.168.64.10
Source IP: 192.168.64.1
Destination user: analyst
Manager name: wazuh-server
MITRE ATT&CK technique: T1110 - Brute Force
MITRE ATT&CK tactic: Credential Access
```

The alert indicated that the same user had entered an incorrect password multiple times within a short period.

<!-- Screenshot will be inserted here after upload. -->

### Evidence 2 - Individual Authentication-Failure Events

Wazuh generated individual SSH authentication-failure alerts for the incorrect password attempts.

```text
Rule ID: 5760
Rule level: 5
Rule description: sshd: authentication failed
```

Three individual failed authentication events were visible in Wazuh Threat Hunting.

### Evidence 3 - Successful SSH Authentication

A successful SSH authentication was later detected.

```text
Rule ID: 5715
Rule level: 3
Rule description: sshd: authentication success
Source IP: 192.168.64.1
Source port: 63227
Destination user: analyst
Agent name: ubuntu-agent
MITRE ATT&CK technique: T1078 - Valid Accounts
MITRE ATT&CK technique: T1021 - Remote Services
```

The full log confirmed:

```text
Accepted password for analyst from 192.168.64.1 port 63227 ssh2
```

A PAM session-opened event was also generated:

```text
Rule ID: 5501
Rule level: 3
Rule description: PAM: Login session opened
```

<!-- Screenshot will be inserted here after upload. -->

### Evidence 4 - Raw Log Validation

The original SSH authentication logs were reviewed directly on `ubuntu-agent` using:

```bash
sudo journalctl _COMM=sshd --since "30 minutes ago" | grep -E "Failed password|Accepted password|authentication failure"
```

The command returned:

```text
Aug 01 19:15:12 ubuntu-agent sshd[113348]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.1 user=analyst
Aug 01 19:15:14 ubuntu-agent sshd[113348]: Failed password for analyst from 192.168.64.1 port 62875 ssh2
Aug 01 19:15:17 ubuntu-agent sshd[113348]: Failed password for analyst from 192.168.64.1 port 62875 ssh2
Aug 01 19:15:22 ubuntu-agent sshd[113348]: Failed password for analyst from 192.168.64.1 port 62875 ssh2
Aug 01 19:15:23 ubuntu-agent sshd[113348]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.1 user=analyst
Aug 01 19:22:29 ubuntu-agent sshd[113363]: Accepted password for analyst from 192.168.64.1 port 63227 ssh2
```

The raw endpoint logs confirmed three failed password attempts followed by a successful authentication from the same source IP address and against the same user account.

<!-- Screenshot will be inserted here after upload. -->

## Event Timeline

```text
19:15:12 - PAM authentication failure recorded
19:15:14 - First failed password event
19:15:17 - Second failed password event
19:15:22 - Third failed password event
19:15:23 - Additional PAM authentication-failure summary recorded
19:22:29 - Successful password authentication
```

## Analysis

The repeated failed SSH authentication attempts initially appeared suspicious because repeated password failures may indicate password guessing, credential misuse, or brute-force activity.

Wazuh correctly detected the individual failures and generated a correlated level 10 alert under Rule ID `2502`. The behavior was also mapped to MITRE ATT&CK technique `T1110 - Brute Force`.

However, contextual analysis reduced the level of suspicion.

The source IP address `192.168.64.1` belonged to the authorized Mac host used in the lab. All attempts targeted one legitimate user account, `analyst`. The same account successfully authenticated from the same source IP shortly afterward.

No invalid usernames, additional user accounts, unknown external source addresses, or suspicious post-authentication commands were observed.

The raw Linux logs matched the Wazuh events and confirmed that the sequence consisted of incorrect password entries followed by a legitimate successful login.

## Alternative Explanation

The authentication pattern could have represented malicious password guessing if:

- The source IP had been unknown or external
- Multiple accounts had been targeted
- The account owner had denied initiating the connection
- The successful login had originated from an unfamiliar device
- Suspicious commands had been executed after authentication
- Similar attempts had continued over an extended period

In this controlled lab, none of those conditions were present.

The most likely explanation was that an authorized user entered an incorrect password several times before successfully authenticating.

## Risk

Repeated SSH authentication failures may indicate:

- Brute-force activity
- Password guessing
- Credential misuse
- Unauthorized access attempts
- Attempted account compromise

A successful login following repeated failures can increase the apparent risk because it may indicate that an attacker successfully discovered or obtained the password.

In this case, the source device and user activity were authorized. Therefore, the final assessed risk was low.

## Alert Disposition

```text
Classification: Benign true positive
Initial severity: High
Rule level: 10
Final assessed severity: Low
Escalation required: No
Closure reason: Authorized user entered an incorrect password several times before successfully authenticating
```

The alert was classified as a benign true positive rather than a technical false positive because the detected authentication failures genuinely occurred and the Wazuh detection logic operated correctly.

The activity was benign because it had a legitimate explanation and did not represent an actual security incident.

## Recommended Next Steps

1. Confirm that the source IP belongs to an authorized device.
2. Confirm that the account owner initiated the authentication attempts.
3. Review the successful login event and associated session activity.
4. Check whether any additional accounts were targeted.
5. Review commands executed after successful authentication.
6. Document the investigation findings and alert disposition.
7. Close the alert if the activity is confirmed as authorized.
8. Escalate if the user denies initiating the activity.
9. Escalate if the source IP is unknown or unauthorized.
10. Escalate if suspicious post-authentication behavior is identified.

## Escalation Decision

Escalation was not required because:

- The source IP belonged to a known internal device
- The destination account was legitimate
- Only one account was targeted
- The same source and user later completed a successful login
- The activity was intentionally generated in a controlled lab
- No suspicious post-authentication behavior was observed

In a production environment, the analyst should verify the activity with the account owner before closing the alert.

## MITRE ATT&CK Mapping

### T1110 - Brute Force

The repeated authentication failures resembled brute-force or password-guessing behavior.

The mapping was relevant to the initial alert pattern but did not prove that malicious brute-force activity had occurred.

### T1078 - Valid Accounts

The successful authentication involved a valid Linux account.

This technique would become more concerning if an unauthorized actor successfully authenticated using compromised credentials.

### T1021 - Remote Services

SSH was used to access the Ubuntu endpoint remotely.

Remote service usage is legitimate in many environments but should be reviewed when it follows repeated authentication failures.

## Final SOC Summary

Wazuh detected multiple failed SSH authentication attempts against the legitimate user account `analyst` on the monitored `ubuntu-agent` endpoint. Three failed password attempts originated from the internal source IP address `192.168.64.1`, causing Wazuh to generate individual Rule `5760` alerts and a correlated level 10 alert under Rule `2502`. A successful SSH authentication was later observed from the same source IP against the same account under Rule `5715`. Raw Linux authentication logs confirmed the complete failed-to-successful event sequence. Because the source device and user activity were authorized and no suspicious post-authentication behavior was identified, the event was classified as a benign true positive. Escalation was not required.

## Lessons Learned

This lab demonstrated that a SOC analyst should not classify an event solely according to its rule level, description, or MITRE ATT&CK mapping.

A high-severity alert does not automatically confirm malicious activity. The analyst must review the source IP, destination account, event timeline, successful authentication activity, raw endpoint logs, and surrounding context.

This lab also demonstrated the distinction between a false positive and a benign true positive:

- A false positive occurs when the detection logic incorrectly identifies benign activity.
- A benign true positive occurs when the detected behavior genuinely happened but had a legitimate explanation.

The Wazuh detection was technically correct because the repeated authentication failures occurred. However, the activity did not represent a real security incident.
