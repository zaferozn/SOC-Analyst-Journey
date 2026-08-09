# Lab 017 - Wazuh File Integrity Monitoring

## Executive Summary

This lab demonstrates the configuration and investigation of Wazuh File Integrity Monitoring (FIM) on a Linux endpoint.

A controlled directory was configured for real-time monitoring on the `ubuntu-agent` endpoint. A test file was created, modified, and deleted to generate multiple FIM events.

Wazuh successfully detected all three stages of the file lifecycle. The investigation included reviewing event types, rule IDs, alert levels, file metadata, cryptographic hash changes, file size changes, and recorded content differences.

The lab demonstrated how File Integrity Monitoring can provide SOC analysts with visibility into potentially unauthorized changes affecting monitored files and directories.

## Objective

The objective of this lab was to:

- Configure Wazuh File Integrity Monitoring
- Enable real-time monitoring for a controlled Linux directory
- Generate file creation, modification, and deletion events
- Review the resulting Wazuh FIM alerts
- Examine file integrity metadata
- Analyze file size and cryptographic hash changes
- Review recorded content differences
- Practice SOC-style file integrity triage

## Environment / Data Source

Host: `ubuntu-agent`

Tool: Wazuh SIEM

Data Source: Wazuh File Integrity Monitoring / Syscheck

Monitored Directory: `/home/analyst/wazuh-fim-lab`

Test File: `/home/analyst/wazuh-fim-lab/test-file.txt`

Monitoring Mode: `realtime`

## FIM Configuration

A controlled directory was created for the lab:

`mkdir -p ~/wazuh-fim-lab`

The directory path was verified as:

`/home/analyst/wazuh-fim-lab`

The following entry was added inside the existing Wazuh `<syscheck>` configuration:

`<directories check_all="yes" report_changes="yes" realtime="yes">/home/analyst/wazuh-fim-lab</directories>`

This configuration enabled real-time monitoring and file content change reporting for the lab directory.

![FIM Configuration](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-017/01-fim-configuration.png?raw=true)

The Wazuh agent was restarted after the configuration was applied:

`sudo systemctl restart wazuh-agent`

The service status was verified with:

`sudo systemctl status wazuh-agent --no-pager`

The Wazuh agent returned to an `active (running)` state.

## Observed Activity

Three controlled filesystem events were generated during the lab:

1. File creation
2. File modification
3. File deletion

Wazuh detected each action as a separate File Integrity Monitoring event.

All events involved:

`/home/analyst/wazuh-fim-lab/test-file.txt`

The observed lifecycle was:

`File Added → File Modified → File Deleted`

## Evidence

### File Creation

The following command was used to create the monitored file:

`echo "SOC FIM test file" > ~/wazuh-fim-lab/test-file.txt`

The file content was verified with:

`cat ~/wazuh-fim-lab/test-file.txt`

Wazuh generated a File Integrity Monitoring event with the following information:

- Event: `added`
- Rule ID: `554`
- Rule Level: `5`
- Agent: `ubuntu-agent`
- Path: `/home/analyst/wazuh-fim-lab/test-file.txt`

The event confirmed that Wazuh detected the creation of a new file inside the monitored directory.

![File Creation Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-017/02-file-created-alert.png?raw=true)

### File Modification

Additional content was appended to the existing file:

`echo "Additional FIM test content" >> ~/wazuh-fim-lab/test-file.txt`

Wazuh generated another FIM alert for the same file.

The observed information included:

- Event: `modified`
- Rule ID: `550`
- Rule Level: `7`
- Agent: `ubuntu-agent`
- Monitoring Mode: `realtime`
- Path: `/home/analyst/wazuh-fim-lab/test-file.txt`

![File Modification Alert](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-017/03-file-modified-alert.png?raw=true)

### File Modification Details

The detailed modification event provided additional integrity evidence.

Observed fields included:

- `syscheck.event`: `modified`
- `syscheck.mode`: `realtime`
- `syscheck.path`: `/home/analyst/wazuh-fim-lab/test-file.txt`
- `syscheck.uname_after`: `analyst`
- `syscheck.uid_after`: `1000`
- `syscheck.size_before`: `18`
- `syscheck.size_after`: `46`

The following cryptographic hashes changed after the modification:

- MD5
- SHA1
- SHA256

