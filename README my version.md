# Wazuh-Linux-PAM-FIM-Investigation
## Overview
Linux Pluggable Authentication Modules (PAM) provide a framework for authentication and account management on Linux systems. PAM allows applications and services to use configurable authentication mechanisms without implementing authentication logic independently.

PAM configuration files are generally stored in the `/etc/pam.d/` directory. These files define how authentication-related operations are handled by services such as SSH, login, sudo, and other system components.

File Integrity Monitoring (FIM) helps detect unauthorised or unexpected changes to important system files. It monitors selected files and directories, records their attributes, and identifies modifications such as changes to file contents, permissions, ownership, or timestamps.

When Wazuh FIM monitors the `/etc/pam.d/` directory, it can detect changes to files within that location. Depending on the monitoring configuration, Wazuh can identify file creation, modification, and deletion events.

### How Wazuh FIM Works

Wazuh Syscheck performs file integrity monitoring by collecting file metadata and calculating cryptographic checksums.

When a monitored file changes, Wazuh compares its current attributes with previously recorded information.

Relevant monitored attributes include:

- File size
- Modification timestamp
- File permissions
- File ownership
- MD5 checksum
- SHA-1 checksum
- SHA-256 checksum

A detected modification can generate a Wazuh alert containing the affected file path and the attributes that changed.

### Why PAM File Monitoring Matters

PAM configuration files directly influence authentication behaviour on Linux systems. Unexpected changes may weaken authentication controls or affect how users and services access the system.

For example, an unauthorised modification could potentially:

- Change authentication requirements.
- Weaken password or account restrictions.
- Affect authentication behaviour for SSH or local login.
- Introduce unexpected authentication mechanisms.
- Disrupt legitimate user access.

Monitoring these files helps security analysts identify configuration changes that may require investigation.

### Real-Time Monitoring

Wazuh supports real-time file monitoring on supported operating systems and filesystems.

When real-time monitoring is configured for a directory, changes to monitored files can be detected without waiting for the next scheduled integrity scan.

For Linux PAM monitoring, the following configuration can be added inside the existing `<syscheck>` block in `/var/ossec/etc/ossec.conf`:

    <directories check_all="yes" realtime="yes">/etc/pam.d</directories>

The `check_all` option enables monitoring of the supported file attributes, while `realtime` enables real-time monitoring for the specified directory.

### Understanding FIM Alerts

A FIM alert indicates that a monitored file has changed. The alert may contain information such as:

- Affected file path.
- Type of filesystem event.
- Previous and current file attributes.
- Previous and current checksums.
- Monitoring mode.
- Associated Wazuh rule information.

A checksum difference confirms that the file contents have changed. However, it does not independently establish who performed the modification or whether the activity was malicious.

### SOC Investigation Perspective

When a PAM configuration modification is detected, a SOC analyst should validate the alert and investigate the surrounding activity.

Important investigation steps include:

1. Identify the affected PAM configuration file.
2. Determine which file attributes changed.
3. Compare previous and current checksums.
4. Review Linux authentication and audit logs.
5. Examine relevant process execution and administrative activity.
6. Determine whether the modification was authorised.
7. Assess whether authentication behaviour or system security was affected.

FIM alerts should be treated as evidence of file changes, not automatic proof of compromise.

This lab investigates changes to Linux Pluggable Authentication Modules (PAM) configuration files using Wazuh File Integrity Monitoring (FIM).

PAM manages authentication-related operations on Linux systems. Unexpected modifications to PAM configuration files may affect authentication, account validation, and session management.

The investigation was conducted on Rocky Linux 9.8 using Wazuh Manager, Indexer, and Dashboard. A harmless test file was created inside `/etc/pam.d/`, modified in a controlled manner, and subsequently deleted.

