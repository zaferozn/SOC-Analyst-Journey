# Lab 004 - Wazuh Environment Validation

## Objective

Verify whether Wazuh is installed or running on the current Linux VM before continuing with Wazuh alert analysis.

This lab also documents basic environment validation, SSH log filtering, source IP extraction, username extraction, and repeated failed login pattern review.

## Environment / Data Source

Main VM: `soclab-main`  
Current user: `analyst`  
Current directory: `/home/analyst`  
VM IP address: `192.168.64.7`  
Tool: `whoami`, `pwd`, `ip a`, `systemctl`, `ls`, `dpkg`, `find`, `docker`, `journalctl`, `grep`, `sort`, `uniq`, `wc`, `tail`  
Log source: Linux journal logs  
Service reviewed: SSH  
SIEM component reviewed: Wazuh  

## Background

The current VM was previously used as an SSH target for failed login and brute-force style lab exercises.

Before analyzing Wazuh alerts, the environment had to be validated to confirm whether Wazuh Manager or related components were actually installed on this system.

This step is important because a SOC analyst should not assume that a SIEM, agent, service, or log source is available without validating the environment first.

## Commands Practiced

```bash
whoami
pwd
ip a
systemctl list-units --type=service | grep -i wazuh
ls /var/ossec
ls /var/ossec/logs
ls /var/ossec/logs/alerts
dpkg -l | grep -i wazuh
sudo find / -iname "*wazuh*" 2>/dev/null | head -n 20
docker ps
```

## Command Understanding

- `whoami` shows the current Linux user.
- `pwd` shows the current working directory.
- `ip a` shows network interfaces and IP address information.
- `systemctl list-units --type=service | grep -i wazuh` checks whether Wazuh-related services are visible.
- `ls /var/ossec` checks whether the default Wazuh/OSSEC installation directory exists.
- `ls /var/ossec/logs` checks whether Wazuh log directories exist.
- `ls /var/ossec/logs/alerts` checks whether Wazuh alert logs exist.
- `dpkg -l | grep -i wazuh` checks whether Wazuh-related Debian/Ubuntu packages are installed.
- `sudo find / -iname "*wazuh*" 2>/dev/null | head -n 20` searches the file system for Wazuh-related files or directories.
- `docker ps` checks whether Docker containers are running.

## Observed Results

- No Wazuh-related services were returned by `systemctl`.
- `/var/ossec` did not exist.
- No Wazuh-related packages were returned by `dpkg`.
- No Wazuh-related files or directories were found using `find`.
- Docker was not installed on this VM.
- A second Linux VM existed, but it failed to boot due to a filesystem consistency issue.

## Wazuh Environment Analysis

The current Linux VM does not appear to have Wazuh Manager installed or running.

The evidence suggests that this VM is currently functioning as a Linux SSH target system rather than a Wazuh Manager node.

Because Wazuh was not available on this VM, Wazuh alert analysis cannot continue on the current system until Wazuh Manager is installed, restored, or located on another VM.

## Key Environment Takeaway

Before analyzing SIEM alerts, the analyst must validate the environment.

In this case, Wazuh alert analysis cannot continue on the current VM because there is no evidence that Wazuh Manager is installed or running.

The lab can continue with Linux authentication log analysis before rebuilding or reinstalling Wazuh.

## Current Learning Note

These commands were used to validate whether the current VM was suitable for Wazuh alert analysis.

The lab also supported command-line fluency through repeated environment checks.

## Time Window Filtering Practice

I used `journalctl --since` to filter SSH authentication events within a specific time window.

Commands used:

```bash
journalctl --since "2 hours ago" | grep -i "failed password" | wc -l
journalctl --since "2 hours ago" | grep -i "failed password" | tail -n 10
journalctl --since "2 hours ago" | grep -i "invalid user" | wc -l
```

Observed results:

- `Failed password` entries: `6`
- `Invalid user` matching lines: `6`
- Source IP: `192.168.64.1`
- Usernames observed: `Fakeuser`, `fakeuser`
- Source ports: `53939`, `53940`
- Time window: `Apr 28 08:22:44 - Apr 28 08:23:25 UTC`
- Approximate duration: `41 seconds`

Example log entries:

```text
Apr 28 08:22:44 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:48 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:22:54 soclab sshd[5411]: Failed password for invalid user Fakeuser from 192.168.64.1 port 53939 ssh2
Apr 28 08:23:16 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:21 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
Apr 28 08:23:25 soclab sshd[5413]: Failed password for invalid user fakeuser from 192.168.64.1 port 53940 ssh2
```

## Time Window Analysis

The logs showed `6` failed SSH login attempts from the same source IP, `192.168.64.1`, within approximately `41 seconds`.

