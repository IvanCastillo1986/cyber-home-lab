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

