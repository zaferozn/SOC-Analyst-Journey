# Case 006 - Unauthorized File Modification Investigation

## Executive Summary

This case documents the investigation of a security-relevant file modification detected by Wazuh File Integrity Monitoring (FIM) within a controlled Ubuntu security lab environment.

A monitored configuration file located at:

`/home/analyst/wazuh-fim-lab/access-control.conf`

was modified through a privileged `sudo` command.

The original configuration contained:

    remote_access=disabled
    admin_access=restricted
    logging=enabled

During the observed activity, the following configuration value was added:

    remote_access=enabled

The investigation included baseline file analysis, file-content verification, filesystem metadata review, Wazuh FIM alert analysis, Linux authentication-log analysis, privileged activity attribution, and timestamp correlation.

Wazuh generated a Level 7 `Integrity checksum changed` alert under Rule ID `550`.

Linux authentication evidence showed that the `analyst` user executed the modification command through `sudo` with `root` privileges.

The collected evidence confirms that a privileged configuration change occurred. However, the available technical evidence does not establish whether the activity was authorized or malicious.

**Final Disposition: Suspicious / Requires Authorization Validation**

---

## Objective

The objective of this investigation was to determine:

- Which file was modified
- What configuration change occurred
- When the modification occurred
- Which user performed the activity
- Whether elevated privileges were used
- Whether Wazuh detected the modification
- Whether endpoint and SIEM evidence could be correlated
- Whether the change represented legitimate administration or unauthorized tampering
- What potential security impact existed
- Whether escalation was justified

---

## Environment / Data Source

**Investigation Type:** File Integrity Monitoring / Privileged Configuration Change Investigation

**Host:** `ubuntu-agent`

**Operating System:** Ubuntu Linux

**SIEM:** Wazuh

**Primary Detection Source:** Wazuh File Integrity Monitoring (FIM)

**Additional Evidence Sources:**

- Linux filesystem metadata
- File-content inspection
- Linux authentication logs
- `sudo` activity
- Wazuh security events

**Monitored Directory:**

`/home/analyst/wazuh-fim-lab`

**Investigated File:**

`/home/analyst/wazuh-fim-lab/access-control.conf`

---

## Investigation Safety

This investigation was performed entirely within a controlled laboratory environment.

The investigated configuration file was created specifically for security-monitoring practice and did not control a production service.

No production systems, accounts, or infrastructure were affected.

---

## Observed Activity

A configuration file located inside a Wazuh real-time FIM-monitored directory was established with a restrictive baseline configuration.

The original file contained:

    remote_access=disabled
    admin_access=restricted
    logging=enabled

A privileged command was subsequently executed to append:

    remote_access=enabled

to the monitored file.

This represented a security-relevant configuration change because a setting associated with remote access changed from a restrictive state toward an enabled state.

The modification was followed by a Wazuh File Integrity Monitoring alert.

Linux authentication evidence was then reviewed to identify the user and privilege context associated with the modification.

---

## Evidence

### 1. Baseline File State

Before the security-relevant modification, the configuration file and its metadata were reviewed.

![Baseline File Metadata](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-006/01-baseline-file-metadata.png?raw=true)

Observed file:

`/home/analyst/wazuh-fim-lab/access-control.conf`

Baseline content:

    remote_access=disabled
    admin_access=restricted
    logging=enabled

Observed ownership:

    User: analyst
    Group: analyst

Initial file size:

    63 bytes

### Analyst Interpretation

This evidence establishes the known baseline state of the investigated file.

Establishing the original state is important because a FIM alert identifies that a change occurred, but an analyst must also determine what changed and whether the resulting state introduces additional security risk.

The baseline represented a more restrictive configuration because remote access was explicitly disabled.

---

### 2. Privileged File Modification Event

The monitored configuration file was modified using a privileged command.

![File Modification Event](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-006/02-file-modification-event.png?raw=true)

Observed command:

    sudo sh -c 'echo "remote_access=enabled" >> /home/analyst/wazuh-fim-lab/access-control.conf'

The file was then reviewed and showed the newly introduced value:

    remote_access=enabled

### Analyst Interpretation

The command explicitly targeted the monitored configuration file and executed through `sudo`.

This is more significant than an ordinary user-level file modification because elevated privileges were deliberately used during the activity.

The command proves that the file was intentionally altered, but it does not establish whether the change was authorized.

In a production environment, this activity would need to be compared with change-management records, maintenance windows, administrative authorization, and expected user behavior.

---

### 3. File Metadata After Modification

Filesystem metadata was reviewed after the configuration change.

![File Metadata After Change](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-006/03-file-metadata-after-change.png?raw=true)

Observed information included:

    File: /home/analyst/wazuh-fim-lab/access-control.conf
    Size: 107 bytes
    Owner: analyst
    Group: analyst
    Modify: 2026-08-10 15:07:38 UTC
    Change: 2026-08-10 15:07:38 UTC

### Analyst Interpretation

The increase in file size and updated modification timestamp independently support the conclusion that the file content changed.

The filesystem timestamp provides an endpoint-side temporal reference that can be correlated with authentication activity and the Wazuh FIM event.

File ownership remained unchanged.

This demonstrates that a content modification does not necessarily result in an ownership change.

---

### 4. Wazuh File Integrity Monitoring Alert

Wazuh detected the integrity change affecting the monitored file.

![Wazuh FIM Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-006/04-wazuh-fim-alert.png?raw=true)

Observed alert information:

    Rule Description: Integrity checksum changed.
    Rule ID: 550
    Rule Level: 7
    Agent: ubuntu-agent

The dashboard displayed the FIM event at approximately:

    Aug 10, 2026 @ 18:07:38

### Analyst Interpretation

The Wazuh alert provides SIEM-side confirmation that the integrity of a monitored object changed.

This evidence demonstrates that the endpoint modification generated centralized security telemetry rather than being identified only through manual filesystem inspection.

The dashboard timestamp differed from the endpoint timestamp because the interfaces displayed time using different timezone contexts.

The underlying event timing remained consistent after accounting for the timezone difference.

---

### 5. Authentication and Privileged Execution Context

Linux authentication logs were reviewed to determine which user and command were associated with the file modification.

![Authentication Context](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/case-006/05-authentication-context.png?raw=true)

Relevant evidence included:

    Aug 10 15:07:38 ubuntu-agent sudo[145427]:
    analyst : TTY=pts/0 ; PWD=/home/analyst ; USER=root ;
    COMMAND=/usr/bin/sh -c 'echo "remote_access=enabled" >> /home/analyst/wazuh-fim-lab/access-control.conf'

Observed user:

    analyst

Target execution context:

    root

Privilege mechanism:

    sudo

Target file:

    /home/analyst/wazuh-fim-lab/access-control.conf

### Analyst Interpretation

This evidence directly connects the `analyst` account to the privileged shell command responsible for modifying the monitored file.

The authentication log timestamp:

    15:07:38 UTC

matches the filesystem modification timestamp.

This correlation substantially strengthens attribution because multiple independent evidence sources point to the same activity.

The evidence establishes the execution context but does not prove whether the human operator was authorized or whether the account itself had been compromised.

---

## Timeline Reconstruction

| Time | Evidence Source | Activity |
|---|---|---|
| 15:00:11 UTC | Filesystem | Baseline configuration established |
| 15:07:38 UTC | Authentication Log | `analyst` executes command through `sudo` as root |
| 15:07:38 UTC | File Content | `remote_access=enabled` introduced |
| 15:07:38 UTC | Filesystem Metadata | `access-control.conf` modification timestamp changes |
| 18:07:38 Dashboard Time | Wazuh FIM | Rule 550 reports integrity checksum change |

### Analyst Interpretation

The evidence reconstructs the following sequence:

    Known baseline
            ↓
    Privileged sudo execution
            ↓
    Configuration modification
            ↓
    Filesystem metadata change
            ↓
    Wazuh FIM detection

