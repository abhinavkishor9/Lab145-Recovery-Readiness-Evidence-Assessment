# Lab145-Recovery-Readiness-Evidence-Assessment
## Overview
### 1. Introduction

Recovery readiness is the process of evaluating whether an endpoint has sufficient evidence, recovery resources, and security conditions to proceed towards recovery after a security incident.

After containment and investigation, the response team must determine whether restoring a system is appropriate.

Restoring too early can reintroduce compromised files, persistence mechanisms, unauthorized configurations, or other elements associated with the original incident.

A recovery readiness assessment helps identify these risks before recovery begins.

### 2. What Is Recovery Readiness?

Recovery readiness is an evidence-based assessment of the endpoint and its recovery dependencies.

It considers:

- Current endpoint configuration and operational condition.
- Availability and integrity of recovery resources.
- Status of endpoint security controls.
- Running processes and services.
- Network activity and connectivity.
- Persistence-related configuration.
- Outstanding investigation findings.
- Recovery approval and authorization.

The analyst must distinguish between information that has been collected, information that has been verified, and information that remains unknown.

### 3. Recovery Readiness vs. Recovery Execution

These activities have different purposes.

| Stage | Purpose |
|---|---|
| Recovery readiness | Determine whether sufficient evidence and resources exist to proceed |
| Recovery execution | Restore systems, data, or configurations |
| Recovery validation | Confirm that restored systems operate correctly and securely |
| Return to service | Authorize the system to resume normal operations |

Completing an evidence collection exercise does not mean that the endpoint has completed recovery.

### 4. Backup Availability and Integrity

A backup location may exist without containing a usable backup.

A meaningful backup assessment should establish:

1. Whether an appropriate backup exists.
2. Whether the backup belongs to the affected system.
3. When the backup was created.
4. Whether its integrity can be verified.
5. Whether it is protected against unauthorized modification.
6. Whether restoration can be completed successfully.
7. Whether the restored system can be validated.

For example, identifying a directory named `Rocky-Linux-Backup` does not establish that a Windows endpoint can be restored from it.

Backup validation must consider the actual recovery requirement.

### 5. Endpoint Security Posture

Security controls should be reviewed before recovery.

Important areas include:

- Antivirus service and protection status.
- Real-time protection.
- Behavior monitoring.
- Network inspection.
- Windows Firewall profiles.
- Security configuration changes.

An antivirus service reporting as enabled does not establish that every protection component is operational.

Similarly, an enabled firewall profile does not independently prove that the effective firewall policy is appropriate.

### 6. Process, Service, and Network Assessment

The endpoint's current operational state should be documented.

#### Processes and Services

Review:

- Process names and executable paths.
- Parent-child process relationships.
- Command-line arguments.
- Service execution paths.
- Service accounts.
- Startup configuration.

Unexpected entries require investigation and contextual validation.

#### Network Connections

Review:

- Local and remote addresses.
- Ports and connection states.
- Owning process IDs.
- Expected application communications.
- Connections to monitoring infrastructure.

A listening or established connection is not automatically malicious.

Network evidence should be attributed to processes and compared against known-good behavior.

### 7. Persistence Assessment

Persistence mechanisms can allow an attacker to regain access after apparent cleanup.

Common locations include:

- Scheduled tasks.
- Startup applications.
- Windows services.
- Registry Run keys.
- WMI event subscriptions.
- Logon scripts.

Collecting these locations is only the beginning of the investigation.

Analysts must examine task actions, executable paths, signatures, timestamps, ownership, and expected behavior.

An unfamiliar name alone is insufficient to classify an entry as malicious.

### 8. Wazuh and Windows Event Evidence

Wazuh can help correlate endpoint activity through collected Windows and Sysmon events.

Examples include:

| Event ID | Evidence |
|---|---|
| Sysmon 1 | Process creation |
| Sysmon 3 | Network connection |
| Sysmon 11 | File creation |
| Sysmon 13 | Registry value modification |
| Security 4624 | Successful logon |
| Security 4625 | Failed logon |
| Security 4697 | Service installation |
| Security 4698 | Scheduled task creation |
| Security 1102 | Security audit log cleared |
| Defender 1116 | Malware detection |
| Defender 1117 | Remediation action |

Event availability depends on Windows audit configuration, Sysmon configuration, collection settings, and indexing.

An empty query result does not prove that the corresponding activity never occurred.

### 9. Evidence Integrity

Evidence files should be protected against accidental or unauthorized modification.

SHA256 hashing provides a way to record file content at a particular point in time.

A hash manifest can help detect subsequent changes to collected assessment files.

However, matching hashes do not prove that the original source was authentic or that the endpoint was uncompromised before collection.

Evidence integrity must also consider collection procedures, timestamps, access controls, and chain of custody.

### 10. Recovery Decision

A recovery readiness assessment should identify:

- Verified conditions.
- Outstanding investigation requirements.
- Missing recovery resources.
- Security control gaps.
- Unresolved persistence concerns.
- Required approvals.

The final assessment should communicate whether the available evidence supports proceeding, whether additional validation is required, or whether recovery should remain on hold.

### 11. Core Principle

**Recovery readiness is demonstrated through evidence, not assumptions.**

A stopped process, an enabled antivirus service, or an available backup directory does not independently establish that an endpoint is safe to restore.

The analyst must evaluate the complete recovery context and document unresolved risks before recommending a return to service.

This lab focuses on assessing whether a Windows endpoint has sufficient evidence and recovery resources to proceed towards recovery after a security incident.

Recovery readiness requires more than confirming that a suspicious process has stopped or that security controls are enabled. Analysts must evaluate backup availability, endpoint protection, system configuration, network activity, persistence locations, and unresolved investigation findings.

PowerShell was used to collect endpoint evidence, while Wazuh was used to review Windows and Sysmon telemetry.

The assessment identifies recovery dependencies and outstanding risks. It does not execute system recovery or declare the endpoint clean.

## Lab Environment

| Component | Details |
|---|---|
| Endpoint | DESKTOP-9MMM37V |
| Operating System | Windows 11 Pro |
| OS Build | 26200 |
| PowerShell | 7.7.0 |
| Endpoint Monitoring | Wazuh Agent |
| Telemetry | Sysmon and Windows Event Channels |
| Workspace | C:\DFIR-Lab145 |
| Evidence ID | EVD-145-001 |

## Lab Objectives

### Primary Objective

Evaluate the recovery readiness of a Windows endpoint by collecting and reviewing evidence related to system state, backup resources, security controls, network activity, and persistence.

The exercise focuses on evidence quality and identifying recovery blockers.

### Technical Objectives

#### 1. Establish Endpoint Identity

- Record hostname, operating system, build, user context, and PowerShell version.
- Preserve the assessment timestamp.
- Create a structured evidence workspace.

#### 2. Document Firewall Configuration

- Capture Domain, Private, and Public firewall profile states.
- Record reported default inbound and outbound actions.
- Distinguish profile status from effective firewall policy.

#### 3. Inventory Recovery Resources

- Identify available storage volumes.
- Check candidate backup locations.
- Distinguish an existing directory from a validated recovery source.
- Identify the need for backup integrity and restoration testing.

#### 4. Evaluate Endpoint Protection

- Collect Microsoft Defender operational status.
- Review antivirus and real-time protection.
- Review behavior monitoring and network inspection.
- Identify security configuration gaps requiring investigation.

#### 5. Collect Operational-State Evidence

- Export process and service inventories.
- Capture current TCP connections.
- Record network configuration.
- Preserve collected outputs for subsequent investigation.

#### 6. Identify Persistence Review Requirements

- Export scheduled tasks.
- Export startup entries.
- Identify entries requiring validation of origin and execution behavior.

#### 7. Correlate Endpoint Telemetry

- Review relevant Wazuh archive events.
- Interpret Sysmon file creation evidence in system context.
- Identify additional telemetry required for correlation.
- Avoid classifying an event as malicious without supporting evidence.

#### 8. Preserve Assessment Integrity

- Generate SHA256 hashes for assessment outputs.
- Avoid hashing the output manifest while it is being written.
- Maintain an identifiable evidence structure.

#### 9. Document Recovery Blockers

- Separate collected evidence from verified findings.
- Identify missing backup validation.
- Record endpoint protection gaps.
- Document unresolved persistence and network investigation requirements.
- Identify recovery authorization as a separate decision.

### Expected Outcome

Produce a structured recovery readiness assessment identifying:

- Known endpoint conditions.
- Available recovery evidence.
- Security posture concerns.
- Outstanding investigation requirements.
- Recovery dependencies and limitations.

