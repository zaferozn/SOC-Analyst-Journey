# Week 1 - Day 2: SSH Log Observation

## Objective
Understand the difference between successful and failed SSH login events in Linux journal logs.

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
- `Accepted password for analyst` indicated a successful SSH login for the `analyst` user.
- This showed that the SSH authentication process succeeded for a valid local user.

## Failed SSH Login Observation
- `Failed password for invalid user fakeuser` indicated a failed SSH login attempt using a username that does not exist on the system.
- `Invalid user` means the attempted username is not a valid local user.
- `Failed password` means the password authentication attempt failed.
- `Connection closed` showed that the SSH connection was closed after the attempt.

## Initial Analyst Interpretation
A single failed login attempt may occur during normal activity. However, repeated failed login attempts, especially against invalid users or from the same source IP, may indicate brute-force activity and require further investigation.

## Current Understanding
I can distinguish between a successful SSH login event and a failed SSH login attempt in the logs. I also understand that brute-force analysis depends on repeated patterns, not only a single failed event.

## Mac-to-Linux Failed SSH Attempt
I generated a failed SSH login attempt from my Mac terminal against the Linux VM using an invalid username.

Observed log entries included:

```text
Invalid user fakeuser from 192.168.64.1 port 50587
pam_unix(sshd:auth): check pass; user unknown
authentication failure
Failed password for invalid user fakeuser from 192.168.64.1 port 50587 ssh2
PAM 2 more authentication failures
```

## Interpretation
In this lab environment, the source IP `192.168.64.1` appeared as the system initiating the SSH attempt. The port `50587` is a temporary source port used by the client side of the connection, not the SSH service port. The SSH service itself is typically listening on port 22.

`Invalid user fakeuser` means the username does not exist on the Linux system. `Failed password` and `authentication failure` show that the authentication attempt failed. The `ssh2` value indicates SSH protocol version 2. The `preauth` stage refers to activity that happened before successful authentication.

This event is a failed SSH login attempt using an invalid user. A single event is not enough to confirm brute-force activity, but repeated failed attempts from the same source IP may indicate a brute-force pattern.

## Key Takeaway
Successful SSH login events, failed login attempts, invalid user attempts, and repeated failed attempts must be interpreted separately. A single failed attempt is not enough to confirm an attack, but repeated failed attempts from the same source IP can become suspicious.

## Repeated Failed Login Pattern

I reviewed failed SSH login attempts within a defined time window using `journalctl`, `grep`, and `wc -l`.

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

## Analyst Interpretation

The logs showed multiple failed SSH login attempts from the same source IP within a short time window. The attempts targeted invalid usernames such as fakeuser and kort.

This can be interpreted as a repeated failed login pattern. A single failed attempt is not enough to confirm brute-force activity, but repeated failed attempts from the same source IP may indicate brute-force behavior and require further investigation.
