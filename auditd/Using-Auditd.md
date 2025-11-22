# Using Auditd

## Rotating auditd logs
Auditd comes with its own built-in log rotation mechanism. This is automatically done because the log files can fill up pretty quickly, especially if there are many rules, and not enough fine-tuning.

You can find it directly in its `/etc/audit/auditd.conf` configuration file. So let’s open it up with a text editor:<br>
`sudo nano /etc/audit/auditd.conf`

Luckily this configuration file isnt too big. Near the top of the page, you’ll find the following property:<br>
`max_log_file_action = ROTATE`

As you can see, it automatically rotates its own files when it gets too big. A few lines down from the top, you can find the following property:<br>
`max_log_file = 8`<br>
The digit is the amount of MB before logs are rotated. The default is 8MB.
Note that while other log rotation schemes depend on time reached, this one depends on storage space occupied before rotating.

Right below that line is another property:<br>
`num_logs = 5`<br>
This is the amount of times that logs will be collected before they’re deleted. 

So when the first log file reaches 8MB, it will be automatically rotated (saved with auditd’s automatic rotation naming scheme), and a new log file will be created to continue storing logs. This pattern will continue until eventually our first log file (now named auditd.log.5 or whatever auditd names it), will finally be deleted. This is because our `num_logs` property is set to 5 by default, but you can choose to assign it whatever number of rotated logs you’d like to keep. 


## Deleting rule
If a rule (such as monitoring home directory) is too noisy and cluttering your logs, you can always delete it. 

List all of your rules:<br>
`sudo auditctl -l`

Find the exact rule you’d like to delete.<br>
Then copy this the exact same rule, but with an uppercase `-W` flag:<br>
`sudo auditctl -W /home/wazuh_manager -p w -k home_dir_write`

List your files again:<br>
`sudo auditcl -l`

Your file should no longer be returned in output.


## Unecessary Noise
Monitoring broad files and syscalls can create an overwhelming amount of log events to sift through.
You can include an option in the command that tells ausearch to only include events from today:
`sudo ausearch -k file_deletion -i --start today`

Another method to manage your search through log events is to pipe it into a more human-readable experience:
`sudo ausearch -k file_deletion -i | less`
`sudo ausearch -k file_deletion -i | tail -n 30`
