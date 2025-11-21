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

## Trigger file monitor #3
This exercise is almost a carbon copy of the `/etc/passwd` watch rule in the previous guide. You might notice some small changes as you make your way through a more modular approach, that scales well.

First, go into the `/etc/audit/rules.d/audit.rules` file, and delete any rules you might have built:<br>
`sudo nano /etc/audit/rules.d/audit.rules`<br>
Delete your added rules, save and exit.

Instead of adding the rule to the `/etc/audit/rules.d/audit.rules` file, create a new file with your editor:<br>
`sudo nano /etc/audit/rules.d/10-file-watches.rules`

Add the following watch rule:<br>
`-w /etc/passwd -p wa -k passwd_change`<br>
Save and exit.

From now on, since you’re building rules with a modular approach, you’ll need to compile any rules you’ve from the `/etc/audit/rules.d` directory with the following command:<br>
`sudo augenrules --load`

Tell `auditd` daemon to reload and activate the new rule. This is still necessary when changing configurations:<br>
`sudo systemctl restart auditd`

Verify that the rule is loaded. It should output your new rule:<br>
`sudo auditctl -l`

Let’s simulate a change by modifying the attributes of the `/etc/passwd` file.<br>
The `touch` command will update the file’s timestamp, which counts as an “attribute change”:<br>
`sudo touch /etc/passwd`

Search the audit logs with the `ausearch` command:<br>
`sudo ausearch -k passwd_change -i`

The returned log will look something like this:
```
----
type=PROCTITLE msg=audit(11/07/2025 04:40:55.276:857) : proctitle=touch /etc/passwd 
type=PATH msg=audit(11/07/2025 04:40:55.276:857) : item=0 name=/etc/passwd inode=527197 dev=fd:02 mode=file,644 ouid=root ogid=root rdev=00:00 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0 
type=CWD msg=audit(11/07/2025 04:40:55.276:857) : cwd=/home/wazuh_manager 
type=SYSCALL msg=audit(11/07/2025 04:40:55.276:857) : arch=aarch64 syscall=openat success=yes exit=3 a0=AT_FDCWD a1=0xffffdd0846c5 a2=O_WRONLY|O_CREAT|O_NOCTTY|O_NONBLOCK a3=0x1b6 items=1 ppid=7238 pid=7239 auid=wazuh_manager uid=root gid=root euid=root suid=root fsuid=root egid=root sgid=root fsgid=root tty=pts1 ses=3 comm=touch exe=/usr/bin/touch subj=unconfined key=passwd_change
```

The output will contain lots of information. Amongst the most notable takeaways is that:
* at 11/07/25 04:40:55, a user successfully used the `touch` command on the file `/etc/passwd`. The `touch` command has two uses:  create a file if it doesn’t exist, and create a timestamp of current time. It does not change the file’s content, and only modifies the last updated timestamp.
* it was done by the user `wazuh_manager`.
* they used `root` privileges
* they did it from the `/home/wazuh_manager` home directory

