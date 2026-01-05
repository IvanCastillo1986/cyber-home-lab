# Velociraptor
Velociraptor is an open-source Digital Forensics and Incident Response (DFIR) platform that gives deep visibility into endpoints. It collects artifacts via its powerful VQL (Velociraptor Query Language) in investigations. While Wazuh detects threats, Velociraptor can investigate and respond to them.


## Velociraptor Features
* Endpoint Visibility:  allows user to see everything, from currently running processes to network connections.
* Artifact Collection:  it automatically collects evidence that might be specific to the incident.
* Timeline Reconstruction:  allows for a reconstruction of what happened and when it happened.
* Real-time monitoring across multiple endpoints


## Reference
opt/velociraptor  :  datastore directory where Velociraptor stores all files
opt/velociraptor/logs  :  where logs are stored

8000  :  Frontend port to listen on
8889  :  port for the GUI to listen on

velociraptor_server  :  Velociraptor server’s systemd service (via systemctl)
velociraptor_client  :  Velociraptor client’s systemd service (via systemctl)


## Install the Velociraptor server
We'll be able to install this application on a simple [Ubuntu](./Ubuntu.md) server. It doesn't use up much storage, so you can pair it with your Shuffle, to save on storage from installing a new OS on a new instance.

Create a directory to store installation preparation items, then change to it:<br>
`mkdir ~/velociraptor_setup && cd ~/velociraptor_setup`

Download the binary specific to the Apple m-chip’s ARM architecture:<br>
`wget -O velociraptor https://github.com/Velocidex/velociraptor/releases/download/v0.75/velociraptor-v0.75.6-linux-arm64`

Change permissions to allow execution:<br>
`chmod +x velociraptor`

This command starts the wizrd which creates the server’s YAML configuration file. This file defines how the server operates as well as some cryptographic oeprations necessary during deployment (like setting up self-signed keys/certificates):<br>
`./velociraptor config generate -i`

Choose default values for everything except:<br>
`Deployment Type: Self Signed SSL`

`What is the public DNS name of the Master Frontend? <your_server_IP_address>`

At the `Username` creation, you can add the user `admin` and a password, then a blank username and password to continue. The user `admin` will be used at initial sign-in. If you want more regular users, you can always configure that later.

After the configuration wizard finishes, you’ll be met with your terminal’s prompt. 

Your `velociraptor/` directory will now have a new configuration file. Open it with your text editor:<br>
`sudo nano server.config.yaml`

Scroll down and change the Gui’s `bind_address` property to accept all traffic:
```
GUI:
    bind_address: 0.0.0.0
```

Create the server installation package with the following command:<br>
`./velociraptor debian server --config ./server.config.yaml`

Install the server package for the ARM architecture with:<br>
`sudo dpkg -i velociraptor-server-0.75.6.arm64.deb`

Check that your newly installed service is running:<br>
`sudo systemctl status velociraptor_server`

NOTE: if you get an error underneath the service’s status, saying:<br>
`Unable to open file /opt/velociraptor/config/inventory.json.db: open /opt/velociraptor/config/inventory.json.db: permission denied`

...then all you’ll need to do is change the file’s owner to `velociraptor`:<br>
`sudo chown velociraptor:velociraptor /opt/velociraptor/config/inventory.json.db`

Now get on your Ubuntu GNOME desktop instance. Open the browser. Replace <velociraptor_server_IP> with its actual IP address, followed by port `8889`:<br>
`<velociraptor_server_IP>:8889`


## Installing the agent
This agent will be installed on an Ubuntu server. You might even want to consolidate storage and place it on the same instance as another application, such as Shuffle.

First you’ll need to create the client config. Use your Ubuntu GNOME desktop instance’s browser to navigate to the Velociraptor dashboard at `<velociraptor_server_IP>:8889`. Click on the home icon on the top left. Scroll down to the section that reads “Current Orgs”. Click on the `client.root.config.yaml` button to download the config file. 

Change into the `Downloads` directory:
```
cd
cd Downloads
```

Use the `scp` command to securely copy the config file to your client:<br>
`sudo scp client.root.config.yaml <client_username>@<client_IP_address>/home/<client_username>`

From your Velociraptor server, do the same thing with the `velociraptor` binary:<br>
`sudo scp velociraptor <client_username>@<client_IP_address>/home/<client_username>`

From your client endpoint, check that you have both files:<br>
`ls`

Now that you have both the binary and the client configuration file, create the installation package:<br>
`./velociraptor debian client --config client.root.config.yaml`

Check that the new package is in your current directory:<br>
`ls`

Install the package:<br>
`sudo dpkg -i velociraptor_client_0.75.6_arm64.deb`

Now that you’ve finally installed the agent, check that the service is up and running:<br>
`sudo systemctl status velociraptor_client`

Congrats! You now have a deployed agent ready to be investigated.