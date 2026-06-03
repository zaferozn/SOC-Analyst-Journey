# Linux Foundation Notes

## Purpose
This note documents the initial Linux, SSH, GitHub, and Wazuh-related foundation work completed during the early SOC portfolio setup.

## Objectives
- Set up a GitHub repository for SOC portfolio development.
- Organize the SOC study structure.
- Review the existing Wazuh lab environment.
- Start documenting hands-on observations.
- Identify technical and reporting gaps for future SOC labs.

## Initial Notes
- A GitHub repository was created for SOC portfolio development.
- The main lab environment is currently based on Wazuh running in a Linux environment through UTM.
- The next step is to review services, logs, and alert-related data from the CLI.

## Previous Hands-On Exposure
- Installed and ran Wazuh in a Linux environment through UTM.
- Connected to the Linux system through SSH.
- Practiced a brute-force style lab in a controlled environment.
- Observed logs during the attack simulation.
- Tested a custom rule idea based on repeated login attempts.

## Current Knowledge
- Basic SSH connection usage.
- Basic Linux identity and network commands such as `whoami` and `ip a`.
- Basic awareness of authentication logs.
- Basic awareness of failed login attempts.
- Basic awareness of brute-force patterns.

## Knowledge Gaps
- How to independently trace logs step by step.
- How Wazuh rules are structured.
- How alerts are generated from raw logs.
- How to explain findings in an analyst-style format.
- How to reproduce the workflow without guidance.

## Concepts Practiced
- SSH
- Failed login attempts
- Source IP identification
- Brute-force activity
- Alert monitoring
- Log review

## Main Difficulties
- Explaining technical actions in English.
- Understanding raw logs independently.
- Separating alert logic from full raw log data.
- Understanding the relationship between raw logs, rules, and alerts.

## Foundation Summary
The initial GitHub structure was created for a SOC portfolio. Previous Wazuh lab exposure was converted into structured documentation. Key learning gaps were identified in Linux log analysis, Wazuh rule logic, alert generation, and analyst-style reporting.

## Next Focus
The next focus is to move from basic observation toward structured SOC investigation logic: raw log review, observable extraction, pattern recognition, alert logic, severity assessment, and escalation decisions.
