# Wazuh Logs
Logcollector module collects logs from monitored endpoints, applications, and network devices.
The Wazuh Server analyzes collected logs in real-time using decoders and rules, by extracting information and mapping them to appropriate fields.
Analysisd module evaluates decoded logs against rules, and records alerts in `/var/ossec/logs/alerts/alerts.log` and `/var/ossec/logs/alerts/alert.json` files.

On top of alert logs, Wazuh also stores collected logs in archive files `/var/ossec/logs/archives.log` and `/var/ossec/logs/archives/archives.json`. These archives even include logs that don’t trigger alerts, providing an even bigger picture of system activity.

Wazuh provides larger coverage to include endpoints that don’t support installation of Wazuh agents by receiving syslog logs.


## Syslog from unsupported endpoints
Some devices that wouldn’t have Wazuh support might be networking hardware such as firewalls, switches, and routers.

Add the following block inside of the `<ossec_config>` blocks within Wazuh Server’s `ossec.conf` configuration file:<br>
```
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>tcp</protocol>
  <allowed-ips>192.168.2.15/24</allowed-ips>
  <local_ip>192.168.2.10</local_ip>
</remote>
```

Let’s break down what each tag is doing:<br>
* `<connection>`:  the type of connection to accept. This property accepts `syslog` or `secure` as values.
* `<port>`:  port used to listen for incoming syslog messages from endpoints. 
* `<protocol>`:  protocol used to listen for incoming syslog messages from endpoints. This property accepts `tcp` or `udp` as values.
* `<allowed_ips>`:  the IP address or network range of endpoints.
* `<local_ip>`:  the IP adddress of the Wazuh Server listening for incoming messages.

Save and exit the `ossec.conf` file.<br>
Restart the service to load changes:<br>
`sudo systemctl restart wazuh-manager`


## Send syslogs from Wazuh agent
From the Wazuh Agent, open the Rsyslog configuration file:<br>
`sudo nano /etc/rsyslog.conf`

Scroll to the bottom of the file, and add the following line. Replace <wazuh_manager_ip> with your server’s actual IP address:<br>
`*.info@@<wazuh_manager_ip>:514`

Restart the agent’s service:<br>
`sudo systemctl restart wazuh-agent`

Add the following block inside of the `<ossec_config>` blocks within Wazuh Server’s `ossec.conf` configuration file:<br>
```
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>tcp</protocol>
  <allowed-ips>192.168.2.15/24</allowed-ips>
  <local_ip>192.168.2.10</local_ip>
</remote>
```

Save and restart manager’s service to load changes:<br>
`sudo systemctl restart wazuh-manager`

## Journald
Journald is the new generation version of Rsyslog, for Linux systems. It completely replaces syslog in current minimized versions of Ubuntu, and perhaps other flavors of Linux. It provides centralized log management, and luckily Wazuh’s agentd service can integrate these logs into our SIEM.

## Journald configuration
The configuration is easy as adding this  block of code to your Wazuh agent’s `/var/ossec/etc/ossec.conf` file:
```
<localfile>
  <location>journald</location>
  <log_format>journald</log_format>
</localfile>
```

The Wazuh agent now has a channel with which to send Journald logs.


### Sending specific Journald logs
On your Wazuh agent, open your `ossec.conf` file:<br>
`sudo nano /var/ossec/etc/ossec.conf` 

Scroll down to the bottom of the file, and there should be another small `<ossec_config>` block of code.
Let’s take advantage of this separation and add the following block to that block:
```
<!-- For monitoring journald service -->
<localfile>
  <location>journald</location>
  <log_format>journald</log_format>
  <filter field="_SYSTEMD_UNIT">^ssh.service$</filter>
</localfile>

<localfile>
  <location>journald</location>
  <log_format>journald</log_format>
  <filter field="_SYSTEMD_UNIT">^cron.service$</filter>
  <filter field="PRIORITY">[0-6]</filter>
</localfile>
```

The `PRIORITY` field defines the range of priority levels that should be forwarded to the Wazuh Manager. Events that fall out of this range will be ignored.

Restart the service:<br>
`sudo systemctl restart wazuh-agent`


From your Wazuh Server, open your local decoder file:<br>
`sudo nano /var/ossec/etc/decoders/local_decoder.xml`

Add the following block of decoders:
```
<decoder name="cron-service">
    <program_name>cron|CRON</program_name>
</decoder>

<decoder name="cron-service1">
    <parent>cron-service</parent>
    <regex type="pcre2">\((\w+)\) CMD \((.*?)\)$</regex>
    <order>service_user, command</order>
</decoder>

<decoder name="cron-service1">
    <parent>cron-service</parent>
    <regex type="pcre2">\((\w+)\) INFO \(Skipping @(\w+) jobs .*?\)$</regex>
    <order>service, message</order>
</decoder>
```

Save and exit.<br>
Next, open your local rules file:<br>
`sudo nano /var/ossec/etc/rules/local_rules.xml`

Add the following block of code:
```
<!-- Rules for journald logs for CRON services-->
<group name="cron-service,">
  <!-- rule for groupped logs -->
  <rule id="111800" level="0">
    <decoded_as>cron-service</decoded_as>
    <description>Journald logs for CRON.</description>
  </rule>

  <!-- rules to detect when the cron.service executes a command-->
  <rule id="111801" level="8">
    <if_sid>111800</if_sid>
    <match>CMD</match>
    <description>CRON: The $(service_user) user executed ($(command)) on $(hostname).</description>
  </rule>

  <!-- rules to detect when the cron.service performs a reboot-->
  <rule id="111802" level="12">
    <if_sid>111800</if_sid>
    <match>not system startup</match>
    <description>CRON: The cron.service on $(hostname) just performed a $(message).</description>
  </rule>

</group>
```

Restart the service:<br>
`systemctl restart wazuh-manager`


### Test the configuration
From your Wazuh Manager, use the following code to establish an SSH connection to the Wazuh Agent. Replace <user_name> with the name of the user on the agent, and replace <agent_ip> with the actual IP address of the agent:<br>
`ssh <user_name>@<agent_ip>`

Pull up your agent, and accept the connection request. You won’t need a password for this, so just press `Enter` when prompted. You are now connected to the agent!

Use this remote opportunity to create a Python script in root’s directory:<br>
`sudo nano /root/hello.py`

Add the following code to the Python script:
```
#!/usr/bin/env python3

def main():
    print("Hello, World!")

if __name__ == "__main__":
    main()
```

Save and exit.<br>
Now let’s add a new cronjob to run this script. Open crontab with the following:<br>
`sudo crontab -e`

Add the following line of code to the bottom of the page, to run the script every minute:<br>
`* * * * * /hello.py`

Navigate to your Wazuh Dahsboard’s `Threat Hunting` page to see these notifications in action!

Before disconnecting from the remote SSH, you’ll want to stop the cronjob from continuing to run every minute. If you disconnect before doing it, it might be difficult to stop the cronjob on your agent later from another user’s account. So use the same command to open up the CRON file:<br>
`sudo crontab -e`

And delete the cronjob you added previously.<br>
The alerts should now have stopped from flooding your Wazuh Dashboard.

