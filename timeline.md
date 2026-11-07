
# Investigation Timeline – Linux PAM Configuration FIM

## Overview

This timeline records the sequence of activities performed during the Wazuh Linux PAM configuration monitoring investigation on Rocky Linux 9.8.

## Timeline of Activities

| Sequence | Activity | Details | Result |
|---|---|---|---|
| 1 | Environment verification | Checked hostname, operating system, current user, and `/etc/pam.d/`. | Rocky Linux 9.8 confirmed. |
| 2 | Agent service verification | Checked the `wazuh-agent` service. | Service not found. |
| 3 | Package investigation | Queried installed Wazuh RPM packages. | Manager, Indexer, and Dashboard installed; Agent package absent. |
| 4 | Manager verification | Checked Wazuh Manager status. | Manager confirmed active. |
| 5 | FIM configuration | Opened `/var/ossec/etc/ossec.conf` and added PAM directory monitoring. | Monitoring entry added. |
| 6 | Service restart | Restarted Wazuh Manager, Indexer, and Dashboard. | Restart commands completed. |
| 7 | Test file creation | Created `/etc/pam.d/lab130-test` with a harmless comment. | Test file created. |
| 8 | Baseline collection | Collected file metadata and SHA-256 hash. | Initial hash recorded. |
| 9 | Controlled modification | Appended a second harmless comment. | File size and SHA-256 changed. |
| 10 | FIM alert detection | Examined the Wazuh Dashboard alert. | Rule 550 detected the modification in real time. |
| 11 | Hash validation | Compared alert hashes with locally collected hashes. | Values matched. |
| 12 | Test file deletion | Removed `/etc/pam.d/lab130-test`. | Deletion command completed. |
| 13 | Deletion verification | Checked whether the test file still existed. | File absence confirmed; deletion alert not included in evidence. |

## Alert Timestamp

The Wazuh Dashboard recorded the modification alert at:

    Nov 7, 2026 @ 09:55:11.893

## Key Evidence Milestones

### Baseline

File:

    /etc/pam.d/lab130-test

SHA-256:

    38fe5cb55494ee7b2d395acc8fb23b6b5caf1709c596508f1078659c8e4a8c64

### Controlled Modification

File:

    /etc/pam.d/lab130-test

SHA-256:

    a53097910575b989a412834d56b93379b6871080c080cc943df31f4a77608eb4

### Wazuh Detection

    Agent ID: 000
    Agent Name: localhost.localdomain
    Rule ID: 550
    Rule Level: 7
    Event: modified
    Mode: realtime
    Path: /etc/pam.d/lab130-test
    Description: Integrity checksum changed.

### Deletion Verification

    ls: cannot access '/etc/pam.d/lab130-test': No such file or directory

## Final Outcome

The controlled modification was successfully detected by Wazuh FIM.

The alert recorded the affected file, changed attributes, and previous and new checksums. The SHA-256 values matched the local evidence.

The test file was subsequently deleted, and its absence was verified. A corresponding deletion alert was not captured in the supplied evidence.

The investigation demonstrated successful detection of a controlled PAM-directory file modification without intentionally changing active authentication configuration.
