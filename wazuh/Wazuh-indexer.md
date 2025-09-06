# Wazuh Indexer installation
This is the first component you'll want to install.<br>
Deviating from installing this first will lead to missing steps during the install of the other components, such as management of certificates.

We’ll be using the 192.168.101.0/24 - 192.168.104.0/24 network ranges for this project as a demonstration, split into different subnets to house the appropriate instances. Feel free to change that to whichever network you’re comfortable with.

The Wazuh Agent’s IP address will be 192.168.101.102.<br>
The Central Node’s IP address will be 192.168.103.101.

The Central Node is comprised of 3 components:
* Wazuh Indexer
* Wazuh Server
* Wazuh Dashboard


## Install Indexer
We’ll be installing it as a single-node cluster for now, and then later cover the changes in configuration if we need to install more nodes.

### Create Certificates
Download `wazuh-certs-tool` script:<br>
`curl -sO https://packages.wazuh.com/4.12/wazuh-certs-tool.sh`

This script will cover the creation and storage of the Root CA. 

But the script alone won’t work without the `config.yml` configuration file:<br>
`curl -sO https://packages.wazuh.com/4.12/config.yml`

The script and configuration file should now be in your working directory.

Edit the `./config.yml` file to replace the node name with IP address of the Server, Indexer, and Dashboard components. They will all have the same address, as our GNOME instance will act as the Central Node which integrates these 3 components.

Replace <indexer-node-ip>, <wazuh-manager-ip>, and <dashboard-node-ip> with your central node machine’s address.<br>
In the example below, I’ve already replaced them with this machine’s address:<br>
```
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: node-1
      ip: “192.168.103.101”
    #- name: node-2
    #  ip: "<indexer-node-ip>"
    #- name: node-3
    #  ip: "<indexer-node-ip>"

  # Wazuh server nodes
  # If there is more than one Wazuh server
  # node, each one must have a node_type
  server:
    - name: wazuh-1
      ip: "192.168.103.101"
    #  node_type: master
    #- name: wazuh-2
    #  ip: "<wazuh-manager-ip>"
    #  node_type: worker
    #- name: wazuh-3
    #  ip: "<wazuh-manager-ip>"
    #  node_type: worker

  # Wazuh dashboard nodes
  dashboard:
    - name: dashboard
      ip: "192.168.103.101"
```

Save the file.

Before you can run the script, you’ll need to change it’s permissions to allow execution:<br>
`sudo chmod +x ./wazuh-certs-tool.sh`

You can check that you now have the proper permissions. There should now be an x across it’s permissions listing (for execute permissions):<br>
`ls -l ./wazuh-certs-tool.sh`

The script manages the creation of certificates of the Wazuh components. The -A flag creates certificates specified in config.yml and admin certificates. Type the following command to run it:<br>
`./wazuh-certs-tool.sh -A`

There should now be a new `wazuh-certificates` directory in your working directory, which contains many .pem files.<br>
.pem is a container file format that’s used to store and transmit cryptographic objects, such as:<br/>
* certificates
* private keys
* public keys
* certificate signing request (CSR)

Use the following command to compress and store the newly created certificates directory into a tar object. You can copy it to other Indexer, Server, or Dashboard nodes with the `scp` command if needed:<br>
`tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .`

You can now delete the original directory:<br>
`rm -rf ./wazuh-certificates`


### Install the Node
Install the following package dependencies (in case they’re missing):<br>
`sudo apt install debconf adduser procps`


#### Add the Wazuh repository
Install the following packages if missing:<br>
`sudo apt install gnupg apt-transport-https`

Download and install the PGP key (joined with a pipe |) and change its permissions so that it’s readable by every user:<br>
`sudo curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg`

The underlying cryptographic verification is handled by a tool called gpgv, which doesn’t run with full root priveleges, which is why the permissions are changed at the end of this command. If the read permissions weren’t added to Group and Read users, it would not be able to be read by the APT package manager when updating and reading the key. Running apt update in the future would be interrupted by an error.

