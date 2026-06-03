# Case 002 - Failed-to-Successful SSH Login Review

## Executive Summary

This case documents a Linux SSH authentication review in a home SOC lab. Three failed SSH login attempts were followed by one successful SSH login from the same source IP address. The activity involved the same user account and occurred within a short time window. This pattern may represent normal user error, but it also requires review because failed login attempts followed by a successful login can indicate possible credential compromise.

## Objective

The objective of this case is to investigate failed SSH login attempts followed by a successful SSH login and determine whether the activity should be treated as expected behavior, suspicious activity, or an event requiring escalation.

## Environment / Data Source

Host: soclab  
User: analyst  
Tool: journalctl  
Log source: Linux journal logs / SSH authentication logs  
Time window: Last 30 minutes  
Source IP: 192.168.64.1  

## Observed Activity

SSH authentication logs showed three failed password attempts for the user `analyst` from the source IP address `192.168.64.1`. Shortly after these failed attempts, a successful SSH login was observed for the same user from the same source IP address.

A session was opened after the successful authentication and later closed.

## Evidence

### Failed and Successful SSH Authentication Events

Command used:

```bash
sudo journalctl --since "30 minutes ago" | grep -Ei "failed password|accepted password"
```

Observed output:

```text
Jun 03 09:30:36 soclab sshd[977]: Failed password for analyst from 192.168.64.1 port 49747 ssh2
Jun 03 09:30:43 soclab sshd[977]: Failed password for analyst from 192.168.64.1 port 49747 ssh2
Jun 03 09:30:47 soclab sshd[977]: Failed password for analyst from 192.168.64.1 port 49747 ssh2
Jun 03 09:31:12 soclab sshd[979]: Accepted password for analyst from 192.168.64.1 port 49771 ssh2
```

### Session Activity

Command used:

```bash
sudo journalctl --since "30 minutes ago" | grep -Ei "session opened|session closed"
```

Observed output:

```text
Jun 03 09:31:12 soclab sshd[979]: pam_unix(sshd:session): session opened for user analyst(uid=1000) by analyst(uid=0)
Jun 03 09:31:21 soclab sshd[979]: pam_unix(sshd:session): session closed for user analyst
```

Additional sudo session activity was also observed:

```text
Jun 03 09:20:19 soclab sudo[970]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
Jun 03 09:31:39 soclab sudo[970]: pam_unix(sudo:session): session closed for user root
Jun 03 09:33:05 soclab sudo[1008]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
Jun 03 09:33:05 soclab sudo[1008]: pam_unix(sudo:session): session closed for user root
Jun 03 09:34:00 soclab sudo[1012]: pam_unix(sudo:session): session opened for user root(uid=0) by analyst(uid=1000)
```

### Source IP Count

Command used:

```bash
sudo journalctl --since "30 minutes ago" | grep -Ei "failed password|accepted password" | grep -oE 'from [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c
```

Observed output:

```text
4 from 192.168.64.1
```

### Username Count

Command used:

```bash
sudo journalctl --since "30 minutes ago" | grep -Ei "failed password|accepted password" | grep -oE 'for (invalid user )?[^ ]+' | sort | uniq -c
```

Observed output:

```text
4 for analyst
```

## Analysis

The logs show a clear failed-to-successful SSH authentication pattern. The same source IP address, `192.168.64.1`, generated three failed login attempts and then completed one successful login for the same user account, `analyst`.

This activity does not automatically confirm malicious access. In this lab context, the failed attempts were intentionally generated and the successful login was expected. However, in a real SOC environment, the same pattern would require investigation.

The key concern is that repeated failed attempts followed by a successful login may indicate that an attacker guessed or obtained valid credentials. The analyst should confirm whether the source IP address is expected, whether the user recognizes the login, and whether any suspicious activity occurred after authentication.

The SSH session was opened at `09:31:12` and closed at `09:31:21`, which indicates a short authenticated session. Additional sudo session activity was also visible in the logs. In a real investigation, this would require further review to determine whether privileged commands were expected or suspicious.

## Risk

The primary risk is unauthorized access through valid SSH credentials. If the successful login was not expected, the attacker may have gained access to the host using compromised or guessed credentials. This could lead to command execution, privilege escalation, persistence, lateral movement, or access to sensitive data.

## Recommended Next Steps

- Confirm whether the source IP address `192.168.64.1` is expected.
- Verify whether the user `analyst` recognizes the successful login.
- Review post-login activity after the SSH session opened.
- Check sudo activity and command history if available.
- Determine whether the login occurred during an expected time window.
- Monitor for repeated failed-to-successful login patterns.
- Escalate if the successful login is not recognized or if suspicious post-login activity is identified.

## MITRE ATT&CK Mapping

Tactic: Credential Access  
Technique: Brute Force  
Technique ID: T1110  

Tactic: Initial Access  
Technique: Valid Accounts  
Technique ID: T1078  

## Final SOC Summary

Three failed SSH login attempts were observed for the user `analyst` from the source IP address `192.168.64.1`. Shortly after the failed attempts, a successful SSH login was observed from the same source IP address for the same user account. A session was opened and later closed. In this lab environment, the activity was expected and controlled. In a real SOC environment, this pattern would require further review because failed login attempts followed by a successful login may indicate possible credential compromise.

## Lessons Learned

This case helped me understand that failed login events should not be reviewed in isolation. The important SOC skill is to correlate failed attempts, successful authentication, source IP address, username, time window, and session activity. A successful login after multiple failed attempts is more important than failed attempts alone because it may indicate unauthorized access with valid credentials.
