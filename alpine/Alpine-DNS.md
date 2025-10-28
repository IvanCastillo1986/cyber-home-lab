# Configuring as DNS server
If you haven't done so yet, build your Alpine instance into a DNS server. If you've already done so, ignore this:<br>
[Convert Alpine into a router](Alpine-Router.md)

First install dnsmasq:<br>
`sudo apk add dnsmasq`

Open the dnsmasq configuration file:<br>
`sudo nano /etc/dnsmasq.conf`

The entire file is commented out.<br>
Instead of removing comments and then setting values to the properties, copy/paste the following (substitute the IP addresses and domains to your own):
```
# Basic settings
port=53
listen-address=127.0.0.1,192.168.101.1,192.168.102.1,192.168.103.1
user=dnsmasq
group=dnsmasq
pid-file=/var/run/dnsmasq.pid

# DNS settings
cache-size=1000
no-resolv
server=8.8.8.8
server=1.1.1.1

# Local domain settings
local=/cyberlab.local/
domain=cyberlab.local

# Logging for troubleshooting
log-queries
log-facility=/var/log/dnsmasq.log

# Don't provide services on WAN interface
no-dhcp-interface=eth0

# ---------------------------------------
# Home Lab Entries
# ---------------------------------------

# Network Infrastructure
address=/router.cyberlab.local/192.168.101.1
address=/dns.cyberlab.local/192.168.101.1

# Subnet 101
address=/snort.cyber.local/192.168.101.101
address=/wazuh-agent.cyber/local/192.168.101.102

# Subnet 103
address=/wazuh.cyberlab.local/192.168.103.101
address=/wazuh-manager.cyberlab.local/192.168.103.101  # alias

```

The `no-resolv` option tells dnsmasq to ignore the system’s DNS resolver configuration for upstream DNS servers (since the main DNS server will be the Alpine router/DNS server that we’re currently configuring. By default, dnsmasq checks the system’s `/etc/resolv.conf` file to find upstrream DNS servers. With `no-resolv` declared, we now need to explictly specify upstream DNS servers with the `server` option (we’re declaring 8.8.8.8 and 1.1.1.1).

The `log-queries` option in dnsmasq enables detailed logging of all DNS queries that pass through this DNS server. Without it, there will be no logs to look at for troubleshooting.

The `log-facility` option specifies where log messages should be sent. It controls the destination and handling of dnsmasq’s logging output. `/var/log/dnsmasq.log` will be the file that logs dnsmasq activity. 

The `local` option defines a local domain and controls how DNS queries are handled for that domain. It’s useful for creating internal domain names and controlling DNS behavior for specific domains. It tells dnsmasq “treat this domain as local, and don’t forward queries for it to upstream DNS servers.” <br>
It also shortens the name to be resolved. Instead of `wazuh-agent.cyberlab.local`, you can just type in `wazuh-agent`.

Now restart the service:<br>
`sudo rc-service dnsmasq restart`

Check that the service is running:<br>
`sudo rc-service dnsmasq status`


Now let’s configure the other home lab servers.<br>
On each server, open the following file:<br>
`sudo nano /etc/systemd/resolved.conf`

Scroll down to the line that reads `[Resolve]` and type these two lines underneath it:
```
DNS=192.168.101.1 (replace this IP with your router’s interface for this subnet)
Domains=cyberlab.local
```
Repeat this process for each server in your lab.

Now everything should be ready for testing.<br>
From your wazuh-manager instance, try to ping the wazuh-agent:<br>
`ping wazuh-agent`

Test the rest of the servers.

Congrats! You have just resolved the server’s IP by name.<br>
This will provide easier configuration for Wazuh by domain name, as well as other purposes such as SSH.

Log file at **/var/log/dnsmasq.log**:<br>
`sudo tail /var/log/dnsmasq.log`