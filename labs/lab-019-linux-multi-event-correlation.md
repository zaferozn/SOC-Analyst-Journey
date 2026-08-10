# Lab 019 - Linux Multi-Event Correlation

## Executive Summary

This lab investigated a connected sequence of Linux security events involving failed SSH authentication, successful remote access, privileged `sudo` activity, and file modification.

Two failed SSH authentication attempts for the `analyst` account from `192.168.64.1` were followed by a successful login to `ubuntu-agent`. The authenticated account later executed commands as `root` and created and modified a file inside a Wazuh-monitored directory.

Raw Linux logs and Wazuh alerts were correlated using timestamps, user context, source IP, privileged commands, host information, and file activity.

The activity was intentionally generated in a controlled environment and was classified as **Authorized Test Activity / Benign Positive**.

---

## Objective

The objective was to correlate multiple Linux security events into a single activity timeline.

The investigation focused on:

- Failed and successful SSH authentication
- Source IP and user correlation
- `sudo` and PAM activity
- Root-level command execution
- File creation and modification
- Wazuh authentication and FIM alerts
- Raw log versus SIEM comparison
- Timeline reconstruction
- Escalation decision

---

## Environment / Data Source

**Host:** `ubuntu-agent`  
**Agent IP:** `192.168.64.10`  
**Wazuh Manager:** `192.168.64.9`  
**Wazuh Agent ID:** `001`  
**SSH Source IP:** `192.168.64.1`  
**User:** `analyst`  
**Privileged User:** `root`

**Monitored Directory:**

```text
/home/analyst/wazuh-fim-lab
```

**Investigated File:**

```text
/home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

**Data Sources:**

- OpenSSH / `sshd`
- Linux `journalctl`
- PAM
- `sudo`
- Wazuh Threat Hunting
- Wazuh File Integrity Monitoring

---

## 1. Environment Verification

The Wazuh agent configuration confirmed real-time monitoring of the lab directory:

```xml
<directories check_all="yes" report_changes="yes" realtime="yes">/home/analyst/wazuh-fim-lab</directories>
```

### Evidence

![Lab 019 - Environment and FIM Configuration](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/01-environment-and-fim-configuration.png?raw=true)

---

## 2. Failed-to-Successful SSH Authentication

An SSH connection was initiated from:

```text
192.168.64.1
```

to:

```text
192.168.64.10
```

using the account:

```text
analyst
```

Two incorrect passwords were entered before successful authentication.

The client-side sequence showed:

```text
Failed authentication
→ Failed authentication
→ Successful authentication
```

### Evidence

![Lab 019 - Failed and Successful SSH Authentication](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/02-failed-and-successful-ssh-authentication.png?raw=true)

---

## 3. Raw SSH Evidence

The authentication activity was verified from Linux logs:

```bash
journalctl --since "10 minutes ago" --no-pager | grep -E "sshd.*(Failed password|Accepted password)"
```

Relevant events:

```text
10:09:49 UTC — Failed password for analyst from 192.168.64.1
10:09:53 UTC — Failed password for analyst from 192.168.64.1
10:10:05 UTC — Accepted password for analyst from 192.168.64.1
```

This confirmed that the same user and source IP were involved throughout the authentication sequence.

### Evidence

![Lab 019 - Raw SSH Authentication Events](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/03-raw-ssh-authentication-events.png?raw=true)

---

## 4. Privileged File Activity

After authentication, the `analyst` account used `sudo`:

```bash
sudo -k
sudo whoami
```

The command returned:

```text
root
```

A file was then created and modified with root privileges:

```bash
sudo touch /home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt

echo "Lab 019 privileged file modification" | sudo tee /home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

Verification showed:

```text
-rw-r--r-- 1 root root 37 Aug 10 10:20 /home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

This confirmed that the file was created and modified under elevated privileges.

### Evidence

![Lab 019 - Privileged File Modification](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/04-privileged-file-modification.png?raw=true)

---

## 5. Raw Sudo Evidence

The privileged commands were verified from Linux logs.

The logs recorded:

```text
analyst : USER=root ; COMMAND=/usr/bin/whoami
```

```text
analyst : USER=root ; COMMAND=/usr/bin/touch /home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

```text
analyst : USER=root ; COMMAND=/usr/bin/tee /home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

The important user relationship was:

```text
Initiating user: analyst
Effective user: root
```

### Evidence

![Lab 019 - Raw Sudo Events](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/05-raw-sudo-events.png?raw=true)

---

## 6. Wazuh Authentication Events

Wazuh independently detected the authentication activity.

Relevant alerts included:

```text
Rule 5760 — sshd: authentication failed.
Level 5
```

and:

```text
Rule 5715 — sshd: authentication success.
Level 3
```

The SIEM therefore confirmed the same failed-to-successful authentication sequence found in the raw Linux logs.

### Evidence

![Lab 019 - Wazuh Authentication Events](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/06-wazuh-authentication-events.png?raw=true)

---

## 7. Wazuh Sudo and PAM Events

Wazuh detected the privileged commands using:

```text
Rule 5402 — Successful sudo to ROOT executed.
Level 3
```

PAM session activity was also recorded:

```text
Rule 5501 — PAM: Login session opened.
Rule 5502 — PAM: Login session closed.
```

The sequence connected the authenticated session with later privileged activity:

```text
SSH authentication success
→ PAM session
→ sudo execution
→ root session activity
```

### Evidence

![Lab 019 - Wazuh Sudo Events](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/07-wazuh-sudo-events.png?raw=true)

---

## 8. Wazuh File Integrity Monitoring

Wazuh detected both creation and modification of:

```text
/home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt
```

File creation generated:

```text
Rule 554 — File added to the system.
Level 5
Event: added
```

The subsequent modification generated:

```text
Rule 550 — Integrity checksum changed.
Level 7
Event: modified
```

The file size changed from:

```text
0 bytes
```

to:

```text
37 bytes
```

The SHA-256 hash also changed after content was written.

### Evidence

![Lab 019 - Wazuh FIM Events](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-019/08-wazuh-fim-events.png?raw=true)

---

## Correlated Event Timeline

| Time (UTC) | Event | User / Entity | Evidence |
|---|---|---|---|
| 10:09:49 | SSH authentication failed | `analyst` / `192.168.64.1` | Raw SSH log |
| 10:09:53 | SSH authentication failed | `analyst` / `192.168.64.1` | Raw SSH log |
| 10:10:05 | SSH authentication succeeded | `analyst` / `192.168.64.1` | SSH + Wazuh 5715 |
| 10:10:05 | PAM session opened | `analyst` | Wazuh 5501 |
| 10:20:20 | `sudo whoami` | `analyst → root` | Sudo log |
| 10:20:20 | `sudo touch` | `analyst → root` | Sudo log |
| 10:20:20 | File added | `lab019-sensitive-file.txt` | Wazuh 554 |
| 10:20:20 | `sudo tee` | `analyst → root` | Sudo log |
| 10:20:20 | File modified | `lab019-sensitive-file.txt` | Wazuh 550 |
| 10:20:22 | Sudo alerts processed | `analyst → root` | Wazuh 5402 |

Overall sequence:

```text
Failed SSH
→ Failed SSH
→ Successful SSH
→ PAM session
→ sudo
→ root execution
→ file creation
→ file modification
→ Wazuh FIM detection
```

---

## Analysis

When viewed separately, each event could have a benign explanation.

A failed authentication may result from an incorrect password. A successful SSH login may be legitimate. `sudo` may represent normal administration, and a file modification may be authorized.

However, the events shared:

- the same user;
- the same host;
- the same SSH source IP;
- a logical chronological sequence;
- related privileged commands;
- the same affected file.

This supported treating the activity as one connected sequence rather than unrelated alerts.

The investigation also demonstrated the value of comparing raw endpoint evidence with normalized SIEM alerts.

The Linux logs showed exactly what occurred, while Wazuh provided rule IDs, severity levels, parsed users, and FIM information.

A small timestamp difference was also observed between the original sudo events and the corresponding Wazuh alerts. This demonstrates why analysts should consider both **event time** and **SIEM processing time** when building timelines.

---

## Risk

In a production environment, the following sequence could indicate suspicious post-authentication activity:

```text
Failed authentication
→ Successful access
→ Privilege elevation
→ Privileged file modification
```

Possible risks include:

- Compromised credentials
- Unauthorized remote access
- Privilege misuse
- File tampering
- Unauthorized administrative changes
- Persistence or security-control modification

The sequence alone does not prove compromise. Authorization and business context must also be reviewed.

---

## Disposition and Escalation

**Disposition: Authorized Test Activity / Benign Positive**

The alerts were valid detections and were therefore **not false positives**.

The activity itself was intentionally generated as part of the lab.

In a production environment, the sequence should be escalated if:

- the login was unexpected;
- the source IP was unauthorized;
- the user denied performing the activity;
- the sudo commands were unusual;
- the file modification was not approved.

---

## Recommended Next Steps

For an equivalent production investigation:

1. Confirm whether the user initiated the SSH session.
2. Verify whether the source IP is approved.
3. Review additional authentication history.
4. Review all sudo commands executed during the session.
5. Determine the sensitivity of the modified file.
6. Review additional FIM events.
7. Check for suspicious processes or network activity.
8. Compare the activity with approved change records.
9. Escalate if authorization cannot be confirmed.

---

## MITRE ATT&CK Mapping

| Technique | Name | Relevance |
|---|---|---|
| T1548.003 | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | The `analyst` account used `sudo` to execute commands with root privileges. |
| T1565.001 | Data Manipulation: Stored Data Manipulation | Wazuh detected modification of a monitored file and associated integrity changes. |

The mappings describe observed techniques and do not prove malicious intent.

---

## Final SOC Summary

Two failed SSH authentication attempts for the `analyst` account from `192.168.64.1` were followed by a successful login to `ubuntu-agent`. The authenticated account subsequently executed commands through `sudo`, resulting in root-level execution and the creation and modification of `/home/analyst/wazuh-fim-lab/lab019-sensitive-file.txt`.

Raw Linux logs confirmed the authentication and privileged commands, while Wazuh generated corresponding authentication, PAM, sudo, file-addition, and integrity-change alerts. The events were correlated using timestamps, user context, source IP information, host information, commands, and the affected file path.

The activity was intentionally generated and was therefore classified as **Authorized Test Activity / Benign Positive**. An equivalent unexpected sequence in production would require further investigation.

---

## Lessons Learned

This lab demonstrated that individual alerts can gain additional significance when correlated with related events.

Key lessons:

- Correlate events using timestamps and shared entities.
- Compare raw logs with SIEM alerts.
- Track both initiating and privileged users.
- Follow activity beyond successful authentication.
- Distinguish suspicious behavior from confirmed malicious activity.
- Consider authorization context before assigning a final disposition.

The central lesson was:

```text
Context + chronology + shared entities = correlation
```

---

## SOC English Sentences

> Two failed SSH authentication attempts were followed by a successful login from the same source IP.

> The authenticated user subsequently executed commands with root privileges using sudo.

> Raw Linux logs and Wazuh alerts were correlated using timestamps, user context, source IP information, and affected resources.

> File Integrity Monitoring detected both the creation and subsequent modification of the monitored file.

> The activity was classified as authorized test activity because it was intentionally generated in a controlled environment.

---

## Interview Explanation

> In this lab, I investigated several Linux security events as one connected activity sequence. Two failed SSH authentication attempts were followed by a successful login. The same account then used sudo to execute root-level commands and created and modified a monitored file. I verified the events using raw Linux logs and correlated them with Wazuh authentication, PAM, sudo, and FIM alerts. I used timestamps, source IP, user context, commands, and the affected file to reconstruct the timeline and determine whether the events were related.

---

## Evidence Summary

| Screenshot | Evidence |
|---|---|
| 01 | Environment and Wazuh FIM configuration |
| 02 | Failed-to-successful SSH authentication |
| 03 | Raw SSH authentication events |
| 04 | Privileged file creation and modification |
| 05 | Raw sudo and PAM events |
| 06 | Wazuh authentication alerts |
| 07 | Wazuh sudo and PAM activity |
| 08 | Wazuh FIM file-addition and modification alerts |
