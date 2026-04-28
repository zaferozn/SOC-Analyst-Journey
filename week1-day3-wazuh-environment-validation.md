# Week 1 - Day 3: Wazuh Environment Validation

## Objective
Verify whether Wazuh is installed or running on the current Linux VM before continuing with Wazuh alert analysis.

## Current Lab System
- Main VM: `soclab-main`
- Current user: `analyst`
- Current directory: `/home/analyst`
- VM IP address: `192.168.64.7`

## Background
The current VM was previously used as an SSH target for failed login and brute-force style lab exercises. Before analyzing Wazuh alerts, I needed to confirm whether Wazuh Manager or related components were actually installed on this system.

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

## Analyst Interpretation
The current Linux VM does not appear to have Wazuh Manager installed or running. The evidence suggests that this VM is currently functioning as a Linux SSH target system rather than a Wazuh Manager node.

The lab will continue with a simplified single-VM setup for Linux authentication log analysis before rebuilding or reinstalling Wazuh.

## Key Takeaway
Before analyzing SIEM alerts, it is necessary to validate the environment. In this case, Wazuh alert analysis cannot continue on the current VM until Wazuh Manager is installed, restored, or located on another VM.
