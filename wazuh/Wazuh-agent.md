# Wazuh Agent
The Wazuh Agent is securely enrolled to the Wazuh Manager through its enrollment service.<br>
It ensures that the communication is configured and authenticated properly.


First, let’s change into privileged shell so we don’t have to worry about permissions errors:<br>
`sudo -s`


## Add Wazuh repository
Install the GPG key. This chain of commands:
* Downloads the GPG key from the repository. 
* It then pipes that output to the gpg command. Its options tell it to not put the keyring in the default location, and instead to create a keyring to store the downloaded certificate in. 
* It then changes appropriate permissions.

`curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg`

Add the repository. Now that the key has been created and stored, this chain of commands will:
* use `echo` to produce an output string pointing to location of the keyring with our public key, and then point to the remote repository
* then it will pipe that command as input using the tee command to write that string into our APT package list for Wazuh’s verification. This allows APT package manager to easily install and update Wazuh
`echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list`

Update your package lists:<br>
`sudo apt update`


## Deploy Wazuh Agent
The following command will:
* set a temporary variable to only exist during this command. Change the address in the variable’s quotes to your Wazuh Manager’s IP address. 
* download and install the wazuh-agent package, which will run installation and configuration script/s on your machine using that variable as input.

`WAZUH_MANAGER=“<your_manager_ip>” apt-get install wazuh-agent`

Enable and start Wazuh’s service daemon:
```
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

## Check agent enrollment
After completing all of the steps, open your browser and navigate to the address of your Wazuh Manager.<br>
Log in with the username: `admin`, and the password that was assigned to `admin` towards the end of the Dashboard Installation.

The Home page will be the Overview dashboard.<br>
The “Agents Summary” on the top left should be displaying 1 Active agent.<br>
This is the agent you’ve just enrolled.

NOTE:  it might take a few minutes for the enrolled agent to show up for the Manager



## Disable automatic updates
The following line will use the `sed` command to add a # symbol to the front of our <br>
`sed -i “s/^deb/#deb/” /etc/sources.list.d/wazuh.list`



# Troubleshooting

## 1. GPG key duplication
For various reasons that might cause complications in the system, you'll probably want to heed the warnings in the apt update output, and remove duplicates. 

If you accidentally duplicated the GPG key earlier, for example, you can simply check the file with:<br>
`nano /etc/apt/sources.list.d/wazuh.list`

<duplicate-gpg-keys-error>

<duplicate-gpg-keys-warning>

## 2. Wazuh Agent not connecting
* This might be due to a network security mechanism blocking a port. Check your firewall rules. They should have the Wazuh Manager's public IP address listed as source for ports 1514 and 1515. Also, the Wazuh manager's firewall should allow for the Wazuh agent's public IP on ports 1514 and 1515.

* Otherwise, the Wazuh manager might not be listed in the Wazuh agent's configuration files. You can resolve this by accessing the config file with `sudo nano /var/ossec/etc/ossec.conf`. Be sure that instead of the agent's private IP address, that the _public IP_ is listed within the `<address>` tag, with no quotes. Then, restart the daemon.

