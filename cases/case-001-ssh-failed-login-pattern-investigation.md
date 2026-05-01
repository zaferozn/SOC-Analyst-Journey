# Case 001 - SSH Failed Login Pattern Investigation

## Objective
This case documents a simulated SSH failed-login investigation in a home SOC lab environment using Linux journal logs.

## Lab Environment
- Hostname: `soclab`
- Target System: Ubuntu Linux VM
- VM IP Address: `192.168.64.7`
- Log Source: Linux journal logs / SSH authentication events
- Service: SSH
- Analyst Focus: Authentication log review, failed login pattern analysis, source IP extraction, username extraction, and incident documentation

## Important Note
Wazuh Manager was not installed on the current VM during this phase. This case focuses on Linux authentication log analysis before SIEM alert analysis.

## Commands Used
```bash
journalctl --since "2 hours ago" | grep -i "failed password" | wc -l
journalctl --since "2 hours ago" | grep -i "failed password" | tail -n 10
journalctl --since "2 hours ago" | grep -i "invalid user" | wc -l
journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'from [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c
journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'invalid user [^ ]+' | sort | uniq -c
```

## Observed Pattern
- `6` failed SSH login attempts were observed.
- The attempts came from source IP `192.168.64.1`.
- The attempts targeted invalid usernames such as `Fakeuser` and `fakeuser`.
- The activity occurred within approximately `41 seconds`.
- The source ports observed were `53939` and `53940`.

## Example Log Entries
```text
Apr 28 08:22:44 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:48 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:54 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:23:16 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:21 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:25 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
```

## Analysis
The logs showed repeated failed SSH login attempts from the same source IP within a short time window. The attempts targeted invalid usernames such as `Fakeuser` and `fakeuser`.

This pattern may indicate brute-force or username-guessing activity. Since the usernames were invalid, the activity may also represent attempts to discover valid usernames on the target system.

## Analyst Actions
- Reviewed failed SSH login events in Linux journal logs.
- Filtered events by time window using `journalctl --since`.
- Counted failed password entries using `wc -l`.
- Extracted the source IP from failed login logs.
- Extracted targeted invalid usernames.
- Reviewed whether the pattern may indicate brute-force or username-guessing activity.

## Recommended Next Steps
- Verify whether any SSH login attempt was successful.
- Check whether the source IP appears in other authentication events.
- Review whether the targeted usernames exist on the system.
- If unauthorized access is confirmed, escalate the incident for further investigation.

## MITRE ATT&CK Mapping
- Tactic: Credential Access
- Technique: Brute Force
- Technique ID: T1110

## Incident Summary
Within the selected time window, `6` failed SSH login attempts were observed from the same source IP, `192.168.64.1`. The attempts targeted invalid usernames such as `Fakeuser` and `fakeuser`. This pattern may indicate brute-force or username-guessing activity. The next step is to verify whether any login attempt was successful.

## Key SOC Concepts
- SSH
- Failed login
- Invalid user
- Source IP
- Source port
- Time window
- Brute-force activity
- Username guessing
- Authentication logs
- Incident summary
