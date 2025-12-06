# Using Active Response
Let’s directly test the Active Response module. 

Let’s use the Kali VM as our attack machine, and the Wazuh Agent Ubuntu server as our victim machine. The Kali instance will execute a brute force on the agent’s SSH port. 

Create a wordlist in your Kali’s home page:<br>
`sudo nano fruits-wordlist`

Add 10 lines of words to your wordlist:
```
banana
apple
orange
strawberry
melon
blueberry
raspberry
kiwi
blackberry
cherry
```

During the attack, Kali will use a brute-force pen-testing tool called Hydra. We will pass in the wordlist as an argument. Hydra will go down the list and try every line of words as a password. This should trigger our Active Response script.

On the Wazuh Server, open the `ossec.conf` file. Scroll down about halfway til you find the section with `<command>` tags. Add in the following block of code as the last list of commands:
```
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>
```
Save and exit.

Now, scroll down all the way to the bottom and add the following block of code. From now on, you can separate your code by adding in separate blocks inside of an `<ossec_config>` block. This way you know where to find your code during troubleshoots:
```
<ossec_config>
  <active-response>
    <disabled>no</disabled>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5763</rules_id>
    <timeout>180</timeout>
  </active-response>
</ossec_config>
```

Restart the Wazuh Manager service to load the changes:<br>
`sudo systemctl restart wazuh-manager`


## Test configuration
First, test your connection by using the Kali VM to ping the agent:<br>
`ping <agent_ip_address>`

The active response we’ll be testing is the “Firewall Drop” script. The name says it all. It will use iptables to render it inaccessible to the attacking IP that triggered it. Be sure to have optables installed on your Wazuh Agent. If you created a minimized Ubuntu install for your agent, you might need to install it:<br>
`sudo apt install iptables`

Hydra should be conveniently installed on your Kali VM, since it’s a pen-testing tool. Kali has this and a bunch of other apps like it pre-installed out-of-the-box. 

Use the wordlist in your home directory, the Wazuh Agent’s username and its IP address in the following command:<br>
`sudo hydra -t 4 -l <agent_username> -P <path_to_wordlist> <agent_ip_address> ssh`

The cursor will hang while it’s executing the brute-force. Afterwards, it’ll give a summary on how it went.

Try pinging the agent from Kali:<br>
`ping <agent_ip_address>`

It should not be accessible. If you look back at the `<active_response>` block of code you added earlier, you might have noticed the `<timeout>` block. This will be the duration for which the attack machine can no longer access the victim machine, declared in amount of seconds. After the time elapses, you’ll be able to attack again.

If you navigate to the Wazuh Dashboard, you should see an alert for the previous `sshd` failed password connection attempt. 

On your Wazuh agent, you can also check the output for the Active Response modules log file:<br>
`sudo tail -f /var/ossec/logs/active-responses.log`

Also note that because this particular Active Response’s `<command>` block has a `<timeout>`, that would make this a “Stateful” response, since it needs to keep track of a timer before performing another action. It’s not a one-time stateless response. 