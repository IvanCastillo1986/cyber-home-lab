# Wazuh Certificates
If you've already finished installing and connecting the Wazuh Central node (the indexer, server, and dashboard combo) to an agent, then you've probably noticed that a large portion of the installation involves the creation and distribution of cerficates and keys.

Under-the-hood, Wazuh' open-source infrastructure is much larger and complex than how it might seem from the user's standpoint. The Wazuh central Node is comprised of 3 main components. But then each of these components is further comprised of many other open-source components that you might not know or read about. For example, the Wazuh Server is comprised of the Wazuh Manager and Filebeat. Wazuh Manager handles data analysis and alerts. Filebeat handles the transport of these logs between the Manager and the Wazuh Indexer.

A technology that involves so many components communicating with each other might leave a large attack surface for malicious attackers to try to exploit. The reason Wazuh manages to be so secure is because of its processes of creating, distributing, and verifying these certificates, and it does this at each point of communication between these components. 


## Scripts
Much of the certificate creation and management is handled by some scripts created by the folks at Wazuh.
The wazuh-certs-tool.sh script uses the data inside the config.yml file to generate the certificates.
* wazuh-certs-tool.sh


## Types of certificates
There are 3 types of certificates used by Wazuh in what is called a PKI (Public Key Infrastructure) framework:

* root-ca certificate:  this is the sole authority that is in charge of signing all of the other "subordinate" certificates. There can only be one and it sits at the top of the PKI chain of trust, thus is referred to as the "trust anchor". Because there is no higher authority, its certificate is "self-signed". This means that it uses its own private key to create the cryptographic signature for its own certificate.

* node certificate:  every node in your cluster needs one of these certificates. It's how they are uniquely identified. Each of these node establishes trust by presenting either the IP address or domain name of your node. These are signed by the root CA. 

* admin certificate:  this is a client certificate with special privileges. They're used for security management throughout the Indexer cluster. They also ensure that only authorized commands are executed within the cluster.


