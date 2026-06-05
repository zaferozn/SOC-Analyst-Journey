# Lab 007 - Wazuh Agent Enrollment and First Event Validation

## Executive Summary

This lab documents the deployment and validation of a Wazuh agent on a dedicated Ubuntu endpoint. The Wazuh server acted as the central SIEM platform, while the Ubuntu endpoint was configured as the monitored host.

The agent was installed using the ARM64 package, configured to communicate with the Wazuh manager, started successfully, and confirmed as active in the Wazuh dashboard. Initial security events from the endpoint were also visible in Wazuh Discover, confirming successful event ingestion.

## Objective

The objective of this lab was to enroll a monitored Ubuntu endpoint into Wazuh and validate that the endpoint could communicate with the Wazuh manager and send events to the SIEM.

## Environment / Data Source

Host: ubuntu-agent  
Wazuh server: wazuh-server  
Wazuh manager IP: 192.168.64.9  
Ubuntu agent IP: 192.168.64.10  
Agent ID: 001  
Operating system: Ubuntu 24.04.4 LTS ARM64  
Wazuh agent version: 4.14.5  
Log source: journald / authlog / Wazuh alerts  
Tool: Wazuh Dashboard, Wazuh Agent, Linux terminal  

## Lab Architecture

The lab used a separated monitoring architecture:

- Wazuh server: central SIEM platform
- Ubuntu agent: monitored Linux endpoint
- Mac host: test source used to generate SSH authentication activity

The Wazuh agent was installed on the Ubuntu endpoint and configured to forward endpoint security events to the Wazuh manager.

Basic lab flow:

```text
Ubuntu endpoint → Wazuh agent → Wazuh manager → Wazuh dashboard
```

## Endpoint Validation

Before installing the Wazuh agent, the Ubuntu endpoint was validated.

Commands used:

```bash
hostname
uname -m
ip a
systemctl status ssh
```

Observed values:

```text
Hostname: ubuntu-agent
Architecture: aarch64
Ubuntu-agent IP: 192.168.64.10
SSH status: active (running)
```

The endpoint was confirmed to be reachable and OpenSSH was active, allowing authentication logs to be generated for future testing.

## Wazuh Agent Installation

Because the endpoint architecture was `aarch64`, the ARM64 Wazuh agent package was selected.

Command used:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.5-1_arm64.deb && sudo WAZUH_MANAGER='192.168.64.9' WAZUH_AGENT_NAME='ubuntu-agent' dpkg -i ./wazuh-agent_4.14.5-1_arm64.deb
```

Observed result:

```text
Selecting previously unselected package wazuh-agent.
Unpacking wazuh-agent (4.14.5-1) ...
Setting up wazuh-agent (4.14.5-1) ...
Processing triggers for libc-bin ...
```

The Wazuh agent package was installed successfully.

## Agent Service Validation

After installation, the Wazuh agent service was enabled and started.

Commands used:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```

Observed result:

```text
wazuh-agent.service - Wazuh agent
Loaded: loaded
Active: active (running)

Started wazuh-execd
Started wazuh-agentd
Started wazuh-syscheckd
Started wazuh-logcollector
Started wazuh-modulesd
Completed
```

This confirmed that the Wazuh agent service and core components were running on the monitored endpoint.

## Manager Configuration Validation

The Wazuh agent configuration was reviewed to confirm the manager address, port, and protocol.

Command used:

```bash
sudo grep -A3 -B3 "server" /var/ossec/etc/ossec.conf
```

Observed configuration:

```xml
<server>
  <address>192.168.64.9</address>
  <port>1514</port>
  <protocol>tcp</protocol>
</server>
```

This confirmed that the Ubuntu endpoint was configured to communicate with the Wazuh manager at `192.168.64.9` over TCP port `1514`.

## Agent Connection Validation

The Wazuh agent log was reviewed to confirm successful communication with the Wazuh manager.

Command used:

```bash
sudo grep -Ei "connected|error|manager|server|auth|registration" /var/ossec/logs/ossec.log | tail -n 30
```

Observed result:

