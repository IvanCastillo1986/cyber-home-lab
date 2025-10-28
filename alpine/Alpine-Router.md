# Converting Alpine into router
We’ll be converting the Alpine instance into a router with `iptables` and DNS server with `dnsmasq`.<br>
Before proceeding, you’ll need 4 network interfaces on the Alpine VM.<br>
So turn off your instance:<br>
`sudo poweroff`

In the Ubuntu dashboard on the left side, right click on the Alpine instance and choose `Edit`.<br>
Under the `Devices` section, click on:<br>
`+ New`  ->  `Network`
Repeat this process until you have 4 network interfaces.<br>
Go to each interface and select `Bridged (Advanced)` as the **Network Mode**.

Now turn the instance back on.

Add packages. We'll mostly use `iptables` for this guide, and `dnsmasq` will handle DNS caching in a future guide.
```
sudo apk add \
    iptables \
    dnsmasq \
    nano
```

Check that previously installed packages are really installed:
**iptables**
`iptables --version`

**dnsmasq**
`dnsmasq --version`


Enable services to run at boot:<br>
**Enable networking (might already be enabled)**
`sudo rc-update add networking default`

**Enable dnsmasq (we'll configure it later)**
`sudo rc-update add dnsmasq default`

**Enable local (for custom scripts)**
`sudo rc-update add local default`

## Configure Network
Instead of Netplan configuration for Ubuntu, Alpine uses the  `/etc/network/interfaces`  file:<br>
`sudo nano /etc/network/interfaces`

Configure the first interface for WAN as DHCP, and configure the other three for each subnet:
```
# Loopback
auto lo
iface lo inet loopback

# WAN Interface (to internet)
auto eth0
iface eth0 inet dhcp

# Subnet 101 Interface
auto eth1
iface eth1 inet static
    address 192.168.101.1
    netmask 255.255.255.0

# Subnet 102 Interface  
auto eth2
iface eth2 inet static
    address 192.168.102.1
    netmask 255.255.255.0

# Subnet 103 Interface
auto eth3
iface eth3 inet static
    address 192.168.103.1
    netmask 255.255.255.0
```


The kernel system variable configuration files found under `/etc/sysctl.d` that end with `.conf`  are parsed within sysctl(8) at boot time.<br>
Setting kernel variables is as easy as appending text to the `/etc/sysctl.conf` file, or creating a new file under `/etc/sysctl.d/`.<br>
Strings can be appended with the `tee -a` command:<br>
`echo “net.ipv4.ip_forward=1” | sudo tee -a /etc/sysctl.conf`

Let’s finally use the powerful `iptables` tool.<br>
This setup will only be for routing. You can find out how to use iptables as a firewall in another guide.
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth3 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth2 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth3 -m state --state RELATED,ESTABLISHED -j ACCEPT
```
You’ve just set up the service to masquerade the IP address before the packet goes out. You’ve also enabled traffic to leave through each interface, and to return from each interface if the connection has already been established.

Enable the iptables service to start automatically when the system boots:<br>
`sudo rc-update add iptables default`

These iptables rules have now been applied, but they’ll reset on poweroff.<br>
So let’s persist them:<br>
`sudo rc-service iptables save`

Test connectivity between your different subnets.<br>
Congrats! You’ve just built a router for your home lab from scratch.<br>
And because Alpine Linux has such a small footprint, it barely takes up any resources.

[Convert Alpine into a DNS server](Alpine-DNS.md)<br>