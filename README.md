# Wazuh Linux PAM FIM Investigation

## Overview

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

## Objectives

- Understand PAM and its role in Linux authentication.
- Identify the purpose of the `/etc/pam.d/` directory.
- Configure Wazuh FIM to monitor the PAM directory.
- Perform controlled file creation and modification.
- Compare SHA-256 hashes before and after modification.
- Examine Wazuh integrity alerts and changed attributes.
- Understand the limitations of FIM evidence.

## Investigation Scenario

A SOC analyst needs to identify unexpected changes to Linux PAM configuration files.

Since modifications to authentication-related files can affect system security, FIM is used to identify changes and provide file integrity evidence.

Instead of modifying active PAM configuration files, an isolated test file named `lab130-test` is used. The file is created, its metadata and SHA-256 hash are recorded, and a harmless comment is appended.

Wazuh generates a real-time integrity alert identifying the modified file and its changed attributes.

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

## Conclusion

The investigation demonstrated successful real-time detection of a controlled modification to a file inside the Linux PAM configuration directory.

Wazuh recorded the affected path, changed attributes, and previous and new checksums. The alert was validated against locally collected SHA-256 evidence.

The exercise also demonstrated why file integrity alerts must be interpreted carefully. A detected modification confirms a file change, but does not independently prove malicious activity.

## Safety Notes

- Do not modify active PAM files such as `system-auth`, `password-auth`, `sshd`, or `sudo` for this exercise.
- Use an isolated test file for controlled monitoring.
- Do not interpret every integrity alert as a confirmed security incident.
- Preserve relevant file metadata and alert evidence during investigations.