Run apt update to make sure that package lists can be read (and their certificate signatures can be verified):<br>
`sudo apt update`


## Install the Wazuh indexer
`sudo apt -y install wazuh-indexer`


## Configure the Wazuh indexer
Here you’ll be editing the  `/etc/wazuh-indexer/opensearch.yml`  file.<br>
But before that, you’ll want easy access to the /etc/wazuh-indexer directory and all of it’s nested content. This gives you root shell privileges, but keeps your shell’s environment variables and configurations:<br>
`sudo -s`

Edit the /etc/wazuh-indexer/opensearch.yml configuration file:<br>
`sudo nano /etc/wazuh-indexer/opensearch.yml`

* network.host:  The Central Node will bind to this IP address and use it as its publish address. Give it your Machine’s IP address from the config.yml file.

* node.name:  You can just use the default name `node-1` as defined in the config.yml file.

* discovery.seed_hosts:  Since we’re only configuring the Wazuh Indexer on our single Central Node VM, we can leave this commented out. If we decide to add multiple nodes for the indexer, we’ll need to return here and configure this.

* plugins.security.nodes_dn:  The first node (our node-1) is already uncommented. Return to uncomment other nodes if you need to later. This is a list of the distinguished names of certificates of all the Wazuh Indexer cluster nodes.

That’s all we need from here for our desired single-node configuration. Save the text config file.


## Deploy certificates

First, let’s use the following command in the terminal to create a bash shell environment variable:<br>
`NODE_NAME=node-1`

We’ve just saved a global environment variable.<br>
Now, whenever you run a command in Bash with a dollar sign $ operator, the successive characters will invoke that environment variable’s value. It’s like a programming language’s variables, except to be stored and accessed within a CLI’s shell. We’ll need to pass in our `node-1` value to some of the following commands.

Create the `/etc/wazuh-indexer/certs` directory:
`mkdir /etc/wazuh-indexer/certs`

Unzip the contents of the wazuh-certificates.tar file that we compressed a few steps ago, and store them within the previously created `/certs` directory:<br>
`tar -xf ./wazuh-certificates.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem`

Change the name of the `node-1.pem` file:<br>
`mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem`

Change the name of the `node-1-key.pem` file:<br>
`mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem`

Change the permissions of the /certs directory, so that it’s only readable and executable by Owner:<br>
`chmod 500 /etc/wazuh-indexer/certs`

Change the permissions of the /certs directory’s contents (* is a wildcard, meaning “everything”), so that it’s only readable by Owner:<br>
`chmod 400 /etc/wazuh-indexer/certs/*`

Recursively change the User Owner and Group Owner of the `/certs` directory, and all files within it to `wazuh-indexer`:<br>
`chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs`

You can now exit the privileged shell:<br>
`exit`


### Start the service

Reload systemd’s configuration files, in case it’s not aware of the wazuh-indexer service:<br>
`sudo systemctl daemon-reload`

Enable the Wazuh Indexer’s service:<br>
`sudo systemctl enable wazuh-indexer`

Start the Wazuh Indexer’s service:<br>
`sudo systemctl start wazuh-indexer`


### Initialize cluster

Run the Wazuh Indexer’s `indexer-security-init.sh` shell script on this VM. This script only needs to be run once now on this node, to load the new certificates information for the entire cluster. No need to return later if you add new nodes to the cluster:<br>
`sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh`


## Test cluster installation

Check that the installation succeeded by retreiving its JSON metadata. Replace the bracket’s contents <> with your Indexer’s IP address:<br>
`curl -k -u admin:admin https://<your_indexer_ip_address>:9200`

Check that the newly installed single node cluster is working correctly:<br>
`curl -k -u admin:admin https://<your_indexer_ip_address>:9200/_cat/nodes?v`

Alternatively, you can open a browser and access the site through the URL bar. The username and password will originally be `admin` and `admin` (until you change it like a good cybersecurity professional!).