The matching endpoint timestamps strongly support the conclusion that the observed `sudo` command caused the monitored file modification.

The Wazuh dashboard displayed the event using a timezone offset relative to the endpoint logs. This difference was considered during timeline reconstruction rather than interpreted as a separate event.

---

## Evidence Correlation Summary

| Evidence | Finding | Confidence |
|---|---|---|
| Baseline Content | Remote access initially disabled | High |
| Modified Content | `remote_access=enabled` added | High |
| File Metadata | File changed at 15:07:38 UTC | High |
| Authentication Logs | `analyst` used sudo to execute the modification command | High |
| Privilege Context | Command executed as root | High |
| Wazuh FIM | Integrity checksum change detected | High |
| Authorization Status | No organizational authorization context available | Unknown |
| Malicious Intent | No evidence proving attacker activity | Unconfirmed |

---

## Analysis

The investigation confirmed that a monitored configuration file experienced a deliberate privileged modification.

The file originally contained a restrictive remote-access setting:

`remote_access=disabled`

A sudo-based shell command subsequently introduced:

`remote_access=enabled`

Linux authentication records associated the command with the `analyst` user executing with `root` privileges.

Filesystem metadata showed that the file changed at the same timestamp recorded in the authentication evidence.

Wazuh independently detected the integrity change and generated a Level 7 FIM alert under Rule ID `550`.

The collected evidence therefore demonstrates successful correlation across:

    User Activity
    +
    Privilege Context
    +
    Filesystem Evidence
    +
    SIEM Detection

From a SOC perspective, the activity warrants investigation because privileged configuration modifications can represent:

- Legitimate administration
- Planned maintenance
- Misconfiguration
- Insider misuse
- Compromised account activity
- Unauthorized configuration tampering

The technical evidence establishes what happened and which account executed the command.

However, determining whether the activity was malicious requires organizational context that is not available from telemetry alone.

The event should therefore not be classified as confirmed malicious solely because `sudo` was used or because Wazuh generated an alert.

---

## Confirmed Facts

The investigation confirmed:

- `/home/analyst/wazuh-fim-lab/access-control.conf` was monitored by Wazuh FIM.
- The file initially contained `remote_access=disabled`.
- The file was modified.
- `remote_access=enabled` was added.
- The modification occurred at approximately `15:07:38 UTC`.
- The `analyst` account executed the relevant command.
- `sudo` was used.
- The command executed with `root` privileges.
- The command directly referenced the modified file.
- File metadata changed at the same timestamp.
- Wazuh detected an integrity checksum change.
- Wazuh Rule ID `550` generated the alert.
- The Wazuh alert level was `7`.
- Endpoint, authentication, and SIEM evidence could be correlated.

---

## Unconfirmed / Unknown

The investigation did not establish:

- Whether the configuration change was formally authorized
- Whether an approved change record existed
- Whether the `analyst` account had been compromised
- Whether another person had access to the authenticated session
- Whether the configuration value controlled an active remote-access service
- Whether additional files were modified as part of related activity
- Whether persistence mechanisms were created
- Whether external access occurred
- Whether the activity formed part of a larger attack
- Whether malicious intent existed

These unknowns prevent classification as a confirmed security compromise.

---

## Risk

The primary risk is unauthorized modification of a security-relevant configuration using elevated privileges.

If comparable activity occurred against a production system, potential consequences could include:

- Increased remote-access exposure
- Weakening of access restrictions
- Unauthorized administrative changes
- Privileged-account misuse
- Persistence preparation
- Expansion of attacker access
- Modification of additional security controls

No actual production exposure or compromise was demonstrated in this controlled investigation.

**Risk Rating: Medium**

**Confidence: Medium**

---

## Recommended Next Steps

In a real SOC environment:

