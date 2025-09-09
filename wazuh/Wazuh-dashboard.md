# Wazuh Dashboard

## Install package dependencies
Before you begin installing the Dashboard itself, switch to privileged shell so that no errors occur when trying to run without root permissions. Many of these commands use the pipe `|` or `&&` operators, and you might get errors if you don’t include the `sudo` keyword after these symbols:<br>
`sudo -s`

Install the following dependencies in case they’re missing. The dashboard uses them:<br>
`apt-get install debhelper tar curl libcap2-bin #debhelper version 9 or later`

You won’t need to add the Wazuh repository, since we already accomplished that in the Indexer or Manager installation. 
If you haven’t installed those components yet, go to that documentation and install them now before proceeding with the Dashboard’s guide.

## Install Wazuh Dashboard
`apt-get -y install wazuh-dashboard`

## Configure the Dashboard
Open the configuration file with an editor:<br>
`nano /etc/wazuh-dashboard/opensearch_dashboards.yml`

The `server.host` IP `0.0.0.0` is like a catch-all to accept all IP addresses, which isn’t very safe. Replace this with your Central node’s IP address.<br>
Also replace the address at `opensearch.hosts` with the same IP address.<br>
```
server.host: your_server_address
server.port: 443
opensearch.hosts: https://<your_indexer_address>:9200
opensearch.ssl.verificationMode: certificate
```

## Deploy certificates
Open your `config.yml` configuration file:<br>
`nano config.yml`

Look at the name for your Wazuh Dashboard within this file.<br>
Assign this same exact name to your environment variable (the default name is `dashboard`):<br>
`NODE_NAME=<your_dashboard_name>`

Manage the certificates in mcuh the same way as the last two components which you should have installed in order:
```
mkdir /etc/wazuh-dashboard/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-dashboard/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME.pem /etc/wazuh-dashboard/certs/dashboard.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
chmod 500 /etc/wazuh-dashboard/certs
chmod 400 /etc/wazuh-dashboard/certs/*
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs
```

## Start the Wazuh Dashboard service
```
systemctl daemon-reload
systemctl enable wazuh-dashboard
systemctl start wazuh-dashboard
```

Open the `wazuh.yml` configuration file:<br>
`nano /usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml`

Scroll all the way down to the bottom of the configuration file.<br>
There should be a `hosts` property that isn’t commented out like the rest of the file.<br>
Replace the `url` property’s value with your Central node’s IP address , as it’s written in the `config.yml` file.<br>
Be sure to leave the`https://` scheme as is:
```
hosts:
  - default:
      url: https://<your_server_ip_address>
      port: 55000
      username: wazuh-wui
      password: wazuh-wui
      run_as: false
```

Open up your web browser. Navigate to your Wazuh dashboard’s IP address.<br>
`https://<your_server_ip_address>`

It might give you a warning that the site is unsafe, as it doesn’t recognize the certificates. But you created and deployed these yourself (using Wazuh’s configuration files and scripts). So you know that you can trust it.

Proceed through the warnings, and enter your login credentials.<br>
Remember that the default is `admin:admin`.

Congrats! You’ve finally finished the installation and configuration of your Wazuh Central node, integrating all 3 main components.<br>
But you're not done yet.


# Securing the Wazuh node
The last step in the Central node configuration is to change the default credentials, which are currently making your infrastructure vulnerable to attackers.<br>
You’ll need to change the Wazuh Server’s username and password, as well as the Wazuh Indexer.

To easily change your password, use the following command created by the good folks at Wazuh:<br>
`/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --api --change-all --admin-user wazuh --admin-password wazuh`

After the script has finished executing, you should see some output with information on what has changed.<br>
It should read a list of the users and their new passwords. In our case, Wazuh is a complex infrastructure of different services all working together under-the-hood to give us the integrated functionality that is Wazuh. In this output, you will see certain users that are actually technologies that are communicating with the Wazuh Indexer. Some of these technologies are Logstash, Kibana, snapshotrestore, etc. The most important of these lines to you will be the admin user, and your new password for logging in. Take note of this password.

Open your browser and navigate to the Wazuh website, which should be the same IP address you’ve been adding into your configurations, preceded by `https://`. 

Use the user ‘admin’ with the random password assigned to you.<br>
You now have a strong password. 


# Configuration
`/etc/wazuh-dashboard/opensearch_dashboards.yml`<br>
`/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml`