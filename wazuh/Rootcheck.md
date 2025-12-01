# Rootcheck
Rootkits are known for stealthily evading detection by antivirus software. As such, Wazuh provides a robust set of utilities to detect and combat this malware.

Once the Wazuh agent is installed on an endpoint, it uses Rootcheck module to monitor the directories and files, registry entries, and system calls that have been specified for abnormal behavior. The rootcheck module scans the endpoint with predefined rootkit signatures.

That’s not the end of it. Once the Wazuh Manager recieves the logs from the agent, it analyzes them with pre-decoding, decoding, which is how Wazuh processes raw logs into a more meaningful, structured, and readable format. It then matches the log against rules. If the log is a match to the rule, Wazuh Manager creates an alert in `/var/ossec/alerts/alerts.log` and `/var/ossec/logs/alerts/alerts.json` files. As you might have determines from the extensions, one of these files uses the Wazuh log format, and the other uses JSON format.

## Running processes
Malware can disguise itself from showing up on a list of running system processes. It can do this by replacing certain commands with a trojaned version. This might be detected by the Rootcheck module, because it inspects all process IDs for discrepancies.

## Hidden ports
Malware can use hidden ports to communicate from your system with remote attackers. Under the hood, your Linux operating system is written in C. Rootcheck uses the C programming language `bind()` function to scan every port. If it can’t bind to a port, and that port doesn’t show up for `netstat`, it might be malware.

## Unusual files and permissions
Rootheck inspects files with `suid` permissions, hidden directories, and other files to check for unusual permissions. For example, if the `/etc/passwd` file has `rw-rw-rw- root:root` permissions, it means any user could add a new root user. This would completely compromise the system. 

## Rootkit checks
Rootcheck regularly performs checks using 2 files:
* `/var/ossec/etc/rootkit_files.txt`  -  this file defines known filepaths to files commonly used by known rootkits.
* `/var/ossec/etc/rootkit_trojans.txt`  -  this file contains signatures of files that have been trojaned by a rootkit. Most are binaries. Rootcheck converts these binaries to strings, and then checks it against the patterns specified in this file. 
You can also add your own custom signatures to these files, which will also be used during Rootcheck scans.