1. Verify whether the configuration modification was authorized.
2. Review change-management or maintenance records for the relevant timestamp.
3. Confirm whether the `analyst` user was expected to perform the action.
4. Contact the system owner or administrator if authorization cannot be established.
5. Review surrounding `sudo` activity for additional privileged commands.
6. Search Wazuh FIM events for other files modified within the same time window.
7. Review authentication logs for unusual login activity involving the `analyst` account.
8. Determine whether the configuration change affected an active service.
9. Revert the configuration if the change was unauthorized.
10. Preserve the relevant authentication and FIM evidence.
11. Escalate if additional evidence indicates account compromise or unauthorized privileged activity.

---

## Escalation Decision

**Conditional Escalation Recommended**

The technical activity is confirmed, but malicious intent is not.

Escalation would be justified if:

- No approved change record exists
- The user denies performing the command
- The user was not authorized to modify the configuration
- Additional suspicious privileged commands are identified
- Other security-sensitive files were modified
- Authentication anomalies suggest account compromise
- The modification resulted in unexpected remote access
- Related persistence or post-compromise activity is discovered

If the modification corresponds to verified and approved administrative activity, the event could be closed as authorized activity.

Without authorization context, the most appropriate disposition remains:

**Suspicious / Requires Authorization Validation**

---

## MITRE ATT&CK Mapping

| Technique | Name | Relevance |
|---|---|---|
| T1548.003 | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | The observed command used `sudo` to execute a shell command with root privileges. |

### Mapping Note

The MITRE ATT&CK mapping describes the observed technical behavior and does not imply that an adversary was confirmed.

The use of `sudo` is common legitimate administrative behavior.

It becomes security-relevant when used unexpectedly, by an unauthorized account, or as part of a broader malicious sequence.

No additional ATT&CK techniques were mapped because the evidence did not demonstrate another confirmed adversary behavior such as persistence, credential access, lateral movement, or defense evasion.

---

## Final SOC Summary

A Wazuh File Integrity Monitoring alert was investigated after an integrity checksum change was detected on:

`/home/analyst/wazuh-fim-lab/access-control.conf`

Baseline analysis showed that the file originally contained a restrictive remote-access configuration.

A subsequent privileged command introduced:

`remote_access=enabled`

Filesystem metadata recorded the modification at approximately `15:07:38 UTC`.

Linux authentication logs showed that the `analyst` account executed the modification command through `sudo` with `root` privileges at the same timestamp.

Wazuh independently detected the file-integrity change through Rule ID `550` at Level `7`.

The evidence therefore confirms a privileged configuration modification and successfully attributes the execution context to the `analyst` account.

However, no evidence establishes that the activity was malicious or unauthorized.

The appropriate disposition is **Suspicious / Requires Authorization Validation**, with escalation dependent on change-management verification, user confirmation, and identification of additional suspicious activity.

---

## Lessons Learned

This investigation reinforced several SOC principles:

- File Integrity Monitoring identifies changes but does not determine intent.
- A FIM alert should be correlated with endpoint and authentication evidence.
- Establishing a baseline makes later configuration changes easier to interpret.
- File metadata provides valuable timestamps for timeline reconstruction.
- `sudo` logs can identify the user, target privilege, working directory, and executed command.
- Multiple evidence sources provide stronger attribution than a single SIEM alert.
- Timezone differences must be considered when correlating endpoint and SIEM timestamps.
- Privileged activity is not automatically malicious.
- Authorization context is essential when investigating administrative changes.
- SOC conclusions should separate confirmed technical facts from unknown intent.
- Escalation decisions should be based on evidence and organizational context rather than alert severity alone.

---

## Final Disposition

    Classification: Suspicious / Requires Authorization Validation
    Risk: Medium
    Confidence: Medium
    File Modification: Confirmed
    Security-Relevant Configuration Change: Confirmed
    Wazuh FIM Detection: Confirmed
    Rule ID: 550
    Alert Level: 7
    User Attribution: analyst
    Privileged Execution: Confirmed - sudo to root
    Authorization: Unknown
    Malicious Intent: Not Confirmed
    Account Compromise: Not Confirmed
    Production Impact: Not Demonstrated
    Escalation: Conditional
