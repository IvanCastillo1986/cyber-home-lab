# Using Shuffle

Important Files:<br>
`sudo docker logs -f <shuffle_container_ID>`

For example, passing in the Shuffle Worker container ID, lets you read the worker logs.<br>
Docker containers can be seen with the following command:<br>
`sudo docker ps`


## Using default workflow
Ensure that your Shuffle server is up and running.<br>
Ensure that the Docker containers are all up and running. They should all literally read `Up <amount_of_time>` on the semi-last column (6th column):<br>
`sudo docker ps`

Get on your Wazuh Manager’s Ubuntu desktop instance.<br>
Open a new tab in your browser.<br>
Navigate to your Shuffle instance’s IP address, on port `3443`:<br>
`https://<shuffle_server_ip_address>:3443`

You should now be in the Shuffle dashboard.<br>
Hover your mouse over the left-most panel.<br>
Click the arrow next to “Automate” -> “Workflows”<br>
Click on the default already-made workflow.

You’ll see a small node in the center. If you hover over that node, you can see “copy” and “delete” icons.<br>
You can test that node if you’d like. When you click on it, a panel will pop out on the right side. You’ll notice that under the “Find Actions” field, there’s an action that reads “Repeat back to me”, by default. 

You can run this default, single-node workflow by clicking on the green arrow button in the bottom panel.
It will repeat the words in the “Call” field, “Hello World” by default. This is the result from running this node. The “Result” can be passed from one node to other nodes in your Workflow dashboard.


# Integrating Wazuh with Webhooks
Create a new workflow.

Name it something descriptive like “SSH logins”.

From the left panel, click and drag the “Webhooks” trigger to the workflow canvas.<br>
Click on the Webhook trigger icon.<br>
Name it something descriptive like “receive user logins”.

Start the webhook by clicking on the “Start” button at the bottom of the Webhook tab.

Drag a “Shuffle Workflow” icon to the canvas.<br>
Click on it. Name it something descriptive like “repeat webhook”.<br>
Make sure that the “Find Actions” field reads “Repeat back to me”.<br>
Click on the “Call” field’s “+” icon. This is the autocomplete, and it will give you the available options. Choose “Runtime Argument”.<br>
This Shuffle Workflow is what will actually show us the alerts coming in from the Webhook.

Now get on your Wazuh Server and open its configuration file:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Copy and paste the following block of code, just above the Active Response section near the bottom of the file:
```
<integration>
  <name>shuffle</name>
  <hook_url></hook_url>
  <level>3</level>
  <rule_id>5715</rule_id>
  <alert_format>json</alert_format>
</integration>
```

Go back to your Shuffle dashboard. Click on the Webhook icon you’ve previously added. Copy the “Webhook URI”. 

Go back to your Wazuh `ossec.conf` file. Add the webhook URI you’ve copied in between the `<hook_url>` tags. This is what connects Wazuh and Shuffle.

Reload configuration changes:<br>
`sudo systemctl restart wazuh-manager`

Now connect any other VM instance to your Wazuh Agent or Wazuh Manager instance via SSH. This will trigger the rule with ID that 5715 was added to the `<integration>` block in your `ossec.conf` file. Rule 5715 is for successful SSH authentication. Whenever Wazuh receives an alert and it matches a rule that’s defined in the integration block, it will forward the alert to Shuffle.

Go back to your Shuffle Dashboard. Click on the `Show previous run` button with icon of the running man in the bottom left. This should display a list of any executions that match our webhook’s configured rules.

