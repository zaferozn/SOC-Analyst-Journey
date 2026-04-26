# Week 1 - Day 2: SSH Log Observation

## Objective
Understand the difference between successful and failed SSH login events in Linux journal logs.

## Commands Used
```bash
ls /var/log
journalctl -n 20
journalctl | grep sshd | tail -n 20
journalctl | grep "Failed password" | tail -n 20
journalctl | grep "Invalid user" | tail -n 20

Observations

* Accepted password for analyst indicated a successful SSH login for the analyst user.
* Failed password for invalid user fakeuser indicated a failed SSH login attempt using a username that does not exist on the system.
* Invalid user means the attempted username is not a valid local user.
* Failed password means the password authentication attempt failed.
* Connection closed showed that the SSH connection was closed after the attempt.

Initial Analyst Interpretation

A single failed login attempt may occur during normal activity. However, repeated failed login attempts, especially against invalid users or from the same source IP, may indicate brute-force activity and require further investigation.

Current Understanding

I can now distinguish between a successful SSH login event and a failed login attempt in the logs. I also understand that brute-force analysis depends on repeated patterns, not only a single failed event.

## Mac-to-Linux Failed SSH Attempt

I generated a failed SSH login attempt from my Mac terminal against the Linux VM using an invalid username.

Observed log entries included:
- `Invalid user fakeuser from 192.168.64.1 port 50587`
- `pam_unix(sshd:auth): check pass; user unknown`
- `authentication failure`
- `Failed password for invalid user fakeuser from 192.168.64.1 port 50587 ssh2`
- `PAM 2 more authentication failures`

## Interpretation
The source IP `192.168.64.1` represents the system initiating the SSH attempt. The port `50587` is a temporary source port used by the client side of the connection, not the SSH service port. The SSH service itself is typically listening on port 22.

`Invalid user fakeuser` means the username does not exist on the Linux system. `Failed password` and `authentication failure` show that the authentication attempt failed. The `ssh2` value indicates SSH protocol version 2. The `preauth` stage refers to activity that happened before successful authentication.

This event is a failed SSH login attempt using an invalid user. A single event is not enough to confirm brute-force activity, but repeated failed attempts from the same source IP may indicate a brute-force pattern.
