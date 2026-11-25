# Investigation Notes
## 1. Investigation Objective

Assess whether the Windows endpoint has sufficient evidence and recovery resources to proceed towards recovery following a security incident.

The investigation focused on collecting system information, reviewing security controls, identifying recovery resources, and documenting unresolved risks.

**Evidence ID:** EVD-145-001

**Endpoint:** DESKTOP-9MMM37V

**Assessment date:** 25 November 2026

## 2. System Baseline

The endpoint baseline was captured at:

`2026-11-25 10:18:46 +05:30`

| Attribute | Value |
|---|---|
| Hostname | DESKTOP-9MMM37V |
| Username | DESKTOP-9MMM37V\Dell |
| Operating System | Windows 11 Pro |
| OS Version | 10.0.26200 |
| Build Number | 26200 |
| PowerShell | 7.7.0 |

The information was exported to:

`baseline\system-baseline.csv`

## 3. Windows Firewall Assessment

Firewall profile information was collected.

The Domain, Private, and Public profiles were reported as enabled.

The default inbound and outbound action fields were reported as `NotConfigured`.

This confirms the reported profile configuration but does not independently establish the effective firewall policy.

**Evidence:**

`baseline\firewall-baseline.csv`

## 4. Recovery Resource Assessment

Storage volumes and candidate backup locations were inventoried.

The following backup-location results were observed:

| Location | Result |
|---|---|
| D:\Rocky-Linux-Backup | Exists |
| C:\WindowsImageBackup | Not found |
| D:\WindowsImageBackup | Not found |

The existing Rocky Linux backup directory is not evidence of a valid Windows endpoint backup.

No backup restoration or integrity test was performed.

**Evidence:**

- `recovery-resources\volume-inventory.csv`
- `recovery-resources\backup-location-inventory.csv`

## 5. Microsoft Defender Assessment

The Defender status collection reported:

| Setting | Result |
|---|---|
| AMServiceEnabled | True |
| AntivirusEnabled | True |
| RealTimeProtectionEnabled | False |
| BehaviorMonitorEnabled | False |
| IoavProtectionEnabled | False |
| NISEnabled | False |

The antivirus service and antivirus were reported as enabled, while several protection components were reported as disabled.

This represents a significant security posture concern.

The reason for these settings was not established during this assessment. Further investigation is required before recovery readiness can be confirmed.

**Evidence:**

`security-posture\defender-status.csv`

## 6. Process and Service Assessment

Process and service inventories were exported.

The service inventory included the Wazuh agent and Sysmon64 in a Running state.

The inventories provide a snapshot of endpoint activity and configuration.

No complete process or service threat assessment was performed. Running state and service names alone are insufficient to determine whether an endpoint is compromised.

**Evidence:**

- `system-assessment\process-inventory.csv`
- `system-assessment\service-inventory.csv`

## 7. Network Assessment

A TCP connection snapshot and network configuration export were collected.

The TCP output contained listening, bound, loopback, and established connections.

The Wazuh agent connection to the manager was visible:

`192.168.174.1:9721 -> 192.168.174.128:1514`

The connection is consistent with the endpoint communicating with the Wazuh manager.

Other connections were not comprehensively attributed to processes or compared against a known-good baseline.

No conclusion about malicious network activity can be made from the snapshot alone.

**Evidence:**

- `system-assessment\network-connections.csv`
- `system-assessment\network-configuration.txt`

## 8. Persistence Assessment

Scheduled tasks and startup entries were exported.

The scheduled-task inventory included entries under:

`\R@1n-KMS`

Examples included:

- Office15ProPlus
- Windows100Professional

These entries require validation of their origin, task actions, executable paths, signatures, timestamps, and expected purpose.

Their presence alone does not establish malicious persistence.

The inventories were collected but not comprehensively reviewed.

**Evidence:**

- `system-assessment\scheduled-tasks.csv`
- `system-assessment\startup-entries.csv`

## 9. Wazuh Event Analysis

A combined KQL query was used to retrieve selected Defender, Sysmon, and Windows Security events for the endpoint.

The supplied result contained a Sysmon Event ID 11 record.

### Event Details

| Attribute | Value |
|---|---|
| Timestamp | Nov 25, 2026 @ 10:45:32.159 |
| Index | wazuh-archives-4.x-2026.11.25 |
| Agent ID | 001 |
| Agent | DESKTOP-9MMM37V |
| Event ID | 11 |
| Channel | Microsoft-Windows-Sysmon/Operational |
| Process | powershell.exe |
| Process ID | 23896 |
| User | NT AUTHORITY\SYSTEM |
| Target filename | C:\Windows\SystemTemp\__PSScriptPolicyTest_nrid4a0r.1xf.ps1 |
| Event Record ID | 2350785 |

### Interpretation

Sysmon Event ID 11 records file creation.

The target filename is consistent with a PowerShell script-policy test artifact. However, the event alone does not establish the exact reason for creation or prove that the file is benign.

Further correlation should include:

- Sysmon Event ID 1 for process creation.
- Process command-line information.
- Parent process details.
- Relevant PowerShell operational events.
- Nearby Defender events.

**Finding:** File creation was observed. Malicious activity was not established by this event.

## 10. Evidence Integrity

A SHA256 manifest was generated for the assessment evidence.

The manifest included hashes for:

- `recovery-readiness-assessment.csv`
- `recovery-readiness-summary.csv`

During generation, a file-access conflict occurred because the output manifest was located in the directory being enumerated for hashing.

The issue was addressed by excluding the output manifest from the input file list.

The generated hashes provide a reference for detecting subsequent changes to those files. They do not establish the authenticity of the original endpoint or backup data.

**Evidence:**

`evidence\evidence-hashes.csv`

## 11. Overall Assessment

The assessment identified several outstanding requirements.

| Area | Finding |
|---|---|
| System baseline | Collected |
| Firewall | Profiles enabled; effective policy requires contextual review |
| Backup | Appropriate Windows recovery source not validated |
| Defender | Several protection components reported disabled |
| Processes and services | Inventories collected; review outstanding |
| Network | Snapshot collected; attribution incomplete |
| Persistence | Scheduled tasks and startup entries require review |
| Wazuh | Sysmon file creation event observed |
| Recovery authorization | Not assessed |

### Outstanding Requirements

1. Identify and validate an appropriate Windows endpoint backup.
2. Establish backup integrity, recency, and restorability.
3. Investigate the reported Defender protection gaps.
4. Review scheduled tasks and startup entries.
5. Attribute relevant network connections to their owning processes.
6. Correlate the PowerShell file creation event with surrounding telemetry.
7. Establish whether incident eradication requirements have been satisfied.
8. Obtain appropriate recovery authorization.

