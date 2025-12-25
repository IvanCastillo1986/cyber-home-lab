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

