# Lab 145 – Recovery Readiness Evidence Assessment

## Overview

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

## Skills Demonstrated

- Windows DFIR evidence collection
- PowerShell-based endpoint assessment
- Windows Defender and Firewall analysis
- Backup resource inventory
- Process and service investigation
- Network connection assessment
- Persistence review
- Wazuh and Sysmon event analysis
- SHA256 evidence integrity documentation
- Recovery readiness reporting
