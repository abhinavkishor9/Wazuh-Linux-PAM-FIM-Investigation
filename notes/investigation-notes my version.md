# Investigation Notes 
## 1. Environment Observations

| Item | Observation |
|---|---|
| Operating System | Rocky Linux 9.8 |
| Hostname | localhost.localdomain |
| Current User | root |
| PAM Directory | `/etc/pam.d/` |
| Wazuh Version | 4.13.1 |
| Manager Service | Active |
| Indexer | Installed |
| Dashboard | Installed |
| Agent Package | Not installed |

The initial service check returned:

    Unit wazuh-agent.service could not be found.

The package inventory confirmed that the machine had Wazuh Manager, Indexer, and Dashboard installed, but not the Wazuh Agent.

The FIM alert was associated with:

    agent.id: 000
    agent.name: localhost.localdomain

This represents the manager-side local monitoring context rather than a separately enrolled endpoint agent.

## 2. FIM Configuration

The Wazuh configuration file was opened:

    sudo nano /var/ossec/etc/ossec.conf

The following entry was added inside the existing `syscheck` configuration:

    <directories check_all="yes" realtime="yes">/etc/pam.d</directories>

The Wazuh Manager, Indexer, and Dashboard services were restarted.

The Manager was subsequently confirmed active.

## 3. Test File Baseline

Test file:

    /etc/pam.d/lab130-test

Initial content:

    # Lab 130 - PAM FIM monitoring test

The following commands were used to collect metadata and the initial hash:

    stat /etc/pam.d/lab130-test
    sha256sum /etc/pam.d/lab130-test

Initial metadata:

| Attribute | Value |
|---|---|
| File Type | Regular file |
| Owner | root |
| Group | root |
| Permissions | 0644 |
| Size | 36 bytes |

Initial SHA-256:

    38fe5cb55494ee7b2d395acc8fb23b6b5caf1709c596508f1078659c8e4a8c64

## 4. Controlled Modification

The following comment was appended:

    # Controlled modification for Lab 130

Command:

    echo "# Controlled modification for Lab 130" | sudo tee -a /etc/pam.d/lab130-test

The modified file size was 74 bytes.

Modified SHA-256:

    a53097910575b989a412834d56b93379b6871080c080cc943df31f4a77608eb4

The changed hash confirmed that the file contents were different from the baseline.

## 5. Wazuh Alert Analysis

The Wazuh Dashboard displayed the following alert:

| Field | Value |
|---|---|
| Index | wazuh-alerts-4.x-2026.11.07 |
| Timestamp | Nov 7, 2026 @ 09:55:11.893 |
| Agent ID | 000 |
| Agent Name | localhost.localdomain |
| Rule ID | 550 |
| Rule Level | 7 |
| Rule Description | Integrity checksum changed. |
| Decoder | syscheck_integrity_changed |
| Event | modified |
| Monitoring Mode | realtime |
| File Path | `/etc/pam.d/lab130-test` |

### Changed Attributes

    size, mtime, md5, sha1, sha256

### File Size Comparison

| Attribute | Before | After |
|---|---:|---:|
| Size | 36 bytes | 74 bytes |

### SHA-256 Comparison

| Stage | SHA-256 |
|---|---|
| Before modification | `38fe5cb55494ee7b2d395acc8fb23b6b5caf1709c596508f1078659c8e4a8c64` |
| After modification | `a53097910575b989a412834d56b93379b6871080c080cc943df31f4a77608eb4` |

The alert's previous and new SHA-256 values matched the hashes collected locally.

### Ownership and Permissions

The alert recorded:

- Owner: root
- Group: root
- Permissions: `rw-r--r--`
- UID: 0
- GID: 0

No ownership or permission change was reported in the supplied alert.

## 6. Deletion Verification

The temporary test file was removed:

    sudo rm /etc/pam.d/lab130-test

The following check was performed:

    ls -l /etc/pam.d/lab130-test

Output:

    ls: cannot access '/etc/pam.d/lab130-test': No such file or directory

This confirms that the test file was deleted.

However, no corresponding deletion alert was included in the collected evidence. Therefore, the deletion operation is confirmed, but Wazuh deletion detection remains unverified.

## 7. Evidence Assessment

### Confirmed

- Rocky Linux 9.8 was running.
- Wazuh Manager was active.
- Wazuh Agent package was not installed.
- The test file was modified.
- Wazuh generated a real-time FIM modification alert.
- The alert identified the correct file path.
- Previous and new SHA-256 values matched local command output.
- The temporary test file was subsequently removed.

### Not Established

- The identity of the person or process responsible for the modification beyond the controlled lab activity.
- Whether any active PAM configuration was weakened.
- Whether the deletion generated a Wazuh alert.
- Whether any authentication or privilege-escalation event occurred.

## 8. SOC Analyst Assessment

The observed alert was consistent with the controlled modification performed during the lab.

The change was detected in real time, and the alert provided file-integrity details that could be validated against the local SHA-256 evidence.

The event should be classified as a confirmed file modification, not a confirmed malicious PAM configuration change.

In a real incident, further investigation would require process execution telemetry, Linux audit records, administrative activity, and authentication logs to establish attribution and determine whether the change affected system security.

#
