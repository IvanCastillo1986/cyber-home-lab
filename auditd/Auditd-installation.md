# Auditd

* Logs system calls and file accesses at the kernel level.
* Tracks security-relevant events like privilege escalations, file modifications, and network access.
* Provides immutable audit trails that are hard for attackers to tamper with.
* Complements tools like Wazuh by providing deeper system visibility, as well as centralized logging of these critical logs.

## Kernel level monitoring
Sees everything that happens on the system.<br>
Can’t be bypassed by user-space applications.<br>
Logs events even if processes try to hide their activities.

## Detailed Process Tracking
Who ran what command?<br>
When did it happen?<br>
What files were accessed?<br>
What was the result? (success/failure)

## Forensic Capabilities
Creates detailed timelines of system activity.<br>
Essential for incident response and investigation.

## What auditd catches that other logs miss
- File access (who read an important log, like /etc/shadow)
- Privilege Escalation attempts
- System Call violations
- Raw Command execution
- Network Socket creation

## 3 core concepts
1. System Calls (syscalls):  these are the fundamental interface between a user-space application and the Linux kernel. When a program wants to open a file, create a process, or connect to a network, it makes a system call. We often audit these, since they provide control over the system. 

2. Filters:  rules are built around filters for specific events.
    * -w:  watch a file or directory for any changes (read, write, execute, attribute change).
    * -a:  append a rule to a specific list, often used with `-S` for syscalls.

3. Keys:  (-k flag) is a crucial field for making sense of your logs. It’s a custom, free-text tag you add to a rule. It’s like a way to name your rule.


# Install Auditd
First, update your Ubuntu system:<br>
`sudo apt update && sudo apt upgrade -y`

Install `auditd` and its the `audispd af_unix` plugin:<br>
`sudo apt install auditd`<br>
`sudo apt install audispd-plugins`

## audispd-plugins
`audispd-plugins` (Audit Dispatch Plugins) is a framework that allows Auditd events to be:
* processed in real-time
* sent to external systems
* filtered and transformed before logging
* integrated with other security tools (like Wazuh!)

It adds a few different plugins to our system, but the plugin of note is `af_unix`. This plugin sends audit events to a Unix domain socket in real-time. This makes it perfect for SIEM integrations (like our Wazuh)!

Restart the Auditd daemon:<br>
`sudo systemctl restart auditd`

Check for rules (should be empty by default):<br>
`sudo auditctl -l`