The objective was to determine whether Wazuh could detect the modification and provide useful file integrity evidence.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Rocky Linux 9.8 |
| Hostname | localhost.localdomain |
| Wazuh Version | 4.13.1 |
| Wazuh Manager | Active |
| Wazuh Indexer | Installed |
| Wazuh Dashboard | Installed |
| Wazuh Agent | Not installed |
| Monitoring Directory | `/etc/pam.d/` |
| Test File | `lab130-test` |

**Environment note:** The Wazuh Agent package was not installed. The observed alert contained `agent.id: 000` and `agent.name: localhost.localdomain`, identifying the manager-side local monitoring context rather than a separately enrolled endpoint agent.

## Lab Objectives

The objectives of this lab are to:

- Understand the security importance of Linux PAM configuration files stored under `/etc/pam.d/`.
- Configure Wazuh File Integrity Monitoring (FIM) to monitor the PAM configuration directory.
- Enable real-time monitoring for changes to files within the monitored directory.
- Perform a controlled file modification to generate a FIM integrity alert.
- Examine changes in file size, modification time, and cryptographic checksums.
- Validate the SHA-256 checksum reported by Wazuh against the locally calculated checksum.
- Analyse Wazuh alert details, including rule ID, severity, event type, and MITRE ATT&CK mapping.
- Understand the distinction between detecting a configuration change and confirming malicious activity.
- Document troubleshooting observations, monitoring limitations, and investigation findings.
- Develop a structured SOC investigation approach for identifying unexpected changes to Linux authentication-related configuration files.

## Lab Scenario

A Linux system relies on Pluggable Authentication Modules (PAM) to manage authentication and access-control operations. PAM configuration files are stored under `/etc/pam.d/` and influence how services such as SSH, login, and sudo handle authentication.

Unexpected changes to these files can affect authentication behaviour and potentially weaken security controls. From a SOC perspective, identifying and investigating such modifications is important for maintaining system integrity.

In this lab, a Rocky Linux 9.8 system running Wazuh Manager, Indexer, and Dashboard was used to investigate PAM configuration file modifications through File Integrity Monitoring (FIM).

### Investigation Setup

The existing Wazuh Manager was active, but the Wazuh Agent package was not installed on the system. Therefore, the investigation used the manager-side monitoring context rather than a separately enrolled endpoint agent.

Wazuh FIM was configured to monitor `/etc/pam.d/` with real-time monitoring enabled. A temporary test file named `lab130-test` was created inside the directory to simulate a monitored configuration-file change.

### Controlled Modification

The test file was initially created with a short comment. Its original SHA-256 checksum was recorded before a controlled modification was performed.

Additional text was appended to the file to simulate a configuration change. The file size and SHA-256 checksum were then checked again to establish the differences between the original and modified versions.

Wazuh generated a real-time FIM alert identifying the modified file. The alert included the affected path, changed attributes, previous and current checksums, and the associated integrity-monitoring rule.

### Investigation Focus

The investigation focused on validating whether the Wazuh alert accurately represented the local file modification.

The following evidence was examined:

- File path and modification event reported by Wazuh.
- Changes in file size and modification timestamp.
- Previous and current SHA-256 checksums.
- File ownership and permission details.
- Wazuh rule ID, severity, and MITRE ATT&CK mapping.
- Consistency between the locally calculated checksum and the checksum reported in the alert.

After collecting the evidence, the temporary test file was deleted. Its removal was verified locally. However, a corresponding deletion alert was not captured in the available investigation evidence.

### Investigation Outcome

The controlled modification successfully generated a Wazuh FIM alert, and the reported SHA-256 values matched the locally calculated checksums.

The investigation demonstrated how real-time file integrity monitoring can provide visibility into changes affecting Linux authentication-related configuration directories.

The exercise also highlighted an important SOC distinction: detecting a file modification confirms that a change occurred, but does not establish whether the activity was authorised or malicious. Additional authentication, audit, and process execution evidence would be required to determine the context and potential security impact.

## Investigation Procedure

### 1. Environment Verification

The operating system, hostname, current user, and PAM directory were checked.

    whoami
    hostname
    cat /etc/rocky-release
    ls -ld /etc/pam.d
    ls -l /etc/pam.d/