The Wazuh event also recorded a content difference showing the additional line:

`Additional FIM test content`

The file size increased from `18` bytes to `46` bytes.

This evidence confirmed that the monitored file had changed and provided information describing how its integrity state differed from the previous version.

![File Modification Details](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-017/04-file-modification-details.png?raw=true)

### File Deletion

The test file was removed using:

`rm ~/wazuh-fim-lab/test-file.txt`

Wazuh generated a third FIM event with the following information:

- Event: `deleted`
- Rule ID: `553`
- Rule Level: `7`
- Agent: `ubuntu-agent`
- Path: `/home/analyst/wazuh-fim-lab/test-file.txt`

The Wazuh dashboard displayed the complete file lifecycle:

- Added → Rule `554` → Level `5`
- Modified → Rule `550` → Level `7`
- Deleted → Rule `553` → Level `7`

![File Deletion and Full Timeline](https://github.com/zaferozn/SOC-Analyst-Journey/blob/main/screenshots/lab-017/05-file-deleted-and-full-timeline.png?raw=true)

## Analysis

Wazuh File Integrity Monitoring successfully detected the full lifecycle of a file located inside a monitored Linux directory.

The initial file creation generated Rule `554` at Level `5`.

The subsequent modification generated Rule `550` at Level `7`. This event provided more detailed integrity evidence, including changes in file size, modification timestamps, and multiple cryptographic hashes.

The file increased from `18` bytes to `46` bytes after additional content was written.

Because `report_changes="yes"` was enabled, Wazuh also recorded the content difference introduced during the modification.

The final deletion generated Rule `553` at Level `7`.

These events demonstrate how FIM can provide visibility into filesystem activity and help analysts identify potentially unauthorized changes.

However, a File Integrity Monitoring alert alone does not prove malicious activity.

Legitimate administrative actions, software updates, maintenance activity, configuration changes, or authorized user activity can produce similar alerts.

The file metadata can support an investigation, but it should not automatically be interpreted as proof of which person or process performed the modification.

Additional endpoint and operational context would therefore be required before determining whether the activity was suspicious or malicious.

## Risk

Unauthorized file changes may indicate:

- System configuration tampering
- Persistence-related activity
- Unauthorized administrative actions
- Application compromise
- Defense evasion
- Modification of security-sensitive files
- Removal of important files or security controls

The actual security impact depends on the affected file, surrounding endpoint activity, sensitivity of the resource, and whether the change was authorized.

## Recommended Next Steps

- Determine whether the file change was authorized
- Identify whether the affected file is security-sensitive
- Review authentication activity around the same timestamp
- Investigate privileged activity on the endpoint
- Review additional FIM events within the same time window
- Determine whether other files were created, modified, or deleted
- Review available user and process context
- Compare the event with approved maintenance or change-management activity
- Correlate the FIM alert with other endpoint and SIEM events
- Escalate unexplained changes affecting sensitive resources

## MITRE ATT&CK Mapping

No specific MITRE ATT&CK technique was assigned to this lab because the activity was intentionally generated in a controlled environment.

A generic file creation, modification, or deletion event should not automatically be mapped to a specific ATT&CK technique.

ATT&CK mapping should be based on additional evidence demonstrating the purpose and context of the activity.

## Final SOC Summary

Wazuh File Integrity Monitoring detected controlled file creation, modification, and deletion activity on the `ubuntu-agent` Linux endpoint. The modification event provided detailed integrity evidence, including changes in file size, MD5, SHA1, and SHA256 hashes, modification timestamps, and recorded file content differences. The complete sequence demonstrated how Wazuh FIM can provide visibility into filesystem changes. Although the activity was authorized for laboratory purposes, similar events in a production environment would require correlation with authentication, privileged activity, process context, and authorized change records before determining whether escalation was necessary.

## Lessons Learned

This lab demonstrated how Wazuh File Integrity Monitoring detects filesystem changes in real time.

I learned how to identify and distinguish file creation, modification, and deletion events and how to analyze relevant FIM fields.

I also learned how changes in file size, cryptographic hashes, modification timestamps, and file content can provide evidence that a monitored resource has been altered.

Most importantly, the lab reinforced that a FIM alert confirms that a change occurred but does not independently establish whether the activity was malicious. Effective SOC triage requires additional contextual evidence before reaching a final disposition.