```text
wazuh-agentd: INFO: Requesting a key from server: 192.168.64.9
wazuh-agentd: INFO: Trying to connect to server ([192.168.64.9]:1514/tcp).
wazuh-agentd: INFO: (4102): Connected to the server ([192.168.64.9]:1514/tcp).
```

This confirmed that the Ubuntu endpoint successfully connected to the Wazuh manager.

## Dashboard Agent Validation

The Wazuh dashboard was reviewed to confirm that the Ubuntu endpoint was enrolled and active.

Observed dashboard fields:

```text
Agent ID: 001
Agent name: ubuntu-agent
Agent IP address: 192.168.64.10
Operating system: Ubuntu 24.04.4 LTS
Wazuh agent version: 4.14.5
Group: default
Status: active
```

This confirmed that the endpoint was successfully visible in the Wazuh dashboard.

## First Event Visibility Validation

Wazuh Discover was used to confirm that events from the monitored endpoint were visible.

Observed dashboard evidence:

```text
Index: wazuh-alerts-*
Agent name: ubuntu-agent
Agent IP address: 192.168.64.10
Manager name: wazuh-server
Data source: authlog
```

Example event types observed:

```text
PAM: Login session opened
PAM: Login session closed
Successful sudo to ROOT executed
```

This confirmed that the Wazuh server was receiving and displaying endpoint events from the Ubuntu agent.

## Analysis

The Wazuh agent was successfully installed, started, and connected to the Wazuh manager. The dashboard confirmed that the endpoint was active, and Discover showed security-related events from the monitored host.

This validates the basic SIEM workflow:

```text
Ubuntu endpoint → Wazuh agent → Wazuh manager → Wazuh dashboard
```

The lab also confirmed the importance of checking endpoint architecture before agent installation. Since the monitored host was `aarch64`, the ARM64 Wazuh agent package was required.

## Risk

No malicious activity was identified in this lab. The purpose of the activity was controlled validation.

However, from a SOC perspective, the visibility achieved in this lab is important because authentication activity, sudo usage, file integrity monitoring, and endpoint telemetry can now be reviewed through Wazuh.

## Recommended Next Steps

- Generate controlled SSH failed login activity.
- Generate a successful SSH login after failed attempts.
- Review the related Wazuh alert fields.
- Compare raw Linux authentication logs with Wazuh alert fields.
- Create a separate SSH alert field breakdown lab.
- Create a case report only after the alert fields are reviewed.

## MITRE ATT&CK Mapping

MITRE ATT&CK mapping is not required for this lab because the main objective was agent enrollment and event ingestion validation.

MITRE mapping will be used in the next SSH authentication alert analysis lab.

## Final SOC Summary

A Wazuh agent was successfully deployed on a dedicated Ubuntu endpoint and configured to communicate with the Wazuh manager at `192.168.64.9`. The agent service was active, the endpoint appeared as active in the Wazuh dashboard, and initial endpoint events were visible in Wazuh Discover. This confirmed successful agent enrollment, manager communication, and event ingestion.

## Lessons Learned

- A monitored endpoint must use the correct Wazuh agent package for its architecture.
- The Wazuh manager IP must be configured on the agent side.
- `systemctl status wazuh-agent` confirms local service status.
- `/var/ossec/logs/ossec.log` confirms manager communication.
- Dashboard visibility confirms successful enrollment.
- Discover visibility confirms event ingestion.
- Agent enrollment must be validated before deeper alert triage.

## SOC English Sentences

The Wazuh agent was successfully installed on the monitored Ubuntu endpoint.

The agent was configured to communicate with the Wazuh manager at 192.168.64.9.

The agent service was active and running.

The endpoint appeared as active in the Wazuh dashboard.

Initial security events from the endpoint were visible in Wazuh Discover.

This confirmed successful agent enrollment and event ingestion.

## CV Bullet

Deployed and validated a Wazuh agent on an Ubuntu endpoint, confirming active dashboard visibility, manager communication, and endpoint event ingestion in a local SOC lab.

## Interview Explanation

In my Wazuh lab, I deployed a Wazuh agent on a separate Ubuntu endpoint and configured it to communicate with the Wazuh manager. I validated the setup by checking the agent service status, reviewing the local Wazuh agent logs, confirming that the endpoint appeared as active in the dashboard, and verifying that endpoint events were visible in Discover.

