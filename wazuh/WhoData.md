# WhoData

If you’re anything like me, you find Auditd logs to be complex and not very readable. In steps WhoData, which is a powerful Wazuh built-in feature which uses Auditd to give even further visibility. It integrates with Wazuh’s File Integrity Monitor (FIM) and uses Auditd logs to include the answer to “Who did it?” and “How did they do it?” It also includes which process (which is often a binary in Linux), and the command that was used. It automatically configures Auditd rules, parses and interprets Auditd logs, correlates Auditd events with FIM events, and presents the combined information as alerts for professionals to view on the Wazuh Dashboard. It parses the Auditd logs into clean JSON fields, a much better reading experience!

In order to use WhoData, just add the “whodata” property to your monitored directory within the FIM section of the configuration file.

From the Wazuh Agent, create a new file in your home directory. Replace <username> with your actual username:<br>
`sudo touch /home/<username>/testfile.txt`

Use an editor to open your Wazuh configuration file:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Scroll down to the FIM section, and add a new `<directory>` tag containing your monitored file, and add the `whodata` property:
```
<syscheck>
  <directories whodata=“yes” real_time=“yes” report_changes=“yes” check_all=“yes”> /home/<username>/testfile.txt </directories>
</syscheck>
```

Restart your Wazuh Agent service to load configuration changes:<br>
`sudo systemctl restart wazuh-agent`

Now that you’ve added whodata to a directory, you should be able to read an automatic rule that was added to your Auditd configuration:<br>
`sudo auditctl -l`

It should read:<br>
`auditctl -w /etc -p wa -k wazuh_fim`

This rule was automatically created by the `wazuh-agent` service, and it disappears when the daemon stops.
Now add some text to your newly created file, save and exit.

Go to your Wazuh Dashboard’s FIM view for your agent, or refresh if it was already open.
You should see a line near the top for the event that was just triggered on your test file.
Click on the magnifying glass icon on the left of that row.

The information will include some new properties that were normalized from the Auditd logs. Be sure to inspect it well to find the differences.