Rocky Linux 9.8 was confirmed, and the PAM configuration directory was present.

### 2. Wazuh Component Verification

Installed Wazuh packages were checked:

    rpm -qa | grep -i wazuh

The following components were present:

- wazuh-indexer
- wazuh-manager
- wazuh-dashboard

The Wazuh Agent package was absent. The Wazuh Manager service was confirmed active.

### 3. FIM Configuration

The Wazuh configuration file was opened:

    sudo nano /var/ossec/etc/ossec.conf

The following directory monitoring entry was added inside the existing `<syscheck>` configuration:

    <directories check_all="yes" realtime="yes">/etc/pam.d</directories>

The Wazuh services were restarted after the configuration change:

    sudo systemctl restart wazuh-manager
    sudo systemctl restart wazuh-indexer
    sudo systemctl restart wazuh-dashboard

The Manager was subsequently confirmed active.

### 4. Test File Creation

A harmless test file was created inside the PAM directory:

    sudo touch /etc/pam.d/lab130-test
    echo "# Lab 130 - PAM FIM monitoring test" | sudo tee /etc/pam.d/lab130-test

No active PAM configuration files were intentionally modified.

### 5. Baseline Collection

File metadata and the initial SHA-256 hash were collected:

    stat /etc/pam.d/lab130-test
    sha256sum /etc/pam.d/lab130-test

Initial SHA-256:

    38fe5cb55494ee7b2d395acc8fb23b6b5caf1709c596508f1078659c8e4a8c64

The file was owned by root and had permissions `0644`.

### 6. Controlled File Modification

A second harmless comment was appended:

    echo "# Controlled modification for Lab 130" | sudo tee -a /etc/pam.d/lab130-test
    sha256sum /etc/pam.d/lab130-test

Modified SHA-256:

    a53097910575b989a412834d56b93379b6871080c080cc943df31f4a77608eb4

The hash difference confirmed that the file contents had changed.

### 7. Wazuh Alert Analysis

The Wazuh Dashboard displayed a real-time integrity alert for:

`/etc/pam.d/lab130-test`

Alert details:

| Field | Observed Value |
|---|---|
| Agent ID | 000 |
| Agent Name | localhost.localdomain |
| Rule ID | 550 |
| Rule Level | 7 |
| Rule Description | Integrity checksum changed. |
| Decoder | syscheck_integrity_changed |
| FIM Event | modified |
| Monitoring Mode | realtime |
| Changed Attributes | size, mtime, md5, sha1, sha256 |
| File Owner | root |
| File Group | root |
| File Permissions | rw-r--r-- |

The alert included previous and new checksum values. The SHA-256 values matched the hashes collected directly from the test file.

### 8. Test File Deletion

The temporary file was deleted:

    sudo rm /etc/pam.d/lab130-test
    ls -l /etc/pam.d/lab130-test

The command returned a no-such-file error, confirming that the file no longer existed.

A corresponding deletion alert was not included in the collected evidence. Therefore, deletion detection is not claimed as a confirmed result.

## Key Findings

- Rocky Linux 9.8 was used as the investigation platform.
- The Wazuh Manager was active.
- The Wazuh Agent package was not installed.
- The test file modification was detected in real time.
- Wazuh generated Rule ID 550 with severity level 7.
- The alert identified the affected file and changed attributes.
- The SHA-256 values in the alert matched the locally collected values.
- The temporary test file was successfully deleted.
- No active PAM configuration files were intentionally modified.

## Investigation Limitations

FIM identifies changes to monitored files and records associated metadata. It does not independently establish:

- Which person or process performed a modification.
- Whether a modification was malicious or authorised.
- Whether the modification weakened authentication security.
- The intent behind the modification.
- Whether a deletion alert was generated without corresponding alert evidence.

Additional Linux audit records, process telemetry, authentication logs, and administrative activity records would be required to establish attribution and investigate the wider context.

