# Investigation Timeline 
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

