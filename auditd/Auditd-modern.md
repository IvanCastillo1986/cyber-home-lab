# Modern Auditd
Auditd has seen many changes in it’s evolution. In this guide, we’ll be using the most modern workflow.

The newer system for using Auditd yields many benefits including:
* better performance
* more specific filtering
* system call auditing
* future compatibility

## Main Components
The main components of this new system not found in older system:
* /etc/audit/rules.d/:  where the rules are stored
* augenrules:  the compiler that processes rules
* auditd:  the background service that does the monitoring
* /var/log/audit/audit.log:  the stored logs
* ausearch:  a tool used to query logs
* aureport:  generates human-readable summary reports from current log file (before rotation)

## The workflow
Your rule files (.rules)  ->  augenrules (compiler)  ->  audit.rules (final config)  ->  
auditd (daemon)  ->  /var/log/audit/audit.log (published log file)


## 3 types of rules
Before moving forward, let’s go over the types of rules.

### File watch rules
Monitor specific files or directories for changes. If the watch rule path is a directory, then the rule is recursive to the bottom of its directory tree.<br>
This is the general structure:<br>
`-w /path/to/file -p permissions -k key_name`

Examples:<br>
`-w /etc/passwd -p wa -k identity_files`<br>
`-w /usr/bin/ -p x -k binary_execution`<br>
`-w /etc/ssh/sshd_config -p wa -k ssh_config`<br>


### Syscall rules
Monitor specific system calls.<br>
System calls are the interface between user applications and the Linux kernel. When a program needs to do anything privileged, it makes a system call.<br>
`-a action,list -S syscall -F field=<value> -k <key_name>`

Examples:<br>
`-a always,exit -S execve -k process_execution`<br>
`-a always,exit -S chmod -S fchmod -k permission_changes`<br>
`-a always,exit -S mount -S unmount -k filesystem_mounts`


### Field comparison rule
Add conditions to Syscall rules using filters.

Examples:<br>
`-a always,exit -S open -F uid=0 -k root_file_access`<br>
`-a always,exit -S open -F success=0 -k failed_file_access`<br>
`-a always,exit -F arch=b64 -S open -k file_access_64bit`<br>
`-a always,exit -S all -F pid=1234 -k specific_process`