The attempts targeted invalid usernames such as `Fakeuser` and `fakeuser`.

Since Linux usernames are case-sensitive, `Fakeuser` and `fakeuser` may be interpreted as separate username attempts.

This can be interpreted as a repeated failed login pattern. The activity may indicate brute-force behavior or username-guessing activity.

## Source IP and Username Extraction

I practiced extracting source IP addresses and targeted usernames from failed SSH login logs.

Commands used:

```bash
journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'from [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c
journalctl --since "2 hours ago" | grep -i "failed password" | grep -oE 'invalid user [^ ]+' | sort | uniq -c
```

Observed results:

- Source IP `192.168.64.1` appeared in `6` failed SSH login attempts.
- `Fakeuser` appeared in `3` failed SSH login attempts.
- `fakeuser` appeared in `3` failed SSH login attempts.

## Extraction Interpretation

Within the selected 2-hour time window, `6` failed SSH login attempts were observed from the same source IP, `192.168.64.1`.

The attempts targeted invalid username variations such as `Fakeuser` and `fakeuser`.

Since Linux usernames are case-sensitive, these may be interpreted as separate username attempts.

This pattern may indicate brute-force or username-guessing activity.

Further investigation would require checking whether any login attempt was successful and whether the same source IP appears in other authentication events.

## Analyst Interpretation

This lab produced two important findings.

First, Wazuh does not appear to be installed or running on the current VM. Therefore, Wazuh alert analysis cannot continue on this specific system without rebuilding, restoring, or reinstalling the Wazuh environment.

Second, SSH authentication logs were still available through Linux journal logs. These logs allowed basic SOC-style analysis, including time window filtering, failed login counting, source IP extraction, username extraction, and repeated failed login pattern review.

The activity from `192.168.64.1` was expected in the controlled lab environment. In a real SOC environment, repeated failed SSH login attempts from the same source IP within a short time window would require further review.

## Risk

The Wazuh-related risk in this lab is visibility loss. If the analyst assumes Wazuh is running when it is not, alerts may be missed or unavailable.

The SSH-related risk is repeated authentication failure. In a real environment, repeated failed login attempts from the same source IP may indicate brute-force activity, username guessing, or unauthorized access attempts.

## Recommended Next Steps

- Rebuild, restore, or reinstall the Wazuh environment before continuing Wazuh alert analysis.
- Continue practicing Linux authentication log analysis using `journalctl`.
- Review whether failed SSH attempts are followed by successful logins.
- Extract source IPs and usernames from authentication logs.
- Compare raw Linux logs with Wazuh alerts after Wazuh is restored.
- Document the difference between raw log visibility and SIEM alert visibility.

## Final SOC Summary

The current Linux VM was reviewed to determine whether Wazuh Manager or related components were installed and running. No Wazuh services, directories, packages, files, or Docker containers were identified. This indicates that the current VM is functioning as a Linux SSH target system rather than a Wazuh Manager node. During the same lab, SSH authentication logs were reviewed using `journalctl`. Six failed SSH login attempts were observed from the source IP `192.168.64.1` within approximately 41 seconds, targeting invalid username variations such as `Fakeuser` and `fakeuser`. In the controlled lab environment, this activity was expected. In a real SOC environment, this pattern would require further review.

## Lessons Learned

This lab showed that environment validation is required before SIEM alert analysis. A SOC analyst must confirm whether the required service, log source, or SIEM component is actually available. The lab also reinforced basic Linux log analysis skills, including time window filtering, failed login counting, source IP extraction, username extraction, and repeated authentication pattern recognition.

## CV Wording

Validated a Linux SOC lab environment by checking Wazuh service availability, installation paths, packages, and Docker status, then continued SSH authentication log analysis using `journalctl` to identify repeated failed login patterns, source IPs, and invalid username activity.

## Interview Answer

In this lab, I first validated whether Wazuh was installed or running on the current Linux VM. I checked services, installation directories, packages, file system traces, and Docker containers. The evidence showed that Wazuh was not available on that VM, so I treated it as a Linux SSH target system. I then continued with raw authentication log analysis using `journalctl`, filtering failed SSH logins by time window, extracting source IPs and usernames, and identifying a repeated failed login pattern.

## SOC English Sentences

The current VM does not appear to have Wazuh Manager installed or running.

No Wazuh-related services, packages, directories, or Docker containers were identified.

The system appears to be functioning as a Linux SSH target rather than a Wazuh Manager node.

Six failed SSH login attempts were observed from the same source IP within approximately 41 seconds.

The attempts targeted invalid username variations such as `Fakeuser` and `fakeuser`.

This pattern may indicate brute-force or username-guessing activity.

Further review is required to determine whether any successful login occurred after the failed attempts.

Environment validation should be completed before continuing SIEM alert analysis.
