# Lab 001 - SSH Log Observation

## Objective

Understand the difference between successful SSH login events, failed SSH login attempts, invalid user attempts, and repeated failed authentication patterns in Linux journal logs.

## Environment / Data Source

Host: `soclab`  
Tool: `journalctl`, `grep`, `tail`, `wc`  
Log source: Linux SSH authentication logs  
Source IP observed: `192.168.64.1`

## Commands Practiced

```bash
ls /var/log
journalctl -n 20
journalctl | grep sshd | tail -n 20
journalctl | grep "Accepted password" | tail -n 5
journalctl | grep "Failed password" | tail -n 20
journalctl | grep "Invalid user" | tail -n 20
journalctl -f | grep sshd
```

## Command Understanding

- `ls /var/log` lists the files and folders inside the `/var/log` directory.
- `journalctl -n 20` shows the latest 20 system journal log entries.
- `journalctl | grep sshd | tail -n 20` filters SSH service-related log entries and shows the latest 20 matching lines.
- `grep` filters log output by a specific word or phrase.
- `tail -n 20` shows the latest 20 matching lines.
- The `|` symbol sends the output of one command into the next command.
- `journalctl -f | grep sshd` follows live journal logs and shows new SSH-related entries as they appear.

## Successful SSH Login Observation

`Accepted password for analyst` indicated a successful SSH login for the `analyst` user.

This showed that the SSH authentication process succeeded for a valid local user.

## Failed SSH Login Observation

`Failed password for invalid user fakeuser` indicated a failed SSH login attempt using a username that does not exist on the system.

Key observations:

- `Invalid user` means the attempted username is not a valid local user.
- `Failed password` means the password authentication attempt failed.
- `Connection closed` means the SSH connection was closed after the attempt.

## Mac-to-Linux Failed SSH Attempt

A failed SSH login attempt was generated from the Mac terminal against the Linux VM using an invalid username.

Observed log entries included:

```text
Invalid user fakeuser from 192.168.64.1 port 50587
pam_unix(sshd:auth): check pass; user unknown
authentication failure
Failed password for invalid user fakeuser from 192.168.64.1 port 50587 ssh2
PAM 2 more authentication failures
```

## Field Interpretation

| Field | Meaning |
|---|---|
| `192.168.64.1` | Source IP initiating the SSH attempt |
| `50587` | Temporary client-side source port |
| `fakeuser` | Invalid username used in the login attempt |
| `Failed password` | Password authentication failed |
| `ssh2` | SSH protocol version 2 |
| `preauth` | Activity occurred before successful authentication |

The source port `50587` is not the SSH service port. It is a temporary client-side port. The SSH service itself is typically listening on port `22`.

## Initial Analyst Interpretation

A single failed login attempt may occur during normal activity. However, repeated failed login attempts, especially against invalid users or from the same source IP, may indicate brute-force activity and require further investigation.

## Repeated Failed Login Pattern

Failed SSH login attempts were reviewed within a defined time window using `journalctl`, `grep`, and `wc -l`.

Commands used:

```bash
journalctl --since "2 hours ago" | grep -i "Failed password" | wc -l
journalctl --since "2 hours ago" | grep -i "invalid user" | wc -l
journalctl --since "2 hours ago" | grep -i "Failed password" | tail -n 10
```

Observed results:

- `Failed password` count: 4
- `Invalid user` matching lines: 7
- Source IP: `192.168.64.1`
- Usernames observed: `fakeuser`, `kort`

Example log entries:

```text
Apr 26 16:17:49 soclab sshd[4508]: Failed password for invalid user fakeuser from 192.168.64.1 port 50861 ssh2
Apr 26 16:17:53 soclab sshd[4508]: Failed password for invalid user fakeuser from 192.168.64.1 port 50861 ssh2
Apr 26 16:17:58 soclab sshd[4508]: Failed password for invalid user fakeuser from 192.168.64.1 port 50861 ssh2
Apr 26 16:18:38 soclab sshd[4510]: Failed password for invalid user kort from 192.168.64.1 port 50862 ssh2
```

## Interpretation of Counts

The `Failed password` count was 4 because it counted only log lines containing the phrase `Failed password`.

The `Invalid user` count was 7 because the phrase `invalid user` appeared in multiple types of log lines, including both `Invalid user ...` entries and `Failed password for invalid user ...` entries.

This means the counts are based on matching log lines, not necessarily unique security events.

## Analysis

The logs showed multiple failed SSH login attempts from the same source IP within a short time window. The attempts targeted invalid usernames such as `fakeuser` and `kort`.

This can be interpreted as a repeated failed login pattern. A single failed attempt is not enough to confirm brute-force activity, but repeated failed attempts from the same source IP may indicate brute-force behavior.

## Risk

Repeated failed SSH login attempts may indicate credential guessing, brute-force activity, or unauthorized access attempts. If the same activity continues, the source IP and targeted usernames should be reviewed further.

## Recommended Next Steps

- Review additional SSH authentication logs from the same time window.
- Check whether the source IP is expected in the lab environment.
- Review whether the usernames are valid or invalid local users.
- Compare failed login attempts with successful login events.
- Monitor whether the same source IP continues to generate failed attempts.
- Escalate if repeated attempts are followed by a successful login.

## Final SOC Summary

Multiple failed SSH login attempts were observed in Linux journal logs. The attempts originated from the same source IP, `192.168.64.1`, and targeted invalid usernames including `fakeuser` and `kort`. While a single failed login attempt may be benign, repeated failed attempts from the same source IP may indicate brute-force behavior and should be reviewed further.

## Lessons Learned

This lab helped distinguish between successful SSH logins, failed login attempts, invalid user events, and repeated failed authentication patterns. It also showed that log counts are based on matching lines, not always unique security events.

## CV Wording

Analyzed Linux SSH authentication logs to distinguish successful logins, failed login attempts, invalid user events, source IPs, source ports, and repeated failed authentication patterns.

## Interview Answer

In this lab, I reviewed Linux journal logs to understand SSH authentication activity. I compared accepted password events, failed password events, and invalid user attempts. I also counted repeated failed login attempts within a time window and learned that matching log-line counts are not always the same as unique security events.

## SOC English Sentences

A failed SSH login attempt was observed from the source IP `192.168.64.1`.

The attempted username did not exist on the Linux system.

Multiple failed authentication attempts from the same source IP may indicate brute-force behavior.

The source port was a temporary client-side port, not the SSH service port.

The activity should be reviewed further if repeated attempts continue or are followed by a successful login.
