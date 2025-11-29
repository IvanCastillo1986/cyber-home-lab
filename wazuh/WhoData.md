# WhoData


## WhoData with auditd
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

Go to your Wazuh Dashboard’s FIM view for your agent, or refresh if it was already open.<br>
You should see a line near the top for the event that was just triggered on your test file.<br>
Click on the magnifying glass icon on the left of that row.

The information will include some new properties that were normalized from the Auditd logs. Be sure to inspect it well to find the differences.


## WhoData with eBPF mode
Extended Berkeley Packet Filter (eBPF) leverages the Linux kernel-space to provide a high-performance alternative to traditional monitoring methods. This mode eliminates the need for external dependencies, such as `auditd`, thus allowing for faster extraction of security events and alerts because of its lower overhead. Who-data monitoring in eBPF mode directly extracts FIM events from programs that use it. It uses resources (like CPU) more efficiently because of its lower overhead. The original BPF was used in traditional tools like `tcpdump`.

Traditional Workflow:<br>
Application  ->  System call to kernel  ->  Kernel   ->  Auditd  ->  Log File  ->  Wazuh Agent

eBPF Workflow:<br>
Application  ->  System call to kernel  ->  Kernel  ->  eBPF program  ->  Wazuh Agent


### Configuration
First, you’ll want to open the `ossec.conf` configuration file with a text editor:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Change to `ebpf` within the <provider> XML tag. If left undefined, it defaults to `audit` mode (the previous `auditd` configuration).<br>
You’ll also want to increase the queue size, since the high speed at which eBPF detects events might result in lost data from within the kernel during mass events.
```
<whodata>
  <provider>ebpf</provider>
  <queue_size>50000</queue_size>
</whodata>
```

Now add a new <directory> tag with the file to monitor, and add the `whodata` property (along with the other properties we’ve covered):<br>
`<directory report_changes=“yes” realtime=“yes” check_all=“yes” whodata=“yes”> /etc/ssh/ssh_config </directory>`

Restart the service with new configuration:<br>
`sudo systemctl restart wazuh-agent`

Test the eBPF configuration by quickly appending from the command line:<br>
`sudo echo “Testing eBPF” >> /etc/ssh/ssh_config`

Open or refresh the FIM page from your Wazuh Dashboard  ->  click on `Events` panel


