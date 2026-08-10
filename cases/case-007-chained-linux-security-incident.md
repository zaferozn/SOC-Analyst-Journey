# Case 007 - Chained Linux Security Incident

## Executive Summary

This case documents the reconstruction of a controlled Linux security incident involving repeated SSH authentication failures, a subsequent successful login, PAM session establishment, privileged `sudo` activity, root-level command execution, modification of a security-sensitive SSH configuration file, and detection by Wazuh File Integrity Monitoring (FIM).

The investigation correlated raw Linux authentication logs, PAM events, `sudo` activity, root-level execution, and Wazuh FIM telemetry instead of treating each event as an isolated alert.

Three failed SSH password attempts against the `analyst` account from `192.168.64.1` were followed approximately two minutes later by successful authentication from the same source IP.

After the SSH session was established, the `analyst` account executed commands with root privileges through `sudo`. A root-level shell command later modified `/etc/ssh/sshd_config`.

Wazuh independently detected the file modification through realtime File Integrity Monitoring and generated Rule `550`, confirming an integrity change to the monitored file.

Because this activity was intentionally generated in a controlled lab environment, the final disposition is:

**True Positive - Simulated Security Incident**

If the same sequence occurred unexpectedly in production, escalation would be justified because the combination of failed-to-successful authentication, privileged execution, and modification of a sensitive SSH configuration file could indicate account compromise and unauthorized administrative activity.

---

## Objective

The objective of this case was to reconstruct a multi-stage Linux security incident from multiple telemetry sources and determine whether the observed events represented unrelated activity or a connected security sequence.

The investigation focused on:

- Repeated SSH authentication failures
- Failed-to-successful authentication transition
- Source IP correlation
- User account correlation
- PAM session establishment
- `sudo` activity
- Root-level command execution
- Failed and successful privileged authentication
- Security-sensitive file modification
- Wazuh File Integrity Monitoring
- Raw Linux journal analysis
- Timestamp correlation
- Attempted versus successful activity
- Risk assessment
- Incident disposition
- Escalation decision

---

## Environment / Data Source

**Host:** `ubuntu-agent`  
**Target IP:** `192.168.64.10`  
**Wazuh Agent ID:** `001`  
**Source IP:** `192.168.64.1`  
**User:** `analyst`  
**Operating System:** Ubuntu Linux  
**SIEM / Monitoring Tool:** Wazuh  
**Authentication Service:** OpenSSH  
**Privilege Mechanism:** `sudo`  
**Sensitive File:** `/etc/ssh/sshd_config`

### Data Sources

- Linux `systemd` journal
- OpenSSH authentication events
- PAM authentication and session events
- `sudo` activity
- Wazuh alerts
- Wazuh File Integrity Monitoring
- Analyst-created incident timeline

### Investigation Window

Primary host activity occurred between approximately:

`2026-08-10 15:59:54 UTC`

and

`2026-08-10 16:12:48 UTC`

The Wazuh dashboard displayed the FIM event using a different local timezone presentation. Timestamp normalization was therefore required when correlating host and SIEM telemetry.

---

## Observed Activity

### 1. Repeated SSH Authentication Failures

Multiple failed SSH password attempts were observed against the `analyst` account from `192.168.64.1`.

Relevant Linux telemetry included:

```text
Aug 10 15:59:54 ubuntu-agent sshd[...] pam_unix(sshd:auth): authentication failure ... rhost=192.168.64.1 user=analyst

Aug 10 15:59:56 ubuntu-agent sshd[...] Failed password for analyst from 192.168.64.1 port 50341 ssh2

Aug 10 16:00:00 ubuntu-agent sshd[...] Failed password for analyst from 192.168.64.1 port 50341 ssh2

Aug 10 16:00:04 ubuntu-agent sshd[...] Failed password for analyst from 192.168.64.1 port 50341 ssh2
```

Observed context:

- Source IP: `192.168.64.1`
- Target account: `analyst`
- Target host: `ubuntu-agent`
- Service: SSH
- Failed password attempts: 3
- Outcome: Authentication failed

![Failed SSH Authentication Attempts](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/01-failed-ssh-attempts.png?raw=true)

---

### 2. Successful SSH Authentication

Approximately two minutes after the failed attempts, the same source IP successfully authenticated to the same account.

Raw Linux evidence:

```text
Aug 10 16:02:44 ubuntu-agent sshd[...] Accepted password for analyst from 192.168.64.1 port 50391 ssh2
```

Client-side verification confirmed:

```text
User: analyst
Host: ubuntu-agent
Source IP: 192.168.64.1
```

![Successful SSH Login](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/02-successful-ssh-login.png?raw=true)

The failed-to-successful transition increased the significance of the authentication activity because the same source IP and target account were involved.

---

### 3. PAM Session Established

Immediately after successful authentication, PAM recorded the creation of a user session.

```text
Aug 10 16:02:44 ubuntu-agent sshd[...] pam_unix(sshd:session): session opened for user analyst(uid=1000)
```

This confirmed that the authentication event resulted in an established interactive SSH session.

---

### 4. Privileged Activity Through Sudo

After the SSH session was established, the `analyst` account invoked `sudo`.

Evidence included:

```text
analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/whoami
```

The command returned:

```text
root
```

This demonstrated successful execution under root privileges.

![Sudo Privilege Activity](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/03-sudo-privilege-activity.png?raw=true)

The evidence does not indicate exploitation of a software vulnerability. The `analyst` account already possessed authorized `sudo` capability.

---

### 5. Failed Sudo Authentication Attempt

During the same session, one `sudo` authentication attempt failed.

```text
Aug 10 16:07:11 ubuntu-agent sudo[...] pam_unix(sudo:auth): authentication failure ... user=analyst
```

A subsequent privileged command succeeded.

This distinction is important because the evidence contains both:

- Attempted privileged authentication
- Successful privileged execution

The failed attempt was preserved as part of the timeline rather than being treated as equivalent to successful privilege use.

---

### 6. Root-Level Modification of SSH Configuration

At approximately `16:12:48 UTC`, the `analyst` account executed a root-level shell command through `sudo`.

The command modified:

```text
/etc/ssh/sshd_config
```

Raw telemetry showed:

```text
analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/sh -c 'echo "# CASE007 simulated unauthorized configuration change ..." >> /etc/ssh/sshd_config'
```

The modification was intentionally limited to adding a comment so the exercise would generate a security-relevant integrity event without changing SSH authentication behavior.

![Sensitive File Modification](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/04-sensitive-file-modification.png?raw=true)

The activity established the following relationship:

```text
analyst
    ↓
sudo
    ↓
root execution
    ↓
/etc/ssh/sshd_config modification
```

---

### 7. Wazuh File Integrity Monitoring Detection

Wazuh independently detected the modification through File Integrity Monitoring.

Observed fields included:

```text
Agent: ubuntu-agent
Agent ID: 001
Path: /etc/ssh/sshd_config
Event: modified
Rule ID: 550
Rule Level: 7
Description: Integrity checksum changed
```

![Wazuh FIM Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/05-wazuh-fim-alert.png?raw=true)

This provided independent SIEM evidence that the monitored file had changed.

---

## Evidence

### Evidence 1 - Failed SSH Authentication

Three failed SSH password attempts targeted the same account from the same source IP.

```text
User: analyst
Source IP: 192.168.64.1
Target Host: ubuntu-agent
Service: SSH
```

![Failed SSH Attempts](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/01-failed-ssh-attempts.png?raw=true)

---

### Evidence 2 - Successful SSH Authentication

The same source IP subsequently authenticated successfully to the same account.

```text
Accepted password for analyst from 192.168.64.1
```

![Successful SSH Authentication](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/02-successful-ssh-login.png?raw=true)

---

### Evidence 3 - Privileged Execution

The authenticated account successfully executed a command as root through `sudo`.

```text
USER=root
COMMAND=/usr/bin/whoami
```

![Privileged Execution](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/03-sudo-privilege-activity.png?raw=true)

---

### Evidence 4 - Sensitive Configuration Modification

A root-level command modified:

```text
/etc/ssh/sshd_config
```

![SSH Configuration Modification](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/04-sensitive-file-modification.png?raw=true)

---

### Evidence 5 - Wazuh FIM Alert

Wazuh recorded the change as a File Integrity Monitoring event.

```text
Rule ID: 550
Rule Level: 7
Event: modified
Path: /etc/ssh/sshd_config
```

![Wazuh File Integrity Monitoring Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/05-wazuh-fim-alert.png?raw=true)

---

### Evidence 6 - Raw Linux Timeline

Linux journal telemetry was reviewed to correlate:

- SSH authentication failures
- SSH authentication success
- PAM session establishment
- Failed `sudo` authentication
- Successful `sudo` execution
- Root-level command execution
- Sensitive file modification

![Raw Linux Timeline](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/06-raw-linux-timeline.png?raw=true)

---

### Evidence 7 - Analyst Incident Reconstruction

The raw events were reconstructed into a chronological analyst timeline.

![Final Incident Timeline Part 1](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/07-final-incident-timeline-1.png?raw=true)

![Final Incident Timeline Part 2](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-007/07-final-incident-timeline-2.png?raw=true)

---

## Incident Timeline

| Time (UTC) | Event | Evidence / Context |
|---|---|---|
| 15:59:54 | Initial SSH authentication failure | `analyst` targeted from `192.168.64.1` |
| 15:59:56 | Failed SSH password | Same source and account |
| 16:00:00 | Failed SSH password | Same source and account |
| 16:00:04 | Failed SSH password | Third failed attempt |
| 16:00:06 | Authentication attempt terminated | SSH connection closed |
| 16:02:44 | Successful SSH authentication | `Accepted password for analyst from 192.168.64.1` |
| 16:02:44 | PAM session opened | Interactive session established |
| 16:05:39 | Successful `sudo` execution | `analyst` executed `/usr/bin/whoami` as root |
| 16:07:11 | Failed `sudo` authentication | Incorrect privileged authentication attempt |
| 16:07:25 | Successful `sudo` execution | Privileged command subsequently succeeded |
| 16:12:48 | Sensitive file modification | `/etc/ssh/sshd_config` modified through root-level execution |
| Wazuh dashboard timestamp | FIM detection | Rule `550`, Level `7`, event `modified` |

---

## Analysis

### What Was Observed?

The investigation identified a connected sequence of authentication, session, privilege, and file-integrity events occurring within the same host context.

The activity began with repeated SSH authentication failures against the `analyst` account.

The same source IP subsequently authenticated successfully.

An interactive PAM session was established.

The authenticated account then used `sudo` to execute commands with root privileges.

A root-level shell command later modified `/etc/ssh/sshd_config`.

Wazuh File Integrity Monitoring independently detected the resulting file integrity change.

---

### Where Was It Observed?

The activity was observed on:

```text
Host: ubuntu-agent
Target IP: 192.168.64.10
Wazuh Agent ID: 001
```

Telemetry sources included:

```text
Linux systemd journal
OpenSSH
PAM
sudo
Wazuh
Wazuh File Integrity Monitoring
```

---

### Which User, Host, IP Address, or File Was Involved?

```text
Source IP:
192.168.64.1

Target Host:
ubuntu-agent

Target Account:
analyst

Privileged Context:
root

Sensitive File:
/etc/ssh/sshd_config
```

---

### Why Is It Suspicious?

An individual failed SSH authentication attempt is common and may be benign.

However, the risk changes when the events are correlated:

```text
Repeated failed SSH attempts
        ↓
Successful authentication
        ↓
Interactive session
        ↓
sudo activity
        ↓
root execution
        ↓
Sensitive configuration modification
        ↓
Wazuh FIM detection
```

In a production environment, this sequence could indicate that an unauthorized actor:

1. Attempted to authenticate to an account.
2. Successfully gained access.
3. Established an interactive session.
4. Used administrative privileges.
5. Modified security-relevant system configuration.

The evidence in this case was intentionally generated in a controlled laboratory environment, so malicious intent is not inferred from the simulation itself.

---

### What Evidence Supports the Assessment?

The assessment is supported by multiple telemetry sources:

1. `sshd` recorded repeated authentication failures.
2. `sshd` recorded successful authentication from the same source IP.
3. PAM recorded an opened session for `analyst`.
4. `sudo` recorded root-level commands executed by `analyst`.
5. `sudo` recorded a failed privileged authentication attempt.
6. `sudo` recorded the command that modified `/etc/ssh/sshd_config`.
7. Wazuh FIM independently detected the resulting integrity change.

The combination of these sources provides stronger evidence than any individual event considered alone.

---

### Attempted vs Successful Activity

The incident contained several different outcomes that must not be treated as equivalent.

#### Failed SSH Authentication

The initial password attempts were unsuccessful.

#### Successful SSH Authentication

Authentication later succeeded from the same source IP against the same account.

#### Failed Sudo Authentication

One privileged authentication attempt failed.

#### Successful Privileged Execution

The account subsequently executed commands successfully as root.

This distinction is important because SOC analysis should identify whether an activity was merely attempted or actually succeeded.

---

### Alternative Explanation

Each activity could potentially have a legitimate administrative explanation.

For example:

- A user may enter an incorrect SSH password several times.
- The same user may then authenticate successfully.
- An administrator may legitimately use `sudo`.
- SSH configuration may be intentionally updated during maintenance.

Therefore, a real-world investigation should not classify the sequence as malicious solely because these events occurred.

Additional context would be required, including:

- Whether `192.168.64.1` is an authorized administrative source
- Whether the user normally connects from this IP
- Whether the account owner confirms the session
- Whether the configuration modification was authorized
- Whether an approved change ticket exists
- What additional commands were executed
- Whether persistence or lateral movement occurred
- Whether additional sensitive files were modified

In this controlled scenario, the events were intentionally generated as a chained incident.

---

## Risk

If the same sequence occurred unexpectedly in a production environment, the potential risk would be **High**.

Possible impacts include:

- Compromised user credentials
- Unauthorized remote access
- Abuse of administrative privileges
- Unauthorized SSH configuration changes
- Weakening of authentication controls
- Persistence establishment
- Additional root-level modification
- Expansion of attacker control over the host

Modification of `/etc/ssh/sshd_config` is particularly security-relevant because configuration changes can affect remote access and authentication behavior.

In this simulation, only a comment was added and SSH functionality was not changed.

---

## Disposition

**Disposition:** True Positive - Simulated Security Incident

The underlying authentication, session, privilege, and file modification events genuinely occurred and were successfully recorded by Linux and Wazuh telemetry.

The activity is therefore not a false positive.

The malicious context was simulated intentionally for the laboratory exercise.

---

## Escalation Decision

**Escalation: Yes - if observed unexpectedly in production.**

The failed authentication attempts alone would not necessarily justify escalation.

However, the correlated sequence materially increases the risk:

```text
Failed authentication
        ↓
Successful authentication
        ↓
Interactive session
        ↓
Privileged execution
        ↓
Sensitive system modification
```

A production SOC analyst should escalate this activity if the successful login, privileged activity, or file modification cannot be validated as authorized.

---

## Recommended Next Steps

1. Validate whether `192.168.64.1` is an authorized administrative source.

2. Contact the owner of the `analyst` account and verify whether the SSH session was expected.

3. Review the account's complete authentication history.

4. Review all commands executed during the SSH session.

5. Compare `/etc/ssh/sshd_config` with the approved baseline.

6. Identify the exact configuration changes.

7. Determine whether the modification was associated with an approved maintenance or change-management ticket.

8. Review additional Wazuh FIM alerts generated during the same time window.

9. Search for additional privileged modifications.

10. Review persistence mechanisms.

11. Review outbound network connections following the configuration change.

12. Review other users and authentication activity originating from the same source IP.

13. Reset credentials if account compromise cannot be ruled out.

14. Isolate the host if evidence indicates active compromise.

---

## MITRE ATT&CK Mapping

### T1110.001 - Brute Force: Password Guessing

Repeated failed password attempts were generated against the `analyst` SSH account.

In this controlled scenario, the activity simulated password guessing behavior.

---

### T1078 - Valid Accounts

The activity progressed to successful authentication using the legitimate `analyst` account.

In a real compromise, successful access following repeated password attempts could indicate use of valid account credentials.

---

### T1548.003 - Abuse Elevation Control Mechanism: Sudo and Sudo Caching

The authenticated `analyst` account used `sudo` to execute commands with root privileges.

No software vulnerability exploitation was observed.

The account already possessed authorized `sudo` capability.

---

### File Modification

The modification of `/etc/ssh/sshd_config` was security-relevant, but no additional MITRE ATT&CK technique is assigned solely because a file was modified.

The simulated modification did not alter SSH functionality, so forcing an additional mapping would overstate the available evidence.

---

## Final SOC Summary

Multiple failed SSH authentication attempts against the `analyst` account from `192.168.64.1` were followed by successful SSH authentication from the same source IP.

PAM telemetry confirmed that an interactive session was established.

Following authentication, the `analyst` account executed commands through `sudo` with root privileges. One subsequent `sudo` authentication attempt failed before privileged activity again succeeded.

A root-level shell command later modified `/etc/ssh/sshd_config`, a security-sensitive system configuration file.

Wazuh File Integrity Monitoring independently detected the change and generated Rule `550` with a `modified` event.

Correlation of SSH, PAM, `sudo`, Linux journal, and Wazuh FIM telemetry established that the events formed a connected activity sequence rather than unrelated alerts.

The final disposition is:

**True Positive - Simulated Security Incident**

If the same sequence occurred unexpectedly in a production environment, escalation would be recommended due to the combination of failed-to-successful authentication, privileged execution, and modification of a sensitive system configuration file.

---

## Lessons Learned

This case demonstrated that incident investigation requires correlation rather than isolated alert review.

Key lessons included:

- A failed login alone provides limited context.
- Repeated failures become more meaningful when followed by successful authentication from the same source.
- Successful authentication does not automatically prove malicious access.
- PAM session events confirm that authentication resulted in an established session.
- `sudo` logs provide attribution between the original user and root-level execution.
- Attempted and successful privileged activity should be distinguished.
- Raw Linux logs can reconstruct activity behind SIEM alerts.
- Wazuh FIM provides independent evidence of security-sensitive file changes.
- Timestamp normalization is essential when host logs and SIEM dashboards use different timezone presentations.
- Multiple low-level events can collectively represent a higher-risk incident.
- Analyst conclusions should separate observed facts from investigative inference.
- Alternative benign explanations should be considered.
- MITRE ATT&CK techniques should only be mapped when supported by the available evidence.
- Escalation decisions should be based on the complete activity chain rather than an individual alert.

The primary outcome of this case was learning to reconstruct:

```text
Failed SSH attempts
        ↓
Successful authentication
        ↓
PAM session establishment
        ↓
Privilege activity
        ↓
Root execution
        ↓
Sensitive file modification
        ↓
Wazuh FIM detection
        ↓
Multi-event correlation
        ↓
Risk assessment
        ↓
Escalation decision
```

rather than investigating each security event independently.
