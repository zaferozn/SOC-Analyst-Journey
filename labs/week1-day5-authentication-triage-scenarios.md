# Week 1 Day 5 - Authentication Triage Scenarios

## Objective

This note documents basic authentication triage practice for SOC L1 analysis.

The goal was to review short SSH authentication scenarios and classify them as expected, suspicious, or requiring more context.

## Triage Focus

The analysis focused on:

- Failed SSH login attempts
- Successful SSH logins
- Invalid usernames
- Source IP review
- Login time review
- Post-authentication activity
- Escalation decision-making

## Scenario 1 - Multiple Invalid User Attempts

### Log Pattern

```text
10:01 - Failed password for invalid user admin from 185.22.10.5
10:01 - Failed password for invalid user test from 185.22.10.5
10:02 - Failed password for invalid user oracle from 185.22.10.5
10:02 - Failed password for invalid user postgres from 185.22.10.5
10:03 - Failed password for invalid user backup from 185.22.10.5
```

### Assessment

Suspicious.

### Reasoning

The logs show multiple failed SSH login attempts from the same source IP targeting different invalid usernames.

This pattern may indicate username-guessing activity or brute-force style behavior.

### First Checks

- Check whether the source IP appeared in previous authentication events.
- Check whether any successful login occurred from the same source IP.
- Check whether the target system is expected to receive SSH connections from this IP.

## Scenario 2 - Single Successful Login

### Log Pattern

```text
09:15 - Accepted password for analyst from 192.168.64.1
09:16 - session opened for user analyst
09:18 - session closed for user analyst
```

### Assessment

Needs more context / likely expected if the source IP and login time match normal user behavior.

### Reasoning

The logs show a single successful SSH login followed by a session open and session close event.

There are no failed attempts, invalid usernames, or repeated authentication failures in this short timeline.

### First Checks

- Check whether the source IP is expected for the user.
- Check whether the login time matches the user's normal behavior.
- Review session activity for anything unusual.

## Scenario 3 - Failed Attempts Followed by Successful Login

### Log Pattern

```text
22:41 - Failed password for analyst from 45.90.12.44
22:41 - Failed password for analyst from 45.90.12.44
22:42 - Failed password for analyst from 45.90.12.44
22:43 - Accepted password for analyst from 45.90.12.44
22:44 - session opened for user analyst
```

### Assessment

Suspicious / needs investigation.

### Reasoning

The logs show repeated failed SSH login attempts against the same valid user, followed shortly by a successful login from the same source IP.

This pattern may indicate possible account compromise, especially if the source IP or login time is unusual for the user.

### First Checks

- Check whether the source IP is expected for the user.
- Compare the login time with the user's normal login behavior.
- Review post-authentication activity after the successful login.

### Escalation Criteria

Escalate to L2 if:

- The source IP is unusual or not expected.
- The user does not recognize the login.
- Suspicious post-authentication activity is observed.
- The successful login is followed by privilege escalation, unusual commands, or access to sensitive resources.

## Key Takeaways

- A single failed login does not confirm brute-force activity.
- Repeated failed attempts from the same source IP should be reviewed carefully.
- Invalid usernames may indicate username-guessing activity.
- A successful login after repeated failed attempts requires further review.
- Source IP, login time, user behavior, and post-authentication activity are important context points for L1 triage.
- Escalation should be based on suspicious context, not only on the existence of a successful login.
