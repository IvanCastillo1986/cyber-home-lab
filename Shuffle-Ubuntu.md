# Shuffle
In this documentation, you'll be installing Shuffle on an Ubuntu instance.

## What is Shuffle?
Shuffle is an open-source SOAR (Security Orchestration, Automation and Response) tool, which integrates well with other apps, such as our Wazuh SIEM and VirusTotal. It takes action in response to our system detecting a security event.

It can be triggered by:
* Webhook:  the SIEM sends an alert to shuffle
* Schedule:  it can be configured to automatically run, like cronjobs
* API call:  manual trigger from an integrated external system

Once installed, you can access Shuffle from a web browser.
Using the web UI, workflows can be built using a visual drag-and-drop interface.

### Shuffle Workflow
1. Wazuh detects something suspicious (brute force from some IP)  ->
2. Wazuh sends webhook to Shuffle  ->
3. Shuffle workflow automatically triggers  ->
4. Shuffle checks with VirusTotal:  “is this IP malicious?”  ->
5. VirusTotal responds:  “Yes, score is 90/100”  ->
6. Shuffle automatically tells firewall to block this IP  ->
7. Shuffle hands message to Slack:  “Alert: blocked malicious IP 192.168.105.62”  ->
8. Security team reads alert on Slack, and can investigate further


## Vocabulary
* Webhook  -  a user-defined HTTP callback function that allows two APIs to communicate with each other in an event-driven way. Webhooks are triggered by an event in a web application and can send data and executable commands from one app to another over HTTP. 
    - Webhooks are automated, in other words, they are automatically sent out when their event is fired in the source system.
    - To recieve Webhook requests, you have to register for one or more of the events (aka topics) for which the platform offers a webhook. 
    - A webhook request will be sent to a destination endpoint on your application so you need to build one for it and register the URL as the *Webhook URL* for that event.


# Create Ubuntu instance
Shuffle is a memory-intensive platform, so be sure to provision at least 6GB of RAM. You'll also want to give it an ample amount of storage. But if you're like me, and you only have 256GB of storage total, you might want to provision the smallest amount of storage without worrying too much about running out of space. I created my Ubuntu instance with 20 GB, and extended the logical volume. 

Instructions for creating an Ubuntu virtual machine can be found here:<br>
[Ubuntu](Ubuntu.md)

Instructions for extending storage to full amount can be found here:<br>
[Extend storage](/processes/Extend-Ubuntu-Storage.md)


### Update Ubuntu instance
Run:<br>
```
$  sudo apt update
$  sudo apt upgrade
```
...to update the debian dependencies.


# Install Docker Engine and Docker Compose
This installation is for your Ubuntu server instance, with no graphical interface.<br>
Installation instructions can be found at:<br>
[Docker](Docker.md)


# Installing Shuffle
Be sure to execute this section only after installing Docker and Docker Compose.

Locally clone the Shuffle git repository:<br>
`git clone https://github.com/shuffle/Shuffle`

Change into the new `Shuffle` directory:<br>
`cd Shuffle`

Run the `docker-compose.yml` configuration:<br>
`sudo docker compose up -d`

Check the currently running Docker containers:<br>
`sudo docker ps`

...as you can see, there are a whole lot of Shuffle containers running after executing the `docker compose up` command!
