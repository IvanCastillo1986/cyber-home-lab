# CDB lists and threat intelligence
A CDB list is a text file which contains known malware threat intelligence indicators as key-value pairs, such as (but not limited to):
* users
* file hashes
* IP addresses
* domain names

Wazuh can use CDB lists along with the FIM module. 

**Workflow:**
FIM module scans monitored directories  ->  <br>
FIM module stores checksums and attributes of monitored files  ->  <br>
FIM module generates alert  ->  <br>
Wazuh engine compares attributes (like hash) to keys in predefined CDB list  ->  <br>
Wazuh analysis engine finds match  ->  <br>
engine either generates or suppresses an alert, based on configuration


## Add CDB list and rule
Create a new CDB list on your Wazuh Manager instance. Wazuh will know to check for the list within the `/var/ossec/etc/lists` directory:<br>
`nano /var/ossec/etc/lists/malware-hashes`

We’ll be adding the hashes of two known pieces of malware to the CDB list, in the form of key/value pairs. The key will be their MD5 hash, and the value will be the name of the malware:
```
e0ec2cd43f71c80d42cd7b0f17802c73:mirai
55142f1d393c5ba7405239f232a6c059:Xbash
```

Open your configuration file:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Scroll down near the bottom of the file until you find the `<ruleset>` tag. Add one more `<list>` to the tag with the path of the CDB list:
```
<ruleset>
  <list>/var/ossec/etc/lists/malware-hashes</list>
</ruleset>
```

Create a custom rule in the `/var/ossec/etc/rules/local_rules.xml` file. What you write in this rule is what will show up in the alert when a file in your monitored FIM path matches the MD5 hash from your CDB list:
```
<group name="malware,">
  <rule id="110002" level="13">
    <!-- The if_sid tag references the built-in FIM rules -->
    <if_sid>554, 550</if_sid>
    <list field="md5" lookup="match_key">etc/lists/malware-hashes</list>
    <description>File with known malware hash detected: $(file)</description>
    <mitre>
      <id>T1204.002</id>
    </mitre>
  </rule>
</group>
```

Restart the service to load the changes in configuration:<br>
`sudo systemctl restart wazuh-manager`


## Add monitored file
In your Wazuh Agent, create a new directory to monitor in your home directory:<br>
`mkdir malware_dir`

Add the newly created directory to your monitored paths. Change `<your_username>` to your actual username:
```
<syscheck>
  <directories check_all=“yes” realtime=“yes”>/home/<your_username>/malware_dir</directories>
</syscheck>
```

Reload for configuration changes:<br>
`sudo systemctl restart wazuh-agent`


## Test with malware
Change into the directory you’ve created to store the malware:<br>
`cd ~/malware_dir`

Download mirai and Xbash malware into this directory:<br>
`sudo curl https://raw.githubusercontent.com/wazuh/wazuh-documentation/refs/heads/4.14/resources/samples/mirai --output ~/malware_dir/mirai`<br>
`sudo curl https://raw.githubusercontent.com/wazuh/wazuh-documentation/refs/heads/4.14/resources/samples/xbash --output ~/malware_dir/Xbash`

Go back to your Wazuh Manager’s dashboard. Navigate to the  Threat Hunting  ->  Event tab.<br>
You should see alerts for the two malware files you’ve just downloaded into your monitored directory. Expand by clicking on the magnifying glass to the left of the row.<br>
You might notice details that you coded into the rule in the Wazuh Server’s `ossec.conf` configuration file.
