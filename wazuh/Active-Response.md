# Active Response
Wazuh has an Active Response module that works alongside its other modules and external integrations. This built-in module is responsible for automating responses to security events or incidents. The Active Response module comes with an assortment of out-of-the-box response scripts, which allows cybersecurity professionals to focus on more human-demanding decision tasks. 

An `<active_response>` block is found in the `ossec.conf` configuration file. It determines when and where a command executes. The `<command>` blocks are also defined in the `ossec.conf` file.<br>
An active response script can be set to trigger when an alert meets response criteria defined in the block, such as:
* a specific rule ID
* an alert level
* a rule group

Following the logic might be a bit confusing so here’s the workflow (for out-of-the-box scripts) from the top down:<br>
a response script (written in any programming language) is found in `/var/ossec/active-response/bin/`  -><br>
the `<command>` block (in `ossec.conf`) has the reference to the response script  -><br>
the `<active_response>` (also in ossec.conf) block has the reference to the `<command>`  -><br>
the `<active_response>` has a rule defined within (like rule ID, rule group, level) that matches an alert that came in  -><br>
A security event (like brute forcing) triggered an alert

Alerts are the triggers to the entire response. Everytime an alert comes in, it’s checked against certain properties in the active response engine that is constantly listening. 

Unfortunately, the Wazuh Manager doesn’t give you notification of when Active Response is triggered. Luckily, you can find this information in the logs. Logs can be found at:<br>
`sudo tail -f /var/ossec/logs/active-responses.log`


## Creating your own Active Response scripts
There are two types of active responses:  stateless and stateful.<br>
Even though they can be programmed in any language, these languages need to be able to provide certain functions for each type of active response:<br>
* Stateless:  one-time actions without an event definition to revert or stop them.
    * read STDIN to get JSON message
    * parse JSON message
    * confirm that `command` field has the `add` action
    * extract necessary information for its execution

* Stateful:  revert or stop their actions after a period of time.
    * read STDIN to get JSON message
    * parse the JSON message
    * analyze the `command` field to check if it has the `add` or `delete` actions. If `add`, the active response executes the main action. If `delete`, the active response stops or reverts the main action.
    * extract the main information for its control and execution. For example, the `firewall-drop` script uses the value in the `srcip` alert field to block or unblock an IP address.
    * build a control message in JSON format with keys extracted from alert fields. The control message contains relevant information to identify specific conditions and assess the active response. For example, it might contain a `srcip` key with the IP address to block. The script extracts this value from alert.
    * write STDOUT to send control message.
    * wait for response via STDIN.
    * parse JSON object in response.
    * analyze `command` field to check if it has to `continue` or `abort` execution. For example, it might need to abort the action if it’s repeating a previous active response action that hasn’t timed out yet and it’s already in execution.