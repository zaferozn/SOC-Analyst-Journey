# Lab 003 - Authentication Triage Scenarios

## Objective

This lab documents basic authentication triage practice for SOC L1 analysis.

The goal was to review short SSH authentication scenarios and classify them as expected, suspicious, or requiring more context.

## Environment / Data Source

Host: `soclab`  
Tool: Scenario-based analysis  
Log source: SSH authentication logs  
Focus: Authentication triage, source IP review, user review, severity, and escalation decision-making  

## Triage Focus

The analysis focused on:

- Failed SSH login attempts
- Successful SSH logins
- Invalid usernames
- Source IP review
- Login time review
- Post-authentication activity
- Escalation decision-making

## Core Triage Questions

For each authentication scenario, the following SOC questions were used:

- What was observed?
- Which user was involved?
- Which source IP was involved?
- Was the username valid or invalid?
- Was the login successful or failed?
- Did failed attempts occur before a successful login?
- Is the source IP expected?
- Is the login time normal for the user?
- Is there any suspicious post-authentication activity?
- Should the activity be escalated?

## Scenario 1 - Multiple Invalid User Attempts

### Log Pattern

```text
10:01 - Failed password for invalid user admin from 185.22.10.5
10:01 - Failed password for invalid user test from 185.22.10.5
10:02 - Failed password for invalid user oracle from 185.22.10.5
10:02 - Failed password for invalid user postgres from 185.22.10.5
10:03 - Failed password for invalid user backup from 185.22.10.5
```

### Assessment

Suspicious.

### Reasoning

The logs show multiple failed SSH login attempts from the same source IP targeting different invalid usernames.

This pattern may indicate username guessing, automated scanning, or brute-force style behavior.

### Severity

Medium.

### First Checks

- Check whether the source IP appeared in previous authentication events.
- Check whether any successful login occurred from the same source IP.
- Check whether the target system is expected to receive SSH connections from this IP.
- Check whether the same IP targeted multiple hosts or multiple users.
- Check whether the IP is internal, external, known, or unknown.

### Escalation Decision

Escalation may be required if the attempts continue, target multiple accounts, come from an unknown external IP, or are followed by a successful login.

## Scenario 2 - Single Successful Login

### Log Pattern

```text
09:15 - Accepted password for analyst from 192.168.64.1
09:16 - session opened for user analyst
09:18 - session closed for user analyst
```

### Assessment

Needs more context. Likely expected if the source IP and login time match normal user behavior.

### Reasoning

The logs show a single successful SSH login followed by a session open and session close event.

There are no failed attempts, invalid usernames, or repeated authentication failures in this short timeline.

A successful login is not automatically suspicious. It becomes suspicious when the source IP, login time, user account, or post-login activity does not match expected behavior.

### Severity

Low if expected.  
Medium if the source IP, login time, or user behavior is unusual.

### First Checks

- Check whether the source IP is expected for the user.
- Check whether the login time matches the user's normal behavior.
- Review session duration.
- Review post-authentication activity for anything unusual.
- Check whether the user account is privileged.

### Escalation Decision

Do not escalate if the login is expected and no suspicious activity is observed.

Escalate if the source IP is unknown, the user does not recognize the login, or suspicious activity occurs after login.

## Scenario 3 - Failed Attempts Followed by Successful Login

### Log Pattern

```text
22:41 - Failed password for analyst from 45.90.12.44
22:41 - Failed password for analyst from 45.90.12.44
22:42 - Failed password for analyst from 45.90.12.44
22:43 - Accepted password for analyst from 45.90.12.44
22:44 - session opened for user analyst
```

### Assessment

Suspicious. Requires investigation.

### Reasoning

The logs show repeated failed SSH login attempts against the same valid user, followed shortly by a successful login from the same source IP.

This pattern may indicate credential guessing followed by successful authentication.

The activity becomes more suspicious if the source IP is unknown, the login time is unusual, the user does not recognize the login, or suspicious post-authentication activity is observed.

### Severity

High if the successful login is not expected or cannot be verified.

### First Checks

- Check whether the source IP is expected for the user.
- Compare the login time with the user's normal login behavior.
- Review post-authentication activity after the successful login.
- Check whether the account has privileged access.
- Check whether the same source IP attempted logins against other users.
- Check whether additional alerts were generated after the successful login.

### Escalation Criteria

Escalate to L2 if:

- The source IP is unusual or not expected.
- The user does not recognize the login.
- Suspicious post-authentication activity is observed.
- The successful login is followed by privilege escalation, unusual commands, persistence activity, or access to sensitive resources.
- The same source IP appears across multiple users or hosts.

## Comparison Table

| Scenario | Pattern | Assessment | Severity | Escalation |
|---|---|---|---|---|
| Scenario 1 | Multiple invalid user attempts from same IP | Suspicious | Medium | Escalate if repeated or followed by success |
| Scenario 2 | Single successful login | Needs context / likely expected | Low to Medium | Escalate only if context is unusual |
| Scenario 3 | Failed attempts followed by successful login | Suspicious | High | Escalate if not verified as legitimate |

## Analyst Notes

A SOC analyst should not classify authentication activity based on one field only.

The same event can have different meanings depending on context:

- One failed login may be a user mistake.
- Repeated failed logins may indicate brute-force activity.
- Invalid usernames may indicate username guessing.
- A successful login may be normal.
- A successful login after repeated failed attempts may indicate possible account compromise.

## Risk

Authentication anomalies may indicate unauthorized access attempts, credential guessing, brute-force behavior, or account compromise.

The highest-risk scenario in this lab is repeated failed login attempts followed by a successful login from the same source IP.

## Recommended Next Steps

- Review source IP reputation and ownership if the IP is external.
- Confirm whether the user recognizes the login.
- Review successful login timestamps.
- Review session duration and post-login activity.
- Check whether the account is privileged.
- Search for the same source IP across other authentication logs.
- Escalate if the login cannot be confirmed as legitimate.

## Final SOC Summary

Three SSH authentication triage scenarios were reviewed. Multiple invalid user attempts from the same source IP were assessed as suspicious because they may indicate username guessing or brute-force style behavior. A single successful login was assessed as requiring context because successful authentication is not automatically suspicious. Failed login attempts followed by a successful login from the same source IP were assessed as suspicious and requiring investigation because the pattern may indicate credential guessing followed by successful access.

## Lessons Learned

This lab showed that SOC triage depends on context. A single failed login does not confirm an attack, and a successful login is not automatically malicious. Source IP, username validity, login frequency, login time, and post-authentication activity must be reviewed together before making an escalation decision.

## CV Wording

Practiced SOC L1 authentication triage by reviewing SSH login scenarios, classifying suspicious patterns, assessing severity, and defining escalation criteria based on source IP, username validity, login timing, and post-authentication context.

## Interview Answer

In this lab, I practiced basic SOC L1 authentication triage using short SSH login scenarios. I reviewed multiple invalid user attempts, a single successful login, and failed attempts followed by a successful login. I learned that escalation should be based on context such as source IP, user behavior, login time, repeated attempts, and post-authentication activity, not only on one log line.

## SOC English Sentences

Multiple failed SSH login attempts were observed from the same source IP.

The activity targeted several invalid usernames, which may indicate username guessing.

A single successful login requires context before it can be classified as suspicious.

The failed login attempts were followed by a successful login from the same source IP.

This pattern may indicate credential guessing followed by successful authentication.

The activity should be escalated if the successful login cannot be verified as legitimate.

Escalation should be based on suspicious context, not only on the existence of a successful login.
