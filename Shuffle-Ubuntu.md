# Shuffle
In this documentation, you'll be installing Shuffle on an Ubuntu instance.


## Vocabulary
* Webhook  -  a user-defined HTTP callback function that allows two APIs to communicate with each other in an event-driven way. Webhooks are triggered by an event in a web application and can send data and executable commands from one app to another over HTTP. Webhooks are automated, in other words, they are automatically sent out when their event is fired in the source system.
To recieve Webhook requests, you have to register for one or more of the events (aka topics) for which the platform offers a webhook. 
A webhook request will be sent to a destination endpoint on your application so you need to build one for it and register the URL as the *Webhook URL* for that event.


# Create Ubuntu instance
You'll want to give it an ample amount of storage. But if you're like me, and you only have 256GB of storage total, you might want to provision the smallest amount of storage without worrying too much about running out of space. I created my Ubuntu instance with 20 GB, and extended the logical volume.

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


<!-- TODO:  continue installation of Shuffle in Docker container -->