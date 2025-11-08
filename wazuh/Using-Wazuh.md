# Wazuh Functions


## Vulnerabilities
Wazuh can track down vulnerabilities. 

Using your Wazuh Manager’s terminal, open the global configuration file, and scroll down about 117 lines:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Toggle on the vulnerability detector:
```
<vulnerability_detector>
	<enabled> no </enabled>
</vulnerability_detector>
```
-> &nbsp; and change to &nbsp; ->
```
<vulnerability_detector>
	<enabled> yes </enabled>
</vulnerability_detector>
```

<hr>

## File Integrity Monitoring (FIM)
is a security process which detects changes in the integrity of files by monitoring, scanning, and verifying in real-time. It does this by scanning a baseline state, and creating a cryptographic checksum hash out of it. Unfortunately, this means that if a file or system has already been compromised when FIM is implemented, it probably won’t detect the anomalies because it will read the compromised system as the baseline state.

Wazuh FIM event data is collected in two databases that are stored within the following directory:<br>
`/var/ossec/queue/fim/db`

One of those databases is created and managed by `wazuh-db` daemon.<br>

FIM is enabled and working by default. If you were to run the `cat` command on one of the `/var/ossec/queue/fim/db` database files, you would find a bunch of directories/file locations and their hashes (which might not be human-readable).<br>

However, you can still change the FIM module’s configuration and tailor it to your particular needs. 

The local configuration file can be found at:<br>
`/var/ossec/etc/ossec.conf`

The centralized configuration is a bit more nuanced, as Wazuh uses a “shared configuration” model (“shared” is literally the directory name). It can be found at:<br>
`/var/ossec/etc/shared`

* `/default/agent.conf`:  this configuration is applied to every single agent by default. Coincidentally, all agents belong to a group called `default`, by default. Any configuration here will be applied to all agents that aren’t in a more specific group.
* `/group/`:  this directory contains sub-directories for custom agent groups. These configurations take precedence over the rules in `agent.conf`.

FIM-monitored directories and files must be declared in the configuration files. 

Tip:  I suggest only using the global for now. The groups seem like a specific use case to me.

Adding entry to FIM
I’ll use a `/hello/world` structure for demonstration purposes, but feel free to use any directory you’d like to monitor or test with.<br>
Create the file (or skip this step if you’re adding a legitimate directory).<br>
`sudo mkdir /hello`

Edit the global configuration file:
`sudo nano /var/ossec/etc/ossec.conf`

Scroll about halfway down the file, til you find the `<syscheck>` tag under the `File Integrity Monitoring` comment.<br>
Add a new entry under the `<directories>` tags.<br>
Be sure to give it the `realtime` property set to “yes”, so that the changes update quickly:<br>
`<directories realtime=“yes”>/hello</directories>`

Write out and exit the text editor.<br>
Restart the service:<br>
`sudo systemctl restart wazuh-agent`

Open the file and write any changes:<br>
`sudo nano /hello/world.txt`

Write out and close.<br>
Navigate to the Wazuh Manager’s **File Integrity Monitor**  ->  **Events** panel.<br>
You might need to hit the refresh button on the top right of the Events dashboard window. 

Congrats! Modifications to any files nested within the chosen directory will trigger an event entry. <br>
(Tip:  adding or deleting any nested directories will not trigger changes, unless you modify a file within it.)


### Report changes
Adding the `check_all` property includes every available attribute in the log, which will be included in alerts. This will allow more checks of integrity, such as md5, sha1, and sha256 hashes, before and after.

Open the `/var/ossec/etc/ossec.conf` config file with text editor.
Add the `report_changes` property to the `<directories>` tag to include a log in the alert:
```
<syscheck>
   <directories  check_all="yes"  report_changes="yes" realtime=“yes”><FILEPATH_OF_MONITORED_FILE></directories>
</syscheck>
```

Log files for `report_changes`:<br>
`/var/ossec/queue/diff/local/`


### Changing scan schedule
There are two forms of scheduling.<br>
One is set to execute at the same repeating interval, and the others are set at your desired times.

Open the global config:<br>
`/var/ossec/etc/ossec.conf`

Scroll down to about the 107th line within the `<syscheck>` tag. Change the default interval number to your desired wait between:<br>
`<frequency>900</frequency>`

