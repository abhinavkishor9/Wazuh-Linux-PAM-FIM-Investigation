# Troubleshooting Notes

## 1. Wazuh Agent Service Not Found

### Issue

The following command was executed:

    sudo systemctl status wazuh-agent --no-pager

Output:

    Unit wazuh-agent.service could not be found.

### Investigation

The installed Wazuh packages were checked:

    rpm -qa | grep -i wazuh

Output:

    wazuh-indexer-4.13.1-1.x86_64
    wazuh-manager-4.13.1-1.x86_64
    wazuh-dashboard-4.13.1-1.x86_64

### Cause

The Wazuh Agent package was not installed on the Rocky Linux VM.

The machine had the Wazuh server components installed instead.

### Lab Approach

The Wazuh Manager service was checked:

    sudo systemctl status wazuh-manager --no-pager

The service was active.

The investigation proceeded using the manager-side local FIM monitoring context, as evidenced by the alert with `agent.id: 000`.

**Lesson:** A Wazuh Manager and a Wazuh Agent are different components. The absence of an agent service does not mean that the Wazuh Manager is unavailable.

## 2. Wazuh Agent Reported as Inactive

### Issue

The following command was executed:

    sudo systemctl is-active wazuh-agent

Output:

    inactive

### Investigation

The package inventory had already shown that the Wazuh Agent package was absent.

The inactive result should not be interpreted as a stopped, installed agent. The agent service was unavailable because the package was not installed.

### Lesson

Check installed packages and service availability before attempting to start or restart a Wazuh component.

## 3. Confirming Wazuh Manager Status

### Check

    sudo systemctl is-active wazuh-manager

Observed output:

    active

The detailed service status also showed:

    Active: active (running)

### Assessment

The Wazuh Manager was running successfully. Its active status provided the basis for continuing with the manager-side FIM investigation.

## 4. FIM Alert Validation

### Observation

After the controlled file modification, Wazuh generated a Rule ID 550 alert.

Relevant fields:

    rule.description: Integrity checksum changed.
    rule.level: 7
    syscheck.event: modified
    syscheck.mode: realtime
    syscheck.path: /etc/pam.d/lab130-test

### Validation

The SHA-256 values recorded in the alert were compared with the hashes collected locally.

Before modification:

    38fe5cb55494ee7b2d395acc8fb23b6b5caf1709c596508f1078659c8e4a8c64

After modification:

    a53097910575b989a412834d56b93379b6871080c080cc943df31f4a77608eb4

Both values matched the corresponding values in the Wazuh alert.

### Assessment

The alert accurately reflected the controlled file modification.

## 5. Deletion Alert Not Included in Evidence

### Observation

The test file was deleted using:

    sudo rm /etc/pam.d/lab130-test

The following existence check was performed:

    ls -l /etc/pam.d/lab130-test

Output:

    ls: cannot access '/etc/pam.d/lab130-test': No such file or directory

### Limitation

The collected evidence did not include a Wazuh deletion alert.

The file's absence confirms deletion, but it does not prove that the deletion event was received, indexed, or displayed by Wazuh.

### Lesson

Keep filesystem verification separate from alert verification. A successful deletion command and a corresponding FIM deletion alert are different pieces of evidence.

