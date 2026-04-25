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
