# Configure Kali
There are a few initial 'tweaks' you'll want to configure in your new Kali instance.

## Kali-tweaks
Open your Kali terminal, and type the following:<br>
`kali-tweaks`

Kali Tweaks  -  allows you to easily configure Kali’s: 
* security hardening
* metapackages
* network repositories
* shell & prompt
* virtualization

Kali comes with Zsh as its default shell, but if you’re anything like me, you’d rather work with Bash. And we can easily change that with this interface.<br>
Use the arrow keys to select `Shell & Prompt`, then press `Enter`.<br>
Select `Default Login Shell` and press `Enter`.<br>
Select `Bash`, and then press `Spacebar` to select it.<br>
Select `Apply` and press `Enter`.

You’ll need to logout, and log back in for these changes to take effect.<br>
Find the Logout icon in the upper right-hand corner that looks like a circle with an arrow inside pointing right.<br>
Select `Log Out`.<br>
Enter your credentials to sign back in.<br>
The changes should have taken effect.


## Configure static IP address
Use the following command to check your dynamically assigned address:<br>
`ip a`

`eth0` should be set to a seemingly random address, which was automatically assigned via DHCP.<br>
Let’s statically assign it manually. For this we can use NetworkManager’s command-line interface tool.

Start off by checking your interface’s name recognized by NetworkManager for our purposes:<br>
`nmcli connection show`

You should see some rows of data. One of them will read `eth0` under the `DEVICE` column. This is the interface that you’re bridging to your main network. On that same row, look at what’s displayed in under the `NAME` column. It should read something along the lines of “Wired connection 1”. This is the name you’ll be using to configure your interface.

Use the following commands to modify your connection:
```
sudo nmcli connection modify "Wired connection 1" \
    ipv4.addresses <your_kali_ip_address>/<CIDR_number> \
    ipv4.gateway <your_gateway_router_ip> \
    ipv4.dns "8.8.8.8,1.1.1.1" \
    ipv4.method manual
```

The following is an example of the previous commands with real values plugged in:
```
sudo nmcli connection modify "Wired connection 1" \
    ipv4.addresses 10.0.4.101/24 \
    ipv4.gateway 10.0.4.1 \
    ipv4.dns "8.8.8.8,1.1.1.1" \
    ipv4.method manual
```

NetworkManager’s `nmcli` tool allows us to easily configure interfaces right from the command line!

You’ll need to restart your connection before changes take effect:<br>
`sudo nmcli connection down “Wired connection 1”`<br>
`sudo nmcli connection up “Wired connection 1”`

Use the following command to ensure that the changes have taken effect:<br>
`ip route show`

Test the configuration by pinging your gateway router, pinging google, and then pinging a machine in another subnet:<br>
`ping 10.0.4.1`<br>
`ping 8.8.8.8`<br>
`ping 10.0.1.101`