# SCA module
Security Configuration Assessment (SCA) is a way of looking for vulnerabilities in systems to verify that all of the security checkboxes have been checked. It’s key to achieving system hardening and reducing the attack surface as much as possible (or as much as the company is willing to tolerate risks). 

Wazuh has a SCA module. It performs scans in the system to expose weaknesses and misconfigurations that can be exploited. During a scan, the configuration of the endpoint is tested against rules in policy files. Wazuh has out-of-the-box policies based on CIS benchmarks, that check files, directories, registry keys and values, and active processes. 

The current state of SCA checks is stored in a local database of each Wazuh Agent. The Wazuh Server has its own database that’s comprised of SCA scan results of all agents. The server only updates its own database when there is a change detected in the corresponding agent, which helps to keep traffic and resource consumption to a minimum. The does this by keeping a hash of its own database, and comparing it with the hash on the agent’s database. When the hash differs, the server asks for the agent’s latest scan before making changes to its own database. 

The checks of an SCA policy contain fields that determines what actions the agent should take to scan the endpoint, and how to evaluate scan results. You can also create your own policies. Each definition of a check is comprised of:
* Metadata including rationale, remediation, and description of the check.
    * The metadata can include an optional compliance field which specifies if the check is relevant to any compliance specifications. Even though it’s optional, I would highly recommend using this field, since the point of SCA is to determine whether the setup complies with standards and policies.
* A logical description with the `condition` and `rules` fields.


## Adding shared agent configuration
The centralized agent configuration file can be set up to include policies to be pushed to every agent. This pretty much means that you can apply rules to every single agent linked to your manager by conveniently storing it in one place, instead of individually adding the same rules to every single agent. 

There are 4 types of rules: 
* c  -  Command
* f  -  File
* p  -  Process
* r  -  Registry

The `c` rule type will need an extra configuration step on each agent.<br>
Use the following command on each agent to allow execution of commands in SCA policies that use the `c`rule type from its Wazuh Server:<br>
`echo “sca.remote_comands=1” >> /var/ossec/etc/local_internal_options.conf`

From the Wazuh Server, add the new custom policy file, and change its ownership to Wazuh user and group.<br>
Enter your policy file’s name within <new_policy_file>:<br>
`chown wazuh:wazuh /var/ossec/etc/shared/default/<new_policy_file>`

In your Wazuh Server, add the following block of code to the `/var/ossec/etc/shared/default/agent.conf` file. Replace <new_policy_file> with your file’s name:
```
<agent_config>
  <!-- Shared agent configuration here -->
  <sca>
    <policies>
        <policy>/var/ossec/etc/shared/default/<new_policy_file></policy>
    </policies>
  </sca>
</agent_config>
```

This `<agent_config>` tag is where you will centrally add any configurations that you’d like all of your agents to inherit. The `<sca>` block you’ve just added is merged with the `<sca>` block on the Wazuh agent. 