# Alpine
When the machine first spins up, it'll ask for a login. This initial login will be `root`. Initially, root has no password.<br>
You'll then be greeted by the welcome text with a cursor prompt.<br>
The Alpine installation has a special script that you can access by typing:<br>
`setup-alpine`

Choose keyboard layout:<br>
`us`

Select variant:<br>
`us`

Enter system hostname:<br>
`alpine-gateway`

Available interfaces are: eth0<br>
Which one do you want to initialize:<br>
press `Enter` to choose default `eth0`

IP address for eth0?<br>
press `Enter` for DHCP automatic assignment. We'll change this later.

Any manual network configuration: `n`<br>

New password:<br>
Retype password:

Which time zone:<br>
`EST`

Which NTP client to run:<br>
press `Enter` for busybox

Choose `f` to add the fastest mirror.<br>
You can add your user now, or do it later.<br>

Choose the disk that’s shown to format (might be `vda`)<br>
How would you like to use it?<br>
For most users, the best simple option would be `sys`

After the installation process has finished, you’ll be prompted to reboot system. But first you’ll need to remove the installation media. So use the `poweroff` command to shut down Alpine.

Clear the .iso file from the CD drive by going to Settings -> Storage -> click on CD icon to the right -> `remove disk from virtual drive`<br>
Turn the system back on.<br>
Sign in as `root` login.<br>
Run `apk update`. Alpine uses apk package manager, instead of apt.<br>
Add nano with `apk add nano`.

Now, turn on the community mirror, by uncommenting the line in your repositories file:<br>
`nano /etc/apk/repositories`

Remove the `#` in front of the community mirror to uncomment it. The community mirror is needed for access to important packages.

Now add sudo:<br>
`apk update`<br>
`apk add sudo`

Now that you have sudo, let’s change some important user configurations.<br>
`wheel`  is a traditional Unix group for system administrators. <br>
It acts as a gatekeeper.<br>
Let’s give your user admin privileges which can be easily updated by adding or removing it from the `wheel` group.

Create a user if you didn't do it during setup:
`adduser <username>`

Add your user to the wheel group:<br>
`addgroup <username> wheel`

Configure sudo file with `visudo`. This command will prevent errors in syntax and race conditions, which render sudo unusable:<br>
`visudo`

Scroll all the way to the bottom of the file, and find this line:
```
# %wheel ALL=(ALL:ALL) ALL
```

Use vi to remove the # and uncomment that line.<br>
Save the sudoers file and quit vi.

Now, your user should be able to run root privileges with the `sudo` command.<br>
Switch to your user:<br>
`su alpine_gladiator`

Test it:<br>
`sudo apk update`

Now you can install all of your desired packages with the `add` command:<br>
`sudo apk add nano bash bash-completion`

This gives you a better shell and text editor.<br>
Enjoy!

Here are some useful tools to download for your Alpine server:
```
sudo apk add \
    iptables \
    iproute2 \
    dnsmasq \
    wireguard-tools \
    curl \
    nano \
    htop \
    wget
```

Check that packages are really installed:<br>
**iptables**
`iptables --version`

**iproute2 (ip command)**
`ip -V`

**dnsmasq**
`dnsmasq --version`

**Wireguard**
`wg --version`

[Convert Alpine into a router](Alpine-Router.md)<br>
[Convert Alpine into a DNS server](Alpine-DNS.md)<br>