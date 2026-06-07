# Lab 009 - Wazuh Alert Field Extraction and Raw Log Comparison

## Executive Summary

This lab documents the extraction and comparison of Wazuh authentication alert fields with raw Linux authentication logs from a monitored Ubuntu endpoint.

The objective was to understand how raw Linux authentication and session activity is represented inside Wazuh as structured SIEM alerts. Three events were reviewed: a failed SSH login attempt against a non-existent user, a successful SSH authentication for the `analyst` user, and a later sudo PAM session opened for root by the `analyst` user.

The reviewed activity was generated in a controlled lab environment. This lab was handled as an alert field extraction and raw log validation exercise, not as a confirmed security incident.

---

## Objective

The objective of this lab was to open individual Wazuh authentication alerts, extract key alert fields, and compare each Wazuh `full_log` value with the original raw Linux log entry.

The focus was on understanding how a SOC analyst validates SIEM alert evidence by comparing:

- timestamp
- agent name
- agent IP
- rule ID
- rule level
- rule description
- location
- full_log
- source IP
- username
- raw Linux log evidence

---

## Environment / Data Source

Host: ubuntu-agent  
Wazuh server: wazuh-server  
Wazuh manager IP: 192.168.64.9  
Ubuntu agent IP: 192.168.64.10  
Agent ID: 001  
Operating system: Ubuntu 24.04.4 LTS ARM64  
Tool: Wazuh  
SIEM module: Wazuh Threat Hunting / Security Events  
Log source: journald / SSH / PAM / sudo logs  

---

## Observed Activity

Three authentication-related events were reviewed in Wazuh Threat Hunting.

The observed timeline was:

```text
14:38:45 - Failed SSH login attempt for invalid user wronguser from 192.168.64.1
14:39:14 - Successful SSH authentication for analyst from 192.168.64.1
14:42:26 - Local sudo PAM session opened for root by analyst
```

The events showed how Wazuh represents remote SSH authentication activity and local privilege-related sudo session activity.

---

## Evidence

### Command Used for Raw Log Review

The raw logs were reviewed on the monitored Ubuntu endpoint, not on the Wazuh server.

```bash
sudo journalctl --since "Jun 05 14:35:00" --until "Jun 05 14:45:00" | grep -Ei "sshd|sudo|pam_unix|failed password|accepted password|invalid user|session opened|authentication failure"
```

---

## Event 1 - Failed SSH Login Using Non-Existent User

### Wazuh Alert Fields

| Field | Value |
|---|---|
| timestamp | Jun 05 14:38:45 |
| agent.name | ubuntu-agent |
| agent.ip | 192.168.64.10 |
| rule.id | 5710 |
| rule.level | 5 |
| rule.description | sshd: Attempt to login using non-existent user |
| location | journald |
| full_log | Jun 05 14:38:45 ubuntu-agent sshd[4503]: Failed password for invalid user wronguser from 192.168.64.1 port 53014 ssh2 |
| source IP | 192.168.64.1 / extracted from full_log |
| username | wronguser / shown as srcuser |

### Matching Raw Log Entry

```text
Jun 05 14:38:45 ubuntu-agent sshd[4503]: Failed password for invalid user wronguser from 192.168.64.1 port 53014 ssh2
```

### Field Comparison

| Wazuh Field | Raw Log Evidence | Match |
|---|---|---|
| timestamp | Jun 05 14:38:45 | Yes |
| agent.name | ubuntu-agent | Yes |
| process | sshd[4503] | Yes |
| username | wronguser | Yes |
| source IP | 192.168.64.1 | Yes |
| source port | 53014 | Yes |
| event type | Failed password for invalid user | Yes |

### Analysis

This Wazuh alert directly matched the raw SSH log entry at `Jun 05 14:38:45`.

The event showed a failed SSH login attempt against a non-existent user account named `wronguser`. The source IP address was `192.168.64.1`, and the source port was `53014`.

The source IP was not clearly observed as a separate Wazuh field during review, but it was visible in the `full_log` value and was extracted from the raw log text.

This event is suspicious because login attempts against invalid or non-existent users may indicate user enumeration, scanning, brute-force activity, or manual testing.

---

## Event 2 - Successful SSH Authentication

### Wazuh Alert Fields

| Field | Value |
|---|---|
| timestamp | Jun 05 14:39:14 |
| agent.name | ubuntu-agent |
| agent.ip | 192.168.64.10 |
| rule.id | 5715 |
| rule.level | 3 |
| rule.description | sshd: authentication success. |
| location | journald |
| full_log | Jun 05 14:39:14 ubuntu-agent sshd[4516]: Accepted password for analyst from 192.168.64.1 port 53015 ssh2 |
| source IP | 192.168.64.1 |
| username | analyst / shown as data.dstuser |

### Matching Raw Log Entry

```text
Jun 05 14:39:14 ubuntu-agent sshd[4516]: Accepted password for analyst from 192.168.64.1 port 53015 ssh2
```

### Field Comparison

| Wazuh Field | Raw Log Evidence | Match |
|---|---|---|
| timestamp | Jun 05 14:39:14 | Yes |
| agent.name | ubuntu-agent | Yes |
| process | sshd[4516] | Yes |
| username | analyst | Yes |
| source IP | 192.168.64.1 | Yes |
| source port | 53015 | Yes |
| event type | Accepted password | Yes |

### Analysis

This Wazuh alert directly matched the raw SSH log entry at `Jun 05 14:39:14`.

The event confirmed a successful SSH authentication for the user `analyst` from the source IP address `192.168.64.1`.

A successful login is not automatically malicious. However, it should be reviewed in context because it occurred shortly after failed authentication activity from the same source IP address.

In a real SOC environment, the analyst should determine whether:

- `192.168.64.1` is an expected source IP.
- `analyst` is an expected SSH user.
- The successful login occurred during an approved time window.
- The failed invalid-user activity was part of expected testing.
- The successful login followed suspicious authentication attempts.

---

## Event 3 - Local Sudo PAM Session Opened

### Wazuh Alert Fields

| Field | Value |
|---|---|
| timestamp | Jun 05 14:42:26 |
| agent.name | ubuntu-agent |
| agent.ip | 192.168.64.10 |
| rule.id | 5501 |
| rule.level | 3 |
| rule.description | PAM: Login session opened. |
| location | journald |
| full_log | Jun 05 14:42:26 ubuntu-agent sudo[4545]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000) |
| source IP | not applicable / local sudo session |
| username | analyst / sudo session opened for root by analyst |

### Matching Raw Log Entry

```text
Jun 05 14:42:26 ubuntu-agent sudo[4545]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
```

### Related Raw Sudo Command Log

```text
Jun 05 14:42:26 ubuntu-agent sudo[4545]:  analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/journalctl -u ssh --since '10 minutes ago'
```

### Field Comparison

| Wazuh Field | Raw Log Evidence | Match |
|---|---|---|
| timestamp | Jun 05 14:42:26 | Yes |
| agent.name | ubuntu-agent | Yes |
| process | sudo[4545] | Yes |
| initiating user | analyst | Yes |
| target user | root | Yes |
| event type | sudo PAM session opened | Yes |
| source IP | not applicable | Yes |

### Analysis

This Wazuh alert matched the raw sudo PAM session log at `Jun 05 14:42:26`.

This was not a remote SSH login event. It was a local sudo PAM session event. The user `analyst` had already authenticated to the host and then executed a command with sudo privileges.

The exact command was not visible in the selected PAM session alert because the PAM session log only confirmed that a root-level sudo session was opened. The command was visible in a related sudo log entry with the same timestamp and process ID `sudo[4545]`.

The executed command was:

```bash
/usr/bin/journalctl -u ssh --since '10 minutes ago'
```

This means the `analyst` user executed the following command with root privileges:

```bash
sudo journalctl -u ssh --since '10 minutes ago'
```

The sudo command event and the PAM session event should be reviewed together:

```text
COMMAND log line = shows what was executed
PAM session opened log line = confirms that a root-level sudo session was opened
```

No source IP was expected in this event because it was not a new network login. The activity occurred locally on the host after authentication.

---

## Raw Log Timeline

The following raw Linux log entries show the full activity sequence observed on the `ubuntu-agent` host.

```text
Jun 05 14:38:28 ubuntu-agent sshd[4503]: Invalid user wronguser from 192.168.64.1 port 53014
Jun 05 14:38:30 ubuntu-agent sshd[4503]: pam_unix(sshd:auth): check pass; user unknown
Jun 05 14:38:30 ubuntu-agent sshd[4503]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.1
Jun 05 14:38:33 ubuntu-agent sshd[4503]: Failed password for invalid user wronguser from 192.168.64.1 port 53014 ssh2
Jun 05 14:38:38 ubuntu-agent sshd[4503]: pam_unix(sshd:auth): check pass; user unknown
Jun 05 14:38:40 ubuntu-agent sshd[4503]: Failed password for invalid user wronguser from 192.168.64.1 port 53014 ssh2
Jun 05 14:38:42 ubuntu-agent sshd[4503]: pam_unix(sshd:auth): check pass; user unknown
Jun 05 14:38:45 ubuntu-agent sshd[4503]: Failed password for invalid user wronguser from 192.168.64.1 port 53014 ssh2
Jun 05 14:38:46 ubuntu-agent sshd[4503]: Connection closed by invalid user wronguser 192.168.64.1 port 53014 [preauth]
Jun 05 14:38:46 ubuntu-agent sshd[4503]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.64.1
Jun 05 14:39:14 ubuntu-agent sshd[4516]: Accepted password for analyst from 192.168.64.1 port 53015 ssh2
Jun 05 14:39:14 ubuntu-agent sshd[4516]: pam_unix(sshd:session): session opened for user analyst(uid=1000) by analyst(uid=0)
Jun 05 14:42:26 ubuntu-agent sudo[4545]:  analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/journalctl -u ssh --since '10 minutes ago'
Jun 05 14:42:26 ubuntu-agent sudo[4545]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
Jun 05 14:42:27 ubuntu-agent sudo[4545]: pam_unix(sudo:session): session closed for user root
```

---

## Raw Log vs Wazuh Alert Mapping

| Wazuh Alert | Matching Raw Log | Meaning |
|---|---|---|
| Rule 5710 - Attempt to login using non-existent user | `Failed password for invalid user wronguser from 192.168.64.1` | Failed SSH login against invalid user |
| Rule 5715 - authentication success | `Accepted password for analyst from 192.168.64.1` | Successful SSH login |
| Rule 5501 - PAM login session opened | `pam_unix(sudo:session): session opened for user root by analyst` | Local sudo/root session opened |

---

## Analysis

The raw Linux logs confirmed the authentication and session activity observed in Wazuh.

The first Wazuh alert showed a failed SSH login attempt for a non-existent user. The matching raw log confirmed the invalid username `wronguser`, the source IP address `192.168.64.1`, and the SSH source port `53014`.

The second Wazuh alert showed a successful SSH authentication for the user `analyst`. The matching raw log confirmed that the successful login came from the same source IP address, `192.168.64.1`, using source port `53015`.

The third Wazuh alert showed a local sudo PAM session opened for root by `analyst`. This was not a remote SSH login event. It represented local privilege-related activity after the user had already authenticated to the host.

The exact sudo command was visible in a related sudo log entry:

```text
COMMAND=/usr/bin/journalctl -u ssh --since '10 minutes ago'
```

The selected Wazuh PAM session alert did not show the executed command because it represented only the session-opened log line. The command was identified by correlating the related sudo log entry using the same timestamp and process ID `sudo[4545]`.

---

## Risk

This activity was generated in a controlled lab environment and was not treated as a confirmed compromise.

However, in a real SOC environment, the sequence would require review because it includes:

- failed SSH attempts against an invalid user
- a successful SSH login shortly after failed attempts
- activity from the same source IP address
- a later sudo/root session by the authenticated user

This pattern may be benign if it was performed by an administrator or tester. It may be suspicious if the source IP, username, time window, or sudo command is unexpected.

---

## Recommended Next Steps

A SOC analyst should review the following points:

- Confirm whether `192.168.64.1` is an expected source IP.
- Confirm whether the `analyst` user is allowed to access the host over SSH.
- Check whether the invalid username `wronguser` was part of approved testing.
- Review whether the successful login occurred after repeated failed attempts.
- Review the exact sudo command executed by the user.
- Determine whether the sudo activity was expected administrative behavior.
- Escalate if the login source, username, timing, or sudo command is not expected.

---

## MITRE ATT&CK Mapping

Mapping should not be forced unless suspicious behavior is confirmed.

Possible relevant mappings:

- T1110 - Brute Force  
  Relevant if repeated failed SSH authentication attempts are confirmed as unauthorized brute-force activity.

- T1078 - Valid Accounts  
  Relevant if the successful SSH authentication is confirmed to involve unauthorized or compromised credentials.

- T1548 - Abuse Elevation Control Mechanism  
  Relevant if sudo usage is confirmed as suspicious or unauthorized privilege escalation activity.

---

## Final SOC Summary

Three Wazuh authentication and session alerts were reviewed and compared with raw Linux logs from the `ubuntu-agent` host. The first alert matched a failed SSH login attempt for the invalid user `wronguser` from `192.168.64.1`. The second alert matched a successful SSH authentication for the user `analyst` from the same source IP address. The third alert matched a local sudo PAM session opened for root by the `analyst` user.

The raw logs confirmed the Wazuh alert evidence through matching timestamps, hostnames, process IDs, usernames, source IPs, rule descriptions, and `full_log` content. No confirmed compromise was identified in this lab environment, but the activity sequence demonstrated how authentication alerts should be reviewed in context before making an escalation decision.

---

## Lessons Learned

- Wazuh `full_log` can be matched directly with raw Linux log entries.
- A Wazuh alert should be validated against the original raw log when needed.
- The source IP may be visible in `full_log` even if it is not clearly visible as a separate parsed field.
- Remote SSH login events usually contain source IP information.
- Local sudo PAM session events do not normally contain a source IP because the activity occurs after login.
- The PAM session line confirms that a session was opened, but it does not always show the exact command.
- The exact sudo command can be found in the related sudo log entry containing the `COMMAND` field.
- Timestamp and process ID can be used to correlate related sudo and PAM log entries.
- Authentication events should be reviewed as a timeline, not as isolated alerts.

---

## SOC English Sentences

```text
The Wazuh full_log field matched the original raw Linux log entry.

The timestamp, hostname, process ID, username, and source IP were used to validate the match.

The invalid user alert matched the raw SSH failed password log.

The successful authentication alert matched the raw SSH accepted password log.

The PAM session alert matched the raw sudo session opened log.

The sudo session did not include a source IP because it was a local privilege-related event.

The exact command was identified from the related sudo log entry.

The COMMAND field showed that the analyst user executed journalctl with sudo privileges.

The event timeline showed failed authentication activity followed by a successful login and a later sudo session.

No escalation decision was made before validating the raw log evidence.
```
