# ClamAV
ClamAV is another free, open-source technology to add to the Linux security stack. It detects viruses, malware, trojans, and other malicious threats on servers, email gateways, and web applications. It also easily integrates into Wazuh.

## Install ClamAV
It’s as simple as installing clamav and its daemon:<br>
`sudo apt install clamav`<br>
`sudo apt install clamav-daemon`

## Using ClamAV
To manually scan using clamav, use the following command:<br>
`clamscan`  -  simply scans your system for malware. It’s on-demand when you’d like a manual scan, as opposed to the daemon which runs silently in the background.

You’ll need to be specific if you’d like it to scan restricted directories. For example:<br>
`clamscan /root`

There’s also a process running called `freshclam`. No need to trigger it, it works in the background and synchronizes with virus database automatically.

## Configuring
The main configuration files are found at:<br>
`/etc/clamav/clamd.conf`<br>
`/etc/clamav/freshclam.conf`

## Rules
Wazuh already has out-of-the-box decoders for ClamAV, so there’s no need to create your own. The rules can be found at:<br>
`/var/ossec/ruleset/rules/0320-clam_av_rules.xml`

## Testing
ClamAV logs are automatically forwarded from `journald`. 

As a test, we can stop the daemon and it’s child process:<br>
`sudo systemctl stop clamav-daemon`<br>
`sudo systemctl stop clamav-daemon.socket`

Navigate to your Wazuh Manager’s “Threat Hunting” section, and you should see these alerts near the top.
