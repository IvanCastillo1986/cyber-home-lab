# Agentless Monitoring
Wazuh provides agentless connections to endpoints which don’t support an agent installation. This is common with routers, firewalls, switches, and many other networking devices.


## Connect an Endpoint
Generate a new SSH key:<br>
`sudo -u wazuh ssh-keygen`

Copy the public key to agentless endpoint. Replace <endpoint_username> and <endpoint_ip_address> with the appropriate credentials:<br>
`sudo ssh-copy-id -i /var/ossec/.ssh/id_ed25519.pub <endpoint_username>@<endpoint_ip_address>`

You’ll be prompted to continue connecting. Type in `yes`.

It will ask you for your endpoint’s password. Be sure to type in endpoint’s main user password, and not the server’s password.

Now you should be able to connect to the agentless endpoint via SSH with the following command:<br>
`sudo ssh -i /var/ossec/.ssh/id_ed25519.pub <endpoint_username>@<endpoint_ip_address>`

Once you’ve ensured you’ve established connection, type `exit` and press `Enter` to get back to your Wazuh Server’s shell. 

Now you’re finally ready to add your agentless endpoint to the Wazuh workflow. You’re going to run a script found at `/var/ossec/agentless/register_host.sh`, with the `add` option. Replace the user and host tags with the appropriate credentials. You’ll specify NOPASS because we’re connecting via public key instead of password:<br>
`sudo /var/ossec/agentless/register_host.sh add <endpoint_username>@<endpoint_ip_address> NOPASS`

You should get output similar to below:<br>
`*Host <endpoint_username>@<endpoint_ip_address> added.`

Let’s ensure even mmore that the endpoint is connected.<br>
Run the `register_host` script with the `list` option:<br>
`/var/ossec/agentless/register_host.sh list`


## Removing agentless configuration
It’s currently not possible to remove only agentless endpoint from the configuration, so you must remove all of them, and then add the endpoints back on that you’d like to keep. 

Agentless endpoint credentials are stored in `/var/ossec/agentless/.passlist`. Delete this file:<br>
rm -r /var/ossec/agentless/.passlist


## Configure Agentless Endpoint
The agentless endpoints are theoretically connected to the server, but they aren’t configured for monitoring yet. 

You’ll need the `expect` package, so let’s download it:
```
sudo apt update
sudo apt install -y expect
```


## Testing
For this test, let’s use the `periodic_diff` state. 

Open your Wazuh Server’s main configuration file:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Add the following block of code to your Wazuh Server. In the `<host>` tags, replace `<endpoint_username>@<endpoint_IP>` with the actual username and IP address of your monitored agentless endpoint:<br>
```
<agentless>
  <type>ssh_generic_diff</type>
  <frequency>200</frequency>
  <host> <endpoint_username>@<endpoint_IP> </host>
  <state>periodic_diff</state>
  <arguments>cat /tmp/newfile.txt</arguments>
</agentless>
```

Restart your service to load changes:<br>
`systemctl restart wazuh-manager`

Pull up your agentless enpoint. Add the following file:<br>
`sudo touch /tmp/newfile.txt`

Open it and add some text to it:<br>
`sudo nano /tmp/newfile.txt`

Navigate to your Wazuh Dashboard’s “Threat Hunting” page, and in the search bar, type `agentless.host:*`. You might need to wait a couple of minutes for data to be collected as specified in the `<frequency>` tag.


## Directories of note
`/var/ossec/agentless/`<br>
`/var/ossec/.ssh/`