### Scope Boundaries

This exercise does not:

- Restore Windows or recover user data.
- Modify or overwrite backup resources.
- Remediate Defender configuration.
- Remove scheduled tasks or services.
- Establish complete threat eradication.
- Authorize production recovery or return to service.

## Investigation Workflow

1. Created the lab workspace and evidence directories.
2. Captured system identity and operating system information.
3. Collected Windows Firewall profile configuration.
4. Inventoried storage volumes and candidate backup locations.
5. Collected Microsoft Defender protection status.
6. Exported process, service, network, scheduled-task, and startup inventories.
7. Created a recovery readiness assessment and summary.
8. Generated SHA256 hashes for assessment evidence.
9. Reviewed collected Wazuh archive events using a combined KQL query.

## Key Findings

### 1. System Baseline

The endpoint was identified as:

- Hostname: DESKTOP-9MMM37V
- Operating System: Windows 11 Pro
- Build: 26200
- PowerShell: 7.7.0
- Assessment date: 25 November 2026

### 2. Windows Firewall

The Domain, Private, and Public firewall profiles were enabled.

The collected default inbound and outbound action fields were reported as `NotConfigured`. These values require interpretation in the context of effective firewall policy.

### 3. Recovery Resources

The inventory identified the following:

- `D:\Rocky-Linux-Backup` existed.
- The checked `C:\WindowsImageBackup` and `D:\WindowsImageBackup` locations were absent.

The existing Rocky Linux backup directory does not establish that a valid Windows endpoint recovery source is available.

Backup integrity and restoration capability were not tested.

### 4. Microsoft Defender

The collected status reported:

- Antivirus service enabled.
- Antivirus enabled.
- Real-time protection disabled.
- Behavior monitoring disabled.
- IOAV protection disabled.
- Network Inspection System disabled.

These protection gaps require investigation before recovery readiness can be established.

### 5. System and Persistence Assessment

Process, service, network, scheduled-task, and startup inventories were exported.

The scheduled-task inventory included entries under `\R@1n-KMS`. Their origin and execution behavior require validation. Their names alone do not establish malicious persistence.

### 6. Wazuh Event Analysis

A supplied Wazuh archive record showed Sysmon Event ID 11.

| Field | Observed value |
|---|---|
| Event ID | 11 |
| Provider | Microsoft-Windows-Sysmon |
| Process | powershell.exe |
| Process ID | 23896 |
| User | NT AUTHORITY\SYSTEM |
| Target | C:\Windows\SystemTemp\__PSScriptPolicyTest_nrid4a0r.1xf.ps1 |
| Agent | DESKTOP-9MMM37V |
| Event time | 25 November 2026, approximately 10:45 IST |

The filename is consistent with a PowerShell script-policy test artifact. The event establishes file creation, not malicious intent.

## Evidence Directory Structure

```text
C:\DFIR-Lab145
│
├── baseline
│   ├── system-baseline.csv
│   └── firewall-baseline.csv
│
├── recovery-resources
│   ├── volume-inventory.csv
│   └── backup-location-inventory.csv
│
├── security-posture
│   └── defender-status.csv
│
├── system-assessment
│   ├── process-inventory.csv
│   ├── service-inventory.csv
│   ├── network-connections.csv
│   ├── network-configuration.txt
│   ├── scheduled-tasks.csv
│   └── startup-entries.csv
│
├── evidence
│   ├── recovery-readiness-assessment.csv
│   ├── recovery-readiness-summary.csv
│   └── evidence-hashes.csv
│
└── logs
```

## Assessment Outcome

| Assessment area | Status |
|---|---|
| System baseline | Collected |
| Firewall configuration | Collected |
| Backup availability | Requires validation |
| Defender protection | Gaps identified |
| Processes and services | Requires review |
| Network state | Requires review |
| Persistence locations | Requires review |
| Recovery authorization | Not assessed |

**Overall assessment: Further validation required.**

## Limitations

- Backup integrity and restorability were not tested.
- Threat eradication was not established.
- Defender protection gaps were not remediated.
- Network connections were not comprehensively attributed to processes.
- Scheduled tasks and startup entries were not fully investigated.
- The supplied Sysmon event does not establish malicious activity.
- Recovery execution and return-to-service authorization were not performed.

