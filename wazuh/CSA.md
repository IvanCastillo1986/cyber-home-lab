# SCA module
Security Configuration Assessment (SCA) is a way of looking for vulnerabilities in systems to verify that all of the security checkboxes have been checked. It’s key to achieving system hardening and reducing the attack surface as much as possible (or as much as the company is willing to tolerate risks). 

Wazuh has a SCA module. It performs scans in the system to expose weaknesses and misconfigurations that can be exploited. During a scan, the configuration of the endpoint is tested against rules in policy files. Wazuh has out-of-the-box policies based on CIS benchmarks, that check files, directories, registry keys and values, and active processes. 

The current state of SCA checks is stored in a local database of each Wazuh Agent. The Wazuh Server has its own database that’s comprised of SCA scan results of all agents. The server only updates its own database when there is a change detected in the corresponding agent, which helps to keep traffic and resource consumption to a minimum. The does this by keeping a hash of its own database, and comparing it with the hash on the agent’s database. When the hash differs, the server asks for the agent’s latest scan before making changes to its own database. 

The checks of an SCA policy contain fields that determines what actions the agent should take to scan the endpoint, and how to evaluate scan results. You can also create your own policies. Each definition of a check is comprised of:
* Metadata including rationale, remediation, and description of the check.
    * The metadata can include an optional compliance field which specifies if the check is relevant to any compliance specifications. Even though it’s optional, I would highly recommend using this field, since the point of SCA is to determine whether the setup complies with standards and policies.
* A logical description with the `condition` and `rules` fields.

