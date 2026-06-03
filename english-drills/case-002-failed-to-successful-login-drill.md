# Case 002 - Failed-to-Successful SSH Login English Drill

## Core SOC Sentences

Three failed SSH login attempts were observed before a successful login.

The same source IP address was involved in both failed and successful authentication events.

The activity involved the user account `analyst`.

A session was opened after the successful SSH authentication.

This pattern may indicate normal user error or possible credential compromise.

Further review is required to confirm whether the login was authorized.

The source IP address should be checked against expected user activity.

Post-login activity should be reviewed to identify any unusual commands or privilege escalation.

The event should be escalated if the user does not recognize the successful login.

## Analyst Summary

The authentication logs showed three failed SSH login attempts followed by one successful login from the same source IP address. The activity involved the same user account and occurred within a short time window. Although this was expected in the lab environment, the same pattern in a real SOC environment would require further investigation.

## SOC Report Version

Three failed SSH login attempts were observed for the user `analyst` from the source IP address `192.168.64.1`. Shortly after these failed attempts, a successful SSH login was observed from the same source IP address. A session was opened and later closed. This pattern may indicate normal user error, but it may also suggest possible credential compromise if the login was not expected or authorized.

## Interview Answer

In this lab, I simulated failed SSH login attempts followed by a successful login. I used `journalctl` to review Linux authentication logs and identified the source IP address, username, failed login count, successful login event, and session activity. The key lesson was that failed login events should not be reviewed alone. A successful login after multiple failed attempts is more important because it may indicate possible credential compromise.

## Stronger Interview Version

I completed a Linux SSH authentication triage lab focused on a failed-to-successful login pattern. I generated three failed SSH login attempts and then performed a successful login from the same source IP address. After that, I reviewed the authentication logs with `journalctl`, correlated the source IP, username, timestamps, authentication results, and session activity. This helped me understand how SOC analysts move from simple log filtering to event correlation and escalation decision-making.

## CV Bullet

Performed SSH authentication log triage by correlating failed login attempts, successful logins, source IP addresses, usernames, and session activity in a Linux lab environment.

## Improved CV Bullet

Investigated failed-to-successful SSH login patterns using Linux journal logs to assess possible credential compromise and document escalation criteria.

## Key Vocabulary

failed login attempt  
successful authentication  
source IP address  
user account  
session opened  
session closed  
credential compromise  
authorized login  
post-login activity  
privilege escalation  
escalation criteria  
authentication pattern  
event correlation  
log triage  
expected behavior  
suspicious activity  

## Mini Drill

Question: What was observed?

Answer: Three failed SSH login attempts were followed by one successful SSH login.

Question: Where was it observed?

Answer: The activity was observed in Linux journal logs on the host `soclab`.

Question: Which user and source IP were involved?

Answer: The activity involved the user `analyst` and the source IP address `192.168.64.1`.

Question: Why is it suspicious?

Answer: It is suspicious because failed login attempts followed by a successful login may indicate possible credential compromise.

Question: What evidence supports it?

Answer: The logs showed three failed password events, one accepted password event, and one SSH session opened event.

Question: What should be done next?

Answer: The source IP, user activity, login time, and post-login behavior should be reviewed.

Question: Should it be escalated?

Answer: It should be escalated if the login was not expected, if the user does not recognize it, or if suspicious post-login activity is identified.

## Final Practice Paragraph

The logs showed three failed SSH login attempts followed by one successful login for the user `analyst`. The same source IP address, `192.168.64.1`, was involved in both failed and successful authentication events. A session was opened after the successful login and later closed. In this lab environment, the activity was expected. However, in a real SOC environment, this pattern would require further investigation because it may indicate possible credential compromise.
