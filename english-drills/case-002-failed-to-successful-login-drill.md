# Case 002 - Defense Drill

## Objective

This drill supports English writing and speaking practice for explaining Case 002 in a SOC analyst interview context.

The goal is to describe the failed-to-successful SSH login investigation using short, clear, and controlled analyst sentences.

## Core Sentence Pattern

```text
I worked on...
I reviewed...
The logs showed...
This pattern may indicate...
The next step is...
```

## Model Answer

```text
I worked on an SSH authentication investigation.

I reviewed Linux journal logs.

The logs showed three failed SSH login attempts followed by one successful SSH login.

The same source IP was involved in both failed and successful authentication events.

This pattern may indicate normal user error or possible credential compromise.

The next step is to verify whether the successful login was expected and authorized.
```

## Key Analyst Phrases

- SSH authentication investigation
- Linux journal logs
- failed SSH login attempts
- successful SSH login
- same source IP
- same user account
- failed-to-successful login pattern
- possible credential compromise
- expected and authorized
- session opened
- session closed
- reviewed further
- next step

## Notes

The answer should stay short and controlled.

The investigation should not be described as a Wazuh alert investigation because Wazuh was not installed on the current VM during this phase.

The safest analyst language is:

```text
This pattern may indicate...
The activity should be reviewed...
The next step is to verify whether...
```

Case 002 is different from Case 001 because this case does not only review failed login attempts.

Case 001 focused on repeated failed SSH login attempts.

Case 002 focuses on failed SSH login attempts followed by a successful SSH login.

The key SOC point is correlation:

```text
Failed attempts alone may indicate attempted access.
Failed attempts followed by successful authentication may indicate possible access achieved.
```

## Final Interview Version

```text
I worked on an SSH authentication investigation in my home SOC lab.

I reviewed Linux journal logs and identified three failed SSH login attempts followed by one successful SSH login.

The activity involved the same user account, analyst, and the same source IP address, 192.168.64.1.

This pattern may indicate normal user error, but it may also suggest possible credential compromise if the login was not expected.

I also checked session activity and confirmed that an SSH session was opened and later closed.

The next step is to verify whether the successful login was expected and authorized.
```
