# Lab 145 – Investigation Timeline

**Date:** 25 November 2026

**Endpoint:** DESKTOP-9MMM37V

**Evidence ID:** EVD-145-001

All displayed local times use the +05:30 timezone unless otherwise stated.

| Time | Activity | Observation |
|---|---|---|
| 10:18:46 | System baseline captured | Windows 11 Pro, build 26200, PowerShell 7.7.0 |
| Not recorded | Firewall inventory collected | Domain, Private, and Public profiles enabled |
| Not recorded | Volume inventory collected | Storage volumes inventoried |
| Not recorded | Backup locations checked | Rocky Linux backup directory existed; checked WindowsImageBackup locations absent |
| Not recorded | Defender status collected | Antivirus enabled; several protection components reported disabled |
| Not recorded | Process and service inventories exported | Endpoint process and service snapshots collected |
| Not recorded | Network assessment performed | TCP connections and network configuration exported |
| Not recorded | Persistence inventories exported | Scheduled tasks and startup entries collected |
| 10:45:32.159 | Wazuh archive event observed | Sysmon Event ID 11 recorded PowerShell temporary file creation |

## Wazuh Event Time

The Wazuh record displayed:

`Nov 25, 2026 @ 10:45:32.159`

The underlying Sysmon event data contained:

`2026-11-25 05:15:31.235 UTC`

This corresponds to approximately 10:45:31 IST.

## Evidence Integrity

A SHA256 manifest was generated for the assessment evidence.

The initial hash command encountered a file-access conflict because the output manifest was located inside the directory being enumerated.

The issue was addressed by excluding the manifest from the input file list.

## Timeline Limitations

Exact timestamps were not preserved for every PowerShell collection activity. Those activities are therefore marked as `Not recorded`.

The Wazuh event is a separately observed telemetry record. Its presence does not establish that it was included in an earlier assessment summary.

No backup restoration, threat eradication validation, recovery execution, or return-to-service approval was performed.
