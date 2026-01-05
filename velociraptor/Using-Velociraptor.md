# Using Velociraptor

## Artifacts
are scripts that perform a particular action related to your investigation or data gathering.
There are many artifacts included out-of-the-box in Velociraptor. There are also 1000+ custom community artifacts that you can download, for even more streamlined actions. This allows you to be as broad or as narrow as you’d like in your investigation. 

In Velociraptor, artifacts are used as containers for VQL code and queries. In the Virtual Filesystem (VFS), whenever you choose to expand a directory, that’s actually an artifact being run under-the-hood. You’re actually sending VQL code wrapped in an artifact container to the client. Once the data is sent back to the server, the browser application can then display it.


## Selecting Host
First, we can specify the host to perform functions on by clicking on the down arrow in the top-most toolbar, right next to the search bar. Then click on “Show All” to list off all of the agent machines. Once selected, this host is available in the application globally. This means that any actions or investigating will be done on the globally selected agent machine.


## Interrogation
is really just the process of querying a host for its basic host information. Everytime a new client is enrolled, the server schedules a collection of the `Generic.Client.Info` artifact on the client. This data is used to populate the client’s database on the server with information. Some of these fields are indexed so that you can perform fast searches for this client. In older versions of Velociraptor, the user would need to manually run this artifact during a hunt in order to populate the client’s basic data.


## Monitoring
An effective client monitoring framework:<br>
* A number of event plugins (usually start with the word “watch”) that can detect events on the endpoint:<br>
    * watch_syslog() allows following of syslog events
* the rest of the VQL query can process these events:<br>
    * applying further filtering or enriching with additional data
* A client monitoring architecture is used to ensure event queries are always running and forwarding the events to the server

The **Client Monitoring Table** (event table) is a set of **VQL Event Queries** that are run in parallel. It gets synced with the server when necessary. These event queries are stored locally in the client’s “writeback file” so that they’re always available, online and offline. If it’s offline, the result of the queries are stored in the client’s `Client.local_buffer` until the client is back online, and the events are finally sent to the server. The client can monitor for events independently from the server via its monitoring table.

This is accessed by clicking on the last icon (the binoculars) in the left toolbar known as “Client Events”. These artifacts are different from the artifacts that are chosen for Hunts. These artifacts are used for monitoring over time, whereas Hunt artifacts are enacted on the current state of the system right now. For example, you can monitor with the artifact named “Linux.Events.SSHLogin”, which will monitor over time for users attempting to log in via SSH. This mode complements the Hunt mode, because it alerts to events that are currently happening. After observing investigative information, you can choose to customize the artifacts that are used during the Hunt for special use cases.

You can initiate monitoring by clicking on:<br>
“Client Events”  ->  **pen-paper icon**  ->  <label_to_monitor>  ->  “Select Artifacts”  ->  “Launch”

The event queries are run indefinitely as soon as the client starts. When an event query produces a row, the event is streamed to the server’s datastore.

You can view the events by clicking on:<br>
“Client Events”  ->  “Select artifact”  ->  <your_event_artifact_to_review><br>
The timeline view at the top allows you to click to the place in time that you’d like to inspect. The details of the events are found beneath the timeline in a table-like structure. 
