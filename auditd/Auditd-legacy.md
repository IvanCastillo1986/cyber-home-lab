# Using Auditd

## Trigger file monitor #1
Start by monitoring a sensitive file with the `-w` option, which stands for “watch”.<br>
Edit the persistent rules file:<br>
`sudo nano /etc/audit/rules.d/audit.rules`

Add the following line to the top of the file. Rules are read sequentially:<br>
`-w /etc/passwd -p wa -k passwd_change`

### What’s happening in this command?
* -w /etc/passwd:  we are telling auditd to watch (-w) the ‘/etc/passwd’ file
* -p wa:  this flag tell auditd to specifically watch permissions (-p) for write (w) and attribute change (a). Other permissions options are read (r) and execute (x).
* -k passwd_change:  this is the key/tag that we’re assigning for this rule. It helps to keep our rules manageable and readable. We’ll be able to call it directly with the name we assign. In this case, we’ve called it `passwd_change`. 

Tell `auditd` daemon to reload and activate the new rule. This is necessary when changing configurations:<br>
`sudo systemctl restart auditd`

Verify that the rule is loaded. It should output your new rule:<br>
`sudo auditctl -l`

Let’s simulate a change by modifying the attributes of the `/etc/passwd` file.<br>
The `touch` command will update the file’s timestamp, which counts as an “attribute change”:<br>
`sudo touch /etc/passwd`

The `ausearch` command is a tool to query the audit logs.<br>
We can refer to a key name we’ve assigned with the `-k` option using the very same flag.<br>
The `-i` flag stands for “interpret”, which converts numeric values (like user IDs) to human-readable names and timestamps. Otherwise it might show up in the returned log as hexadecimal computer-readable format.

Search the audit logs with the `ausearch` command:<br>
`sudo ausearch -k passwd_change -i`

The returned log will look something like this:<br>
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


## Trigger file monitor #2
Edit the rules file again:<br>
`sudo nano /etc/audit/rules.d/audit.rules`

Add the following command to monitor your home file. Change <your_username> to your actual username. Call it `home_dir_write`:<br>
`-w /home/<your_username> -p w -k home_dir_write`

Reload the daemon after modifying your configuration:<br>
`sudo systemctl restart auditd`

Create a new file in your home directory:<br>
`touch ~/test_audit_file.txt`

Search for the new event:<br>
`sudo ausearch -k home_dir_write -i`

Because many processes stem from the user’s home directory, monitoring this might be noisy. So feel free to browse the excessive logs when queried, and acquaint yourself with other types of event logs. Try to find the logs for your actions amongst these. Doing so might spark the motivation to learn how to fine-tune the configurations to minimize the amount of logs down to just the relevant ones.

