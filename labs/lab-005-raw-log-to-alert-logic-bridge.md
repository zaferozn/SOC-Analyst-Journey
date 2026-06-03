# Lab 005 - Raw Log to Alert Logic Bridge

## Objective

This lab explains how a raw SSH authentication log can be converted into observable fields, suspicious patterns, basic alert logic, severity assessment, and escalation decisions.

The goal is to understand the analyst workflow behind SIEM alerts before moving deeper into Wazuh alert triage.

## Environment / Data Source

Host: `soclab`  
Tool: `journalctl`, `grep`, manual analysis  
Log source: Linux SSH authentication logs  
Service: SSH  
Use case: Failed SSH login review and failed-to-successful login correlation  

## Why This Lab Matters

A SOC analyst should not only look at dashboards or alerts.

Before using a SIEM alert, the analyst should understand how raw log data becomes an alert.

The basic investigation chain is:

```text
Raw log line
→ Observable fields
→ Pattern
→ Alert logic
→ Severity
→ Escalation decision
```

This lab connects the previous SSH authentication labs to future Wazuh alert triage.

## Raw Log Example 1 - Failed SSH Login

Example raw log:

```text
Apr 28 08:22:44 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
```

## Observable Field Extraction

| Field | Extracted Value |
|---|---|
| Timestamp | `Apr 28 08:22:44` |
| Host | `soclab` |
| Process | `sshd` |
| Process ID | `5411` |
| Event type | `Failed password` |
| Username | `Fakeuser` |
| Username status | Invalid user |
| Source IP | `192.168.64.1` |
| Source port | `53939` |
| Protocol | `ssh2` |

## Field Meaning

The raw log shows that an SSH authentication attempt failed.

The username `Fakeuser` does not exist on the Linux system.

The source IP `192.168.64.1` initiated the SSH connection.

The source port `53939` is a temporary client-side port.

The protocol `ssh2` indicates SSH protocol version 2.

## Single Event Interpretation

A single failed login attempt does not automatically confirm an attack.

Possible explanations include:

- User typo
- Wrong username
- Misconfigured script
- Controlled lab activity
- Unauthorized login attempt

A SOC analyst should avoid over-escalating a single isolated failed login event.

## Raw Log Example 2 - Repeated Failed SSH Logins

Example raw logs:

```text
Apr 28 08:22:44 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:48 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:54 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:23:16 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:21 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:25 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
```

## Pattern Identification

The repeated logs show:

- Same source IP: `192.168.64.1`
- Same service: SSH
- Same event type: `Failed password`
- Multiple invalid username attempts
- Six failed attempts within approximately 41 seconds

This creates a repeated failed login pattern.

## Pattern Statement

```text
Multiple failed SSH login attempts were observed from the same source IP within a short time window.
```

## Possible SOC Interpretation

This pattern may indicate:

- Brute-force activity
- Username guessing
- Credential guessing
- Scripted authentication attempts
- Controlled lab-generated activity

In this lab, the activity was expected because it was generated in a controlled environment.

In a real SOC environment, this pattern would require further review.

## Alert Logic Example 1 - Repeated Failed SSH Login

Basic alert logic:

```text
IF event_type = "Failed password"
AND service = "SSH"
AND source_ip = same source IP
AND failed_attempt_count >= 5
AND time_window <= 5 minutes
THEN generate alert = Possible SSH brute-force activity
```

## Alert Logic Explanation

This alert logic focuses on repeated failed SSH authentication attempts from the same source IP within a short time window.

The goal is not to alert on every single failed login.

The goal is to alert when failed login events form a suspicious pattern.

## Severity Assessment - Repeated Failed Login

Suggested severity:

```text
Medium
```

Reasoning:

Repeated failed SSH login attempts may indicate brute-force activity or unauthorized access attempts.

However, if there is no successful login and no post-authentication activity, the severity may remain medium unless the source IP, target account, or volume increases.

