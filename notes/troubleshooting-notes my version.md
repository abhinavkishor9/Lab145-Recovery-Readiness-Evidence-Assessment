# Troubleshooting Notes

## 1. PowerShell 7.7 – NoTypeInformation Warning

### Observation

CSV export commands displayed:

`WARNING: Parameter 'NoTypeInformation' is obsolete and has no effect.`

### Explanation

PowerShell 7.7 omits type information by default when using Export-Csv.

### Resolution

Remove the obsolete parameter.

```powershell
$SystemInfo | Export-Csv "$BasePath\baseline\system-baseline.csv"
```

The warning does not indicate that the CSV export failed.

## 2. Evidence Hash Manifest File-Access Conflict

### Observation

The following command returned a file-access error:

```powershell
Get-ChildItem "$BasePath\evidence" -File |
    Get-FileHash -Algorithm SHA256 |
    Export-Csv "$BasePath\evidence\evidence-hashes.csv" -NoTypeInformation
```

Error:

`The process cannot access the file because it is being used by another process.`

### Explanation

The output manifest was being written inside the directory being enumerated for hashing.

The output file could be included in the input file enumeration.

### Resolution

Exclude the output manifest explicitly.

```powershell
$HashFile = "$BasePath\evidence\evidence-hashes.csv"

Get-ChildItem "$BasePath\evidence" -File |
    Where-Object { $_.FullName -ne $HashFile } |
    Get-FileHash -Algorithm SHA256 |
    Export-Csv $HashFile
```

Verify the manifest:

```powershell
Import-Csv $HashFile | Format-Table -AutoSize
```

### Lesson

When generating a hash manifest inside the directory being hashed, exclude the manifest itself from the input files.

## 3. Defender Status Interpretation

### Observation

The collected Defender status reported:

- AntivirusEnabled: True
- AMServiceEnabled: True
- RealTimeProtectionEnabled: False
- BehaviorMonitorEnabled: False
- IoavProtectionEnabled: False
- NISEnabled: False

### Explanation

An enabled antivirus service does not mean that every protection component is enabled.

### Investigation Requirement

Determine why these protection settings were reported as disabled.

Review applicable security policies, Defender configuration, and relevant operational events.

Do not change Defender settings as part of this evidence-collection exercise.

## 4. Firewall Profile Interpretation

### Observation

All three firewall profiles were enabled, while the default inbound and outbound action fields were reported as `NotConfigured`.

### Explanation

The collected values describe the reported profile configuration.

They should not be treated as proof that the effective firewall policy permits or blocks all traffic.

### Investigation Requirement

Review effective firewall policy and applicable Group Policy settings before making a traffic-enforcement determination.

## 5. Backup Location Interpretation

### Observation

The inventory reported:

`D:\Rocky-Linux-Backup` – Exists: True

The checked WindowsImageBackup locations were absent.

### Explanation

The existing directory is associated with a Rocky Linux backup. Its existence does not establish that a usable Windows endpoint backup is available.

### Investigation Requirement

Identify an appropriate Windows recovery source and validate its integrity, recency, and restoration capability.

Do not modify or overwrite the existing backup directory during this assessment.

## 6. Network Snapshot Interpretation

### Observation

The TCP snapshot contained Bound, Listen, Established, and loopback connections.

The Wazuh agent connection to `192.168.174.128:1514` was also visible.

### Explanation

These connection states are not inherently suspicious.

### Investigation Requirement

Correlate connections with owning process IDs and compare activity against a known-good baseline.

Do not classify every established external connection as malicious.

## 7. Scheduled Task Review

### Observation

The scheduled-task inventory included entries under:

`\R@1n-KMS`

Examples included:

- Office15ProPlus
- Windows100Professional

### Explanation

Task names and paths can provide investigation leads, but they do not establish malicious persistence by themselves.

### Investigation Requirement

Review:

- Task actions.
- Executable paths.
- Digital signatures.
- Creation and modification times.
- Task ownership.
- Expected software purpose.

Do not delete unfamiliar tasks without validation.

## 8. Wazuh Sysmon Event ID 11

### Observation

The supplied event showed:

- Process: `powershell.exe`
- User: `NT AUTHORITY\SYSTEM`
- Target: `C:\Windows\SystemTemp\__PSScriptPolicyTest_nrid4a0r.1xf.ps1`

### Explanation

Sysmon Event ID 11 records file creation.

The filename is consistent with a PowerShell script-policy test artifact, but the event does not independently establish the reason for creation or malicious intent.

### Investigation Requirement

Correlate the event with:

- Sysmon Event ID 1.
- Process command-line details.
- Parent process information.
- PowerShell operational logs.
- Defender events around the same time.

