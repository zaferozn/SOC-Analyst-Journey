# Lab 018 - Linux Privileged Activity and Sudo Analysis

## Executive Summary

This lab investigated privileged Linux activity generated through `sudo` on the monitored Ubuntu endpoint.

Controlled administrative commands were executed by the `analyst` account and reviewed using both the Linux systemd journal and Wazuh SIEM.

The investigation confirmed that Wazuh successfully identified:

- successful sudo execution;
- the initiating user;
- the target privileged user;
- the executed command;
- PAM root-session activity;
- a failed sudo authentication attempt.

A successful sudo event generated Wazuh rule `5402` at level `3`, while an intentionally failed sudo attempt generated rule `5401` at level `5`.

The activity was intentionally generated in a controlled lab environment. The final disposition was **Benign Positive / Authorized Test Activity**.

---

## Objective

The objective of this lab was to investigate Linux privileged activity and determine how successful and failed sudo activity appears in both raw endpoint logs and Wazuh SIEM alerts.

The investigation focused on:

- identifying the initiating user;
- identifying the target privileged account;
- reviewing commands executed through sudo;
- identifying root-session activity;
- comparing successful and failed privilege attempts;
- correlating raw Linux evidence with Wazuh alerts;
- determining whether the activity required escalation.

---

## Environment / Data Source

Host: `ubuntu-agent`

User: `analyst`

Target privileged user: `root`

Operating System: Ubuntu Linux

SIEM: Wazuh

Log source: systemd journal / journald

Wazuh data source: Threat Hunting

Relevant Wazuh Rules:

- `5402` — Successful sudo to ROOT executed.
- `5401` — Failed attempt to run sudo.
- `5501` — PAM: Login session opened.

---

## Controlled Activity

The following commands were executed to generate privileged activity:

```bash
sudo whoami
sudo ls /root
sudo touch /root/lab018-test.txt
sudo ls -l /root/lab018-test.txt
```

The created file was verified with:

```text
-rw-r--r-- 1 root root 0 Aug 9 16:26 /root/lab018-test.txt
```

This confirmed that the file was created with root ownership.

---

## Raw Linux Log Investigation

The expected `/var/log/auth.log` file was not available on the endpoint:

```bash
sudo grep "sudo" /var/log/auth.log | tail -n 30
```

Result:

```text
grep: /var/log/auth.log: No such file or directory
```

The investigation therefore used the systemd journal:

```bash
sudo journalctl _COMM=sudo --since "15 minutes ago" --no-pager
```

Relevant events included:

```text
analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/whoami

analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/ls /root

analyst : TTY=pts/1 ; PWD=/home/analyst ; USER=root ; COMMAND=/usr/bin/touch /root/lab018-test.txt
```

The journal also recorded PAM session activity:

```text
pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
```

### Evidence

![Raw sudo log evidence](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-018/01-sudo-raw-log-evidence.png?raw=true)

### Interpretation

The raw endpoint evidence showed that:

- `analyst` initiated the sudo activity;
- the working directory was `/home/analyst`;
- the target user was `root`;
- privileged commands were successfully executed;
- `/root/lab018-test.txt` was created with root privileges;
- PAM opened root sessions associated with the sudo activity.

---

## Wazuh Successful Sudo Detection

The corresponding activity was identified in Wazuh Threat Hunting.

For the file-creation event, Wazuh extracted:

```text
agent.name:   ubuntu-agent
data.command: /usr/bin/touch /root/lab018-test.txt
data.dstuser: root
data.pwd:     /home/analyst
data.srcuser: analyst
data.tty:     pts/1
```

The alert was classified as:

```text
Rule ID: 5402
Rule Level: 3
Rule Description: Successful sudo to ROOT executed.
```

### Evidence

![Wazuh successful sudo alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-018/02-wazuh-sudo-alert.png?raw=true)

### Analysis

The SIEM alert correlated directly with the raw Linux journal event.

The raw event showed:

```text
analyst
→ sudo
→ USER=root
→ COMMAND=/usr/bin/touch /root/lab018-test.txt
```

Wazuh parsed the same activity into structured fields, allowing the analyst to identify the source user, destination user, command, working directory, host, and detection rule more efficiently.

---

## PAM Root Session Analysis

Wazuh also detected the PAM session associated with the privileged operation.

Relevant fields included:

```text
agent.name: ubuntu-agent
data.srcuser: analyst
data.dstuser: root
data.uid: 1000
```

The raw log contained:

```text
session opened for user root(uid=0) by analyst(uid=1000)
```

Wazuh classified the event as:

```text
Rule ID: 5501
Rule Level: 3
Rule Description: PAM: Login session opened.
```

### Evidence

![PAM root session](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-018/03-pam-root-session.png?raw=true)

### Interpretation

The PAM event provided additional authentication context.

It confirmed that:

```text
analyst(uid=1000)
→ sudo
→ root(uid=0) session opened
```

The PAM event did not independently prove malicious activity. It provided supporting evidence that a privileged root session was created as part of the sudo operation.

---

## Failed Sudo Authentication Test

A failed privilege attempt was intentionally generated using:

```bash
sudo -k
sudo whoami
```

An incorrect password was entered.

The terminal returned:

```text
Sorry, try again.
sudo: 1 incorrect password attempt
```

The systemd journal showed:

```text
pam_unix(sudo:auth): authentication failure;
logname=analyst uid=1000 euid=0 tty=/dev/pts/1
ruser=analyst user=analyst
```

It also recorded:

```text
analyst : 1 incorrect password attempt ;
TTY=pts/1 ;
PWD=/home/analyst ;
USER=root ;
COMMAND=/usr/bin/whoami
```

### Evidence

![Failed sudo raw evidence](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-018/04-failed-sudo-raw-evidence.png?raw=true)

### Interpretation

Unlike the successful sudo events, authentication was not completed successfully.

The event showed:

```text
Source user: analyst
Target user: root
Requested command: /usr/bin/whoami
```

The incorrect password prevented the requested privileged execution.

---

## Wazuh Failed Sudo Detection

Wazuh detected the failed sudo attempt and extracted the relevant context.

The alert contained:

```text
data.srcuser: analyst
data.dstuser: root
data.command: /usr/bin/whoami
data.pwd: /home/analyst
data.tty: pts/1
```

The full log recorded:

```text
analyst : 1 incorrect password attempt ;
TTY=pts/1 ;
PWD=/home/analyst ;
USER=root ;
COMMAND=/usr/bin/whoami
```

Wazuh classified the activity as:

```text
Rule ID: 5401
Rule Level: 5
Rule Description: Failed attempt to run sudo.
```

### Evidence

![Wazuh failed sudo alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-018/05-wazuh-failed-sudo-alert.png?raw=true)

---

## Successful vs Failed Privileged Activity

| Activity | Source User | Target User | Command | Result | Wazuh Rule | Level |
|---|---|---|---|---|---|---|
| Successful sudo | analyst | root | `/usr/bin/touch /root/lab018-test.txt` | Executed | 5402 | 3 |
| PAM root session | analyst | root | sudo session | Opened | 5501 | 3 |
| Failed sudo | analyst | root | `/usr/bin/whoami` | Authentication failed | 5401 | 5 |

An important observation was that the failed sudo attempt generated a higher Wazuh rule level than the successful sudo activity.

Successful sudo use can represent normal administrative behavior, while failed privilege attempts may require greater analyst attention depending on the user, host, frequency, command, and surrounding activity.

---

## Analysis

### What was observed?

Both successful and failed sudo activity was observed on the monitored Ubuntu endpoint.

### Where was it observed?

The activity was observed in:

- the Linux systemd journal;
- Wazuh Threat Hunting alerts.

### Which user and host were involved?

```text
Host: ubuntu-agent
Source user: analyst
Target privileged user: root
```

### What privileged activity occurred?

The `analyst` account successfully executed several commands as root, including:

```text
/usr/bin/whoami
/usr/bin/ls /root
/usr/bin/touch /root/lab018-test.txt
/usr/bin/ls -l /root/lab018-test.txt
```

A later attempt to execute:

```text
/usr/bin/whoami
```

through sudo failed because an incorrect password was supplied.

### Why could this activity be suspicious?

Privileged activity is security-relevant because successful sudo access allows a user to perform actions with root permissions.

In a production environment, unexpected privileged activity could be associated with:

- unauthorized administrative access;
- privilege escalation;
- security-control modification;
- persistence;
- file tampering;
- credential access;
- defense evasion.

A failed sudo attempt could also indicate an unauthorized user attempting to obtain elevated privileges.

### What evidence supports the investigation?

The evidence included:

- Linux journal sudo events;
- initiating user information;
- target root account information;
- executed command information;
- PAM root-session events;
- Wazuh rule `5402`;
- Wazuh rule `5501`;
- Wazuh rule `5401`;
- successful root-owned file creation;
- failed sudo authentication evidence.

---

## Risk

Privileged command execution represents higher-risk activity because root permissions can affect the entire Linux system.

The successful sudo activity in this lab was authorized and intentionally generated.

However, similar events in a production environment should be evaluated against:

- expected administrative behavior;
- authorized user roles;
- command sensitivity;
- change-management records;
- login history;
- source IP information;
- surrounding security alerts;
- file modifications;
- persistence indicators.

Repeated or unexplained failed sudo attempts would increase concern, particularly if followed by successful privilege elevation.

---

## Disposition

```text
Disposition: Benign Positive / Authorized Test Activity
```

Both the successful and failed sudo events were intentionally generated as part of the controlled laboratory exercise.

The detections were valid, but the underlying behavior was authorized.

Therefore:

```text
Confirmed malicious activity: No
Unauthorized privilege escalation: No
Escalation required: No
Additional containment required: No
```

In a production environment, equivalent unexplained activity would require additional investigation before closure.

---

## Recommended Next Steps

1. Confirm whether the source user is authorized to use sudo.
2. Review the exact privileged command.
3. Determine whether the command is expected for the user's role.
4. Review authentication activity immediately before the sudo event.
5. Check for repeated failed privilege attempts.
6. Review related file modifications.
7. Examine subsequent root-level activity.
8. Correlate the event with other endpoint and SIEM alerts.
9. Escalate if the privileged activity cannot be explained by authorized administration.

---

## MITRE ATT&CK Mapping

| Technique | Name | Relevance |
|---|---|---|
| T1548.003 | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | The observed events involved execution of commands through `sudo` with root privileges. In this lab the activity was authorized, so the mapping describes the mechanism observed rather than confirming malicious privilege escalation. |

---

## Final SOC Summary

Wazuh detected both successful and failed sudo activity on the monitored `ubuntu-agent` endpoint.

The `analyst` account successfully executed commands as `root`, including creation of `/root/lab018-test.txt`, which generated Wazuh rule `5402` and associated PAM session activity under rule `5501`.

A separate intentionally failed sudo authentication attempt generated rule `5401` at level `5`.

Raw systemd journal events were correlated with Wazuh alerts to verify the initiating user, target privileged account, executed command, authentication result, and session context.

All observed activity was intentionally generated during controlled testing, and no evidence of unauthorized privilege escalation was identified.

The final disposition was **Benign Positive / Authorized Test Activity**, with no escalation required.

---

## Lessons Learned

This lab demonstrated that privileged Linux activity should not be evaluated only by the presence of a sudo alert.

Important investigation context includes:

- who initiated the command;
- which privileged account was requested;
- what command was executed;
- whether authentication succeeded;
- whether a root session was opened;
- what actions occurred after privilege elevation;
- whether the behavior was expected.

The investigation also demonstrated that Linux authentication evidence is not always stored in `/var/log/auth.log`.

On this endpoint, the relevant sudo evidence was available through `journald`, requiring the investigation workflow to adapt to the host's logging configuration.

The comparison between Wazuh rules `5402` and `5401` also demonstrated that successful administrative activity may receive a lower alert level than a failed privilege attempt.

Alert severity must therefore be interpreted together with user, command, authentication, host, and operational context rather than being treated as proof of malicious activity.