## Escalation Decision - Repeated Failed Login

Escalation may be required if:

- The source IP is external or unknown.
- The number of attempts is high.
- Multiple usernames are targeted.
- Multiple hosts are targeted.
- The same source IP appears repeatedly over time.
- A successful login occurs after the failed attempts.

Escalation may not be required if:

- The activity is confirmed as a lab test.
- The source IP is known and expected.
- There is no successful login.
- The event volume is low.
- The user or administrator confirms the activity.

## Raw Log Example 3 - Failed Attempts Followed by Successful Login

Example raw logs:

```text
May 10 14:59:35 soclab sshd[23736]: Failed password for invalid user fakeuser from 192.168.64.1 port 57213 ssh2
May 10 15:00:04 soclab sshd[23736]: Failed password for invalid user fakeuser from 192.168.64.1 port 57213 ssh2
May 10 15:00:08 soclab sshd[23736]: Failed password for invalid user fakeuser from 192.168.64.1 port 57213 ssh2
May 10 15:00:29 soclab sshd[23739]: Accepted password for analyst from 192.168.64.1 port 57215 ssh2
```

## Failed-to-Successful Pattern

The logs show:

- Three failed SSH login attempts
- One successful SSH login
- Same source IP: `192.168.64.1`
- Failed attempts occurred before successful authentication
- Valid user involved in the successful login: `analyst`

## Pattern Statement

```text
Multiple failed SSH login attempts were followed by a successful SSH login from the same source IP.
```

## SOC Interpretation

This pattern is more suspicious than failed attempts alone.

A successful login after repeated failed attempts may indicate that valid credentials were eventually used.

In this lab, the successful login was expected because the activity was manually generated.

In a real SOC environment, this pattern should be reviewed carefully.

## Alert Logic Example 2 - Failed Attempts Followed by Successful Login

Basic alert logic:

```text
IF failed_login_count >= 3
AND event_type = "Accepted password"
AND source_ip = same source IP
AND time_window <= 10 minutes
THEN generate alert = Successful SSH login after repeated failed attempts
```

## Alert Logic Explanation

This logic detects a risky authentication sequence.

The key point is correlation.

The successful login is not reviewed alone.

It is reviewed together with previous failed attempts from the same source IP.

## Severity Assessment - Failed-to-Successful Login

Suggested severity:

```text
High
```

Reasoning:

A successful login after repeated failed attempts may indicate possible credential guessing followed by valid authentication.

If the login is unexpected, this may represent unauthorized access.

## Escalation Decision - Failed-to-Successful Login

Escalation is recommended if:

- The successful login is not expected.
- The source IP is unknown or external.
- The user does not recognize the login.
- The account is privileged.
- The login occurs outside normal working hours.
- Suspicious activity occurs after login.
- The same source IP targeted other users or hosts.

Escalation may not be required if:

- The activity is confirmed as a controlled lab test.
- The source IP is expected.
- The user confirms the login.
- No suspicious post-authentication activity is observed.

## Severity Comparison Table

| Pattern | Example | Suggested Severity | Reason |
|---|---|---|---|
| Single failed login | One failed SSH attempt | Low | May be user error or typo |
| Repeated failed logins | Multiple failed attempts from same IP | Medium | May indicate brute-force or username guessing |
| Failed attempts followed by success | Failed attempts followed by accepted password | High | May indicate successful credential guessing |
| Success followed by suspicious activity | Login followed by unusual commands or privilege escalation | Critical | May indicate active compromise |

## Raw Log to Alert Workflow

The complete workflow is:

```text
1. Read the raw log line.
2. Extract observable fields.
3. Identify whether the event is isolated or repeated.
4. Look for patterns across time.
5. Build or understand alert logic.
6. Assess severity based on context.
7. Decide whether escalation is required.
8. Document the finding in analyst language.
```

## Analyst Questions

A SOC analyst should ask:

- What was observed?
- Where was it observed?
- Which user was involved?
- Which source IP was involved?
- Was the username valid or invalid?
- Was the authentication successful or failed?
- Did repeated failed attempts occur?
- Did a successful login follow failed attempts?
- Is the source IP expected?
- Is the login time normal?
- Is the account privileged?
- Was any suspicious post-login activity observed?
- Should the event be escalated?

## Evidence to Collect

Useful evidence includes:

- Raw SSH authentication log lines
- Timestamp
- Hostname
- Source IP
- Source port
- Username
- Event type
- Number of attempts
- Time window
- Successful login evidence
- Session open and close events
- Post-authentication activity if available

## Example Commands

```bash
journalctl --since "2 hours ago" | grep -i "failed password"

journalctl --since "2 hours ago" | grep -i "accepted password"

journalctl --since "2 hours ago" | grep -Ei "failed password|accepted password|invalid user"

journalctl --since "2 hours ago" | grep -i "failed password" | wc -l

journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'from [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c

journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'invalid user [^ ]+' | sort | uniq -c
```

## Analysis

This lab shows how raw SSH authentication logs can be transformed into structured SOC reasoning.

A single raw log line provides useful observable fields, but it may not be enough to confirm malicious activity.

Repeated events create patterns.

Patterns can become alert logic.

Alert logic supports severity assessment.

Severity and context support escalation decisions.

This is the basic reasoning chain behind many SIEM alerts.

## Risk

The main security risk is unauthorized access through SSH.

Repeated failed SSH login attempts may indicate brute-force or username-guessing activity.

A successful login after repeated failed attempts increases the risk because it may indicate that valid credentials were used.

If suspicious post-login activity occurs, the risk may escalate to active compromise.

## Recommended Next Steps

- Continue practicing raw SSH log review.
- Compare raw Linux logs with future Wazuh alerts.
- Identify which raw fields appear in Wazuh alert fields.
- Practice severity assessment using different authentication scenarios.
- Review post-authentication activity after successful logins.
- Build a Wazuh alert field breakdown lab next.

## Final SOC Summary

This lab connected raw SSH authentication logs to SOC alert logic. Failed SSH login events were reviewed by extracting observable fields such as timestamp, host, event type, username, source IP, source port, and protocol. Repeated failed login attempts from the same source IP were interpreted as a possible brute-force or username-guessing pattern. Failed attempts followed by a successful login were assessed as higher risk because they may indicate credential guessing followed by valid authentication. The lab demonstrated how raw logs can support alert logic, severity assessment, and escalation decisions.

## Lessons Learned

This lab showed that alerts are not magic outputs from a dashboard. Alerts are based on raw log fields, repeated patterns, rule logic, severity, and context. Before relying on Wazuh or any SIEM dashboard, a SOC analyst must understand how raw log events become meaningful security alerts.

## CV Wording

Built a SOC lab workflow connecting raw SSH authentication logs to observable fields, suspicious patterns, basic alert logic, severity assessment, and escalation decisions.

## Interview Answer

Before working only with SIEM dashboards, I created a bridge lab to understand how raw SSH logs become alert logic. I practiced extracting fields such as timestamp, host, user, source IP, event type, source port, and protocol. Then I connected repeated failed login events to suspicious patterns, severity levels, and escalation criteria. This helped me understand how a SIEM alert is built from raw log activity.

## SOC English Sentences

A raw SSH authentication log was reviewed and converted into observable fields.

Multiple failed SSH login attempts were observed from the same source IP.

The repeated failed login pattern may indicate brute-force or username-guessing activity.

A successful login after repeated failed attempts increases the severity of the event.

The successful login should be reviewed to determine whether it was expected and authorized.

Alert logic should be based on repeated patterns, not only on a single failed login.

Severity should be assessed based on context, including source IP, username, login result, time window, and post-authentication activity.

Escalation is recommended if the successful login cannot be verified as legitimate.
