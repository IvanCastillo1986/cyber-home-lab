# Command Monitoring
Wazuh’s Command Monitoring allows you to monitor the output of specific commands, and pull up alerts for any changes that are detected. In other words, this module creates its own logs within Wazuh. It can be used to detect a variety of anomalies and threats. 

Command Monitoring can be used to monitor many things, including:
* Disk space utilization
* Load average
* Change in network listeners
* Running processes

Command Monitoring Workflow:
1. The user adds desired command to the local agent configuration file or remotely through the Wazuh server. Achieve this configuration through the Command or Logcollector module.
2. Wazuh agent uses the configured frequency or interval to periodically execute the command.
3. Wazuh agent monitors the command’s execution and forwards its output to Wazuh Server for analysis.
4. Wazuh Server analyzes output by predecoding, decoding, and matching the monitor’s logs against predefined rules to generate security alerts. When a match occurs, an alert is generated and stored in Wazuh Server’s `/var/ossec/logs/alerts/alerts.log` and `/var/ossec/logs/alerts/alerts.json` files. The alert is also simultaneously displayed on the Dashboard.


Wazuh offers two modules for monitoring executed system command output:  Logcollector and Command modules.

## Command module
This is the module officially recommended by Wazuh. It provides:
* Checksum Verification:  verifies the integrity of every executed binary or script by comparing hashes. It ensures that binaries have not been modified or replaced.
* Encrypted Communication:  communication between Wazuh Agent and Wazuh Server is encrypted with AES.
* Scheduling Execution:  this module is flexible and can be configured to run the commands on endpoints as follows:
    * immediately after Wazuh agent service starts
    * specific intervals
    * specific day of month, day of week, or time of day.


## Logcollector module
This module receives logs through text files, Windows event logs, and syslog messages. This makes it suitable for firewalls and other network devices. It also functions as an alternative for running commands and processing command outputs. 

Logcollector modules offers the following the features:
* Formatting Command Output:  format the output of a command by including fields like `timestamp`, `hostname`, `program_name`, and more for better log identification.
* Reading Multiline Command Output:  allows the module to read a command output as one or more log messages, depending on whether you pass in the `command` or `full_command` values.


## Custom Rulesets for Logcollector
Wazuh provides an out-of-the-box decoders at `var/ossec/ruleset/decoders/` that go hand in hand with out-of-the-box rules at `/var/ossec/ruleset/rules/`. However, these rules have the `level` set to `0`, so they won’t generate an alert. In order to genarate an alert from one of these rules, you need to create a custom rule that inherits that base rule with an increased level. Otherwise, you’ll need to create custom decoders on top of custom rules in order for Logcollector to trigger alerts. 


## Decoding and rule-matching
The pre-decoding phase of the analysis workflow extracts data from the raw log header (like timestamp, hostname, program name). Then, the analysis engine looks for a decoder to match the log. If a decoder is found, the engine uses the decoder to extract even more fields and matches the appropriate rules to generate security alerts. If no matching decoder is found, the log is ignored (Wazuh assumes it’s unimportant).


## Monitoring running processes
This is an exercise in which netcat is used, and is detected by Wazuh with the Command Monitoring module. 

First, install `ncat` and `nmap` (ncat is an extension of the nmap suite of tools):
```
sudo apt update
sudo apt install ncat nmap -y
```

Add the following configuration block to the Wazuh Agent’s `/var/ossec/etc/ossec.conf` configuration file:
```
<ossec_config>
  <localfile>
    <log_format>full_command</log_format>
    <alias>process list</alias>
    <command>ps -e -o pid,uname,command</command>
    <frequency>30</frequency>
  </localfile>
</ossec_config>
```

Restart the agent service:<br>
`sudo systemctl restart wazuh-agent`

Now pull up your Wazuh Server.<br>
Add the following rules block to your `/var/ossec/etc/rules/local_rules.xml` file:
```
<group name="ossec,">
  <rule id="100050" level="0">
    <if_sid>530</if_sid>
    <match>^ossec: output: 'process list'</match>
    <description>List of running processes.</description>
    <group>process_monitor,</group>
  </rule>

  <rule id="100051" level="7" ignore="900">
    <if_sid>100050</if_sid>
    <match>nc -l</match>
    <description>netcat listening for incoming connections.</description>
    <group>process_monitor,</group>
  </rule>
</group>
```

You might’ve noticed that there are two rules there. The top rule is the “parent” rule, and the bottom rule is the “child” rule. The parent rule is like a filter before passing it down to a more specific rule. And if you look closely at the parent’s `<if_sid>` tag, you’ll see that this “parent” rule belongs to a “grandparent” rule with the id of 530. 

You won’t be using it for this exercise, but this is what the grandparent rule looks like:
```
<rule id="530" level="0">
  <match>^ossec: output:</match>
  <description>List of running processes.</description>
  <group>process_monitor,</group>
</rule>
```

The rule chain looks like this:
* The raw log looks something like this:  `ossec: output: 'process list': root 1234 nc -l -p 4444`. It’s caught by the grandparent rule as it gets filtered through all other rules.
* Grandparent rule with id 530 catches all process list outputs from Wazuh’s Command Monitoring module. 
* Parent rule with id 100050 filters from all process outputs, down to just process list outputs. This rule can have many children. For our case, child catches netcat.
* Child rule with id 100051 is finally what triggers the alert. It alerts when netcat listeners are found in process lists.

Back to the exercise.<br>
Restart the service for new rules in configuration to take effect:<br>
`sudo systemctl restart wazuh-manager`

Now you’re ready to trigger the alert.<br>
From your Wazuh Agent’s shell, run:<br>
`nc -l 8000`

You can view the alerts from the “Threat Hunting” page in your Wazuh Dashboard.
