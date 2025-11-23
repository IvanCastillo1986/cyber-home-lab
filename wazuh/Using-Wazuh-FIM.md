# Wazuh FIM


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

### Adding entry to FIM
I’ll use a `/hello/world` structure for demonstration purposes, but feel free to use any directory you’d like to monitor or test with.<br>
Create the file (or skip this step if you’re adding a legitimate directory).<br>
`sudo mkdir /hello`

Edit the global configuration file:
`sudo nano /var/ossec/etc/ossec.conf`

Scroll about halfway down the file, til you find the `<syscheck>` tag under the `File Integrity Monitoring` comment.<br>
Add a new entry under the `<directories>` tags.<br>
Be sure to give it the `realtime` property set to “yes”, so that the changes update quickly:<br>
`<directories realtime=“yes”>/home/wazuh_user/hello</directories>`

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


## Creating custom FIM rule
Create a custom rules file inside of your Wazuh Manager's `/var/ossec/etc/rules` directory:<br>
`sudo touch /var/ossec/etc/rules/test_FIM`

Add the following rule definition to your newly created file:<br>
```
<group name="syscheck">
  <rule id="100002" level="8">
    <if_sid>550</if_sid>
    <field name="file">.sh$</field>
    <field name="changed_fields">^permission$</field>
    <field name="perm" type="pcre2">\w\wx</field>
    <description>Execute permission added to shell script.</description>
    <mitre>
      <id>T1222.002</id>
    </mitre>
  </rule>
</group>
```

When your `wazuh-manager` daemon starts up, it loads any custom rules files inside of that directory.<br>
Restart the service so your new rule takes effect:<br>
`sudo systemctl restart wazuh-manager`

On your Wazuh Agent, create a new directory to monitor in your home directory. Replace <your_username> with your actual username on the Wazuh Agent's VM instance:<br>
`sudo mkdir /home/<your_username>/FIM-test`

Create a new shell script inside of that directory to trigger alert:<br>
`sudo touch /home/<your_username>/FIM-test/test-script.sh`

Open the Wazuh Agent's configuration file with a text editor:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Scroll halfway down the file, and find the `<syscheck>` tag.<br>
Add the following `<directories>` tag within it to monitor the new directory with the test script:
```
<syscheck>
   <directories realtime="yes"> /home/<your_username>/FIM-test </directories>
</syscheck>
```

Restart the `wazuh-agent` service to load the new configuration:<br>
`sudo systemctl restart wazuh-agent`

By default, execute permissions are off when you create a new file on Linux systems (for good reason). But we'll be adding execute permissions in order to trigger the FIM alert:<br>
`sudo chmod +x /home/<your_username>/FIM-test/test-script.sh`

Go back to your Wazuh Manager instance.<br>
Navigate to the Wazuh Manager’s **File Integrity Monitor**  ->  **Events** panel.<br>
You might need to hit the refresh button on the top right of the Events dashboard window. 

Near the top of the list of events in the bottom panel, you should see an event entry that reads the same `description` that you added to the custom rule you created: `Execute permissions added to script`.


## Troubleshooting FIM
If changes from FIM aren't showing up in the Wazuh Manager's dashboard:<br>
1. Changes might not have taken effect yet. You might need to wait a few seconds for data to show up on the dashboard.
2. Go back to the `/var/ossec/etc/ossec.conf` configuration file and check for any syntax errors.
3. Ensure that you have the `realtime="yes"` property on the `<directories>` tag, which enables real-time monitoring.
4. You might have fogotten to restart the `wazuh-agent` service (or `wazuh-manager` if that's the instance you're testing).
5. You might not have an absolute path in the tag. Wazuh can't automatically find your directory (like `/hello`) unless you've added an absolute path that stems from the root directory (like `/home/wazuh_user/hello`).
6. You are trying to monitor an actual file instead of directory. Wazuh is not currently able to accept a file as the subject for FIM. So you need to choose a directory.

