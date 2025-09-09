# Wazuh Server

This Wazuh component receives data from agents, external APIs, and network devices.<br>
It then correlates and matches that data against a predefined rulest. If matches are found, it will generate alerts for threats and anomalies.<br>
These generated alerts and data can be used by security professionals for monitoring and management of networks and systems.

The Wazuh Server itself is comprised of two components:
* Wazuh Manager:  this is the closest thing to the brain of the Wazuh SIEM system. This is what collects and analyzes data received from the agents. It’s also what triggers alerts for security events.
* Filebeat:  this works closely with the Manager, by securely forwarding the security events to the Wazuh Indexer, via TLS encryption.

In this portion, we’ll be installing both.<br>
The Wazuh Server will be installed on the same VM as the Wazuh Indexer, joining it as the Central Node.



# Install Server
The Server installation will skip some of the steps outlined in the official documentation, since we’ve already covered those steps in the Indexer installation.<br>
Let’s try something different now, and work in a non-login shell for root. This gives us all the power of root privleges that we’d get from sudo su, or sudo -i.

## Install Wazuh Manager
Install this package through the APT manager:<br>
`apt install -y wazuh-manager`

## Install Filebeat
Also install this package through the APT manager:<br>
`apt -y install filebeat`

## Configure Filebeat
Download the Filebeat configuration file:<br>
`curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.12/tpl/wazuh/filebeat/filebeat.yml`

Filebeat has automatically created a directory for itself. You can find the configuration file inside.
Let’s edit this configuration:<br>
`nano /etc/filebeat/filebeat.yml`

Most lines of this preconfigured file is commented out. <br>
Scroll down near the bottom of the file, where you will find the “Elasticsearch Output” section.

In this section, you’ll want to do a few things.<br>
Since we’re installing Wazuh Indexer to the same Central node, add the IP address to the VM you’re currently on, inside the quotes. Add port `:9200` directly after it. <br>
Uncomment the line with `protocol: “https”`.<br>
Uncomment the `username` and `password` lines and replace it with the variables `${username}` and `${password}`.<br>
In the example below, I’ve already edited the file:

```
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: [“192.168.103.101:9200”]

  username: ${username}
  password: ${password}
```

Port 9200 is used by Elasticsearch to listen to external TCP traffic.<br>
Save and exit the configuration file.

Filebeat Keystore is a feature which Filebeat uses to manage your sensitive data. It allows Filebeat to store credentials (such as passwords) securely, instead of writing in your credentials into the Filebeat configuration file in plaintext. You assign your secret string to a variable, and Filebeat will pass in this variable, which obscures your credential. The value stored in external systems such as GitHub will just look like the variable name as the property.

Create a Filebeat keystore. <br>
`filebeat keystore create`

Add the default “admin”:”admin” username and password into the keystore (these can be changed later):<br>
`echo admin | filebeat keystore add username --stdin --force`<br>
`echo admin | filebeat keystore add password --stdin --force`

Download alerts template:<br>
`curl -o /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.12.0/extensions/elasticsearch/7.x/wazuh-template.json`

Add read permissions to Group and Others:<br>
`chmod go+r /etc/filebeat/wazuh-template.json`

Download a compressed version of the Wazuh module for Filebeat. Then extract the contents into the selected directory:<br>
`curl https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz | tar -xvz -C /usr/share/filebeat/module`

## Deploy certificates
The .tar file with the compressed certificates should already be in your home directory (or wherever you stored them), from the Wazuh Indexer installation. Change into the directory with the `wazuh-certificates.tar` file before continuing.

Now go into your `config.yml` file.
Be sure to look closely at the name of your SERVER, and NOT the name of your indexer (node-1 by default).
The default name of the server is `wazuh-1`, so it should still be that unless you purposely changed it.

Much like the Indexer installation, we’re going to assign an environment variable to pass in to upcoming commands. The node name should be the same as the value you passed into your Wazuh Indexer (for this documentation it was node-1). Just be sure it’s consistent with the name you assigned to it in the `config.yml` file. Do not store it in quotes. Wazuh will use the names in these files when verifying or creating keys and signing certificates, so be sure that the information is accurate so you don’t have difficult errors down the line:
$ NODE_NAME=<your_server_name>

The following process will be very similar to the certificate deployment process you followed earlier, so just type in the commands in the following order:<br>
```
mkdir /etc/filebeat/certs
tar -xf ./wazuh-certificates.tar -C /etc/filebeat/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/filebeat/certs/$NODE_NAME.pem /etc/filebeat/certs/filebeat.pem
mv -n /etc/filebeat/certs/$NODE_NAME-key.pem /etc/filebeat/certs/filebeat-key.pem
chmod 500 /etc/filebeat/certs
chmod 400 /etc/filebeat/certs/*
```

Instead of changing the owner to `wazuh-indexer`, this time we’ll change this `/etc/filebeat/certs` directory’s owner as root:<br>
`chown -R root:root /etc/filebeat/certs`

## Configure connection to Wazuh Indexer
In this step you’ll be using `wazuh-keystore`, which is a binary file which runs a program created by the developers of Wazuh. This program securely stores your secret username and password credentials in a keystore. This eliminates the need to store in plaintext in our configuration file. 

Replace `<INDEXER_USERNAME>` and `INDEXER_PASSWORD` with `admin` (if you set the Indexer password to something else, write that in instead):<br>
`echo '<INDEXER_USERNAME>' | /var/ossec/bin/wazuh-keystore -f indexer -k username`
`echo '<INDEXER_PASSWORD>' | /var/ossec/bin/wazuh-keystore -f indexer -k password`

Let’s edit the `/var/ossec/etc/ossec.conf`, Wazuh Manager’s main configuration file:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Scroll down the file until you find the <indexer> tag. Nested inside that tag, you’ll find the <host> tag. Replace the default value of `https://0.0.0.0:9200` with your IP address. Be sure to include `https://` before and the `9200` port number after (in the example below, I’ve already replaced the IP address):<br>
```
<indexer>
  <enabled>yes</enabled>
  <hosts>
    <host>https://192.168.103.101:9200</host>
  </hosts>
  <ssl>
    <certificate_authorities>
      <ca>/etc/filebeat/certs/root-ca.pem</ca>
    </certificate_authorities>
    <certificate>/etc/filebeat/certs/filebeat.pem</certificate>
    <key>/etc/filebeat/certs/filebeat-key.pem</key>
  </ssl>
</indexer>
```

Also, ensure that the values within the <certificate> tags contain the proper absolute path and names to your `filebeat.pem` and `filebeat-key.pem` certificates. You can check this by running:<br>
`ls /etc/filebeat/certs`

## Start the Wazuh Manager
`systemctl daemon-reload`<br>
`systemctl enable wazuh-manager`<br>
`systemctl start wazuh-manager`

Verify the Wazuh Manager daemon’s status:<br>
`systemctl status wazuh-manager`

Finally, verify that your Wazuh Manager installation works:<br>
`filebeat test output`

It should read like the following:<br>
```
elasticsearch: https://192.168.103.101:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 192.168.103.101
    dial up... OK
  TLS...
    security: server's certificate chain verification is enabled
    handshake... OK
    TLS version: TLSv1.3
    dial up... OK
  talk to server... OK
  version: 7.10.2
```

Congrats! You’re almost done installing the Wazuh Central node.


# Configuration Files
`/etc/filebeat/filebeat.yml`<br>
`/var/ossec/etc/ossec.conf`