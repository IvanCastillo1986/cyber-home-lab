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


## Creating custom policies
### Variables
Variables in policies behave the way they would in code: you assign values to them, to be used later. This might save time or space in your file, since the same values might be used in different areas throughout the policy. It’s especially useful when used for lengthy filepaths.

### Requirements
These have to be met. Otherwise, the scan won’t begin on a specific file.

Requirement Properties (with examples):
```
title: "Check that the SSH service and password-related files are present on the system"
description: "Requirements for running the SCA scan against the Unix based systems policy."
condition: any
rules:
  - 'f:$sshd_file'
  - 'f:/etc/passwd'
  - 'f:/etc/shadow
```

### Checks
Checks are only executed if requirements are met. An SCA check uses a YAML-based policy language in its rules that can execute:
1. Shell commands (such as bash or Powershell)
2. Scripts (such as Python and Perl)
3. Regular Expressions for parsing output (osregex by default, which is a regex library for C)
4. Conditional Logic to determine compliance 

Checks are the most important part of the file, since this is what tests the system during scans.

Check Properties (with examples):
```
id:			3000
title:			"SSH Hardening: Port should not be 22"
description:	"The ssh daemon should not be listening on port 22 (the default value) for incoming connections."
rationale:	"Changing the default port you may reduce the number of successful attacks from zombie bots, an attacker or bot doing port-scanning can quickly identify your 			SSH port."
remediation:	"Change the Port option value in the sshd_config file."
compliance:	
			- pci_dss: ["2.2.4"]
      			- nist_800_53: ["CM.1"]
condition:	all
rules:		
			- 'f:$sshd_file -> !r:^# && r:Port && !r:\s*\t*22$'
```

#### Condition
The result of each check is governed by conditions, and the results of the evaluation of its rules. 
It specifies how rule results are aggregated in order to calculate the final value of check.
A condition has one of three possible values:
* all  -  the check passes if all of its rules are satisfied. It fails if even one rule is not met.
* any  -  the check passes if at least one rule is met.
* none  -  the check passes if none of its rules are satisfied. It fails if even one rule is met.


#### Rules
Rules can check for the existence of: 
* files
* directories
* registry keys and values (Windows)
* running processes
* recursively check for files withn directories

Rules can also check the contents, including:
* check for file content
* recursively check for the contents of files inside directories
* check command output
* check registry value data (Windows)

There are 5 types of rules: 
* c  -  Command
* f  -  File
* p  -  Process
* r  -  Registry (Windows)
* d  -  Directory

**Operators for content checking (with examples)**
*Literal Comparison, Exact Match*<br>	
f:/file -> CONTENT<br>
**by ommision (the absence of operator signifies exact match)

*Lightweight RegEx Match*<br>
r:<br>
f:/file -> r:REGEX

*Numeric Comparison*<br>
n:<br>
f:/file -> n:REGEX_WITH_CAPTURE_GROUP<br>
compare <= VALUE

**Operators for Numeric Comparison (with examples)**<br>
**<** <br>
n:SomeProperty (\d) compare < 42<br>
**<=** <br>
n:SomeProperty (\d) compare <= 42<br>
**==** <br>
n:SomeProperty (\d) compare == 42<br>
**!=** <br>
n:SomeProperty (\d) compare != 42<br>
**>=** <br>
n:SomeProperty (\d) compare >= 42<br>
**>** <br>
n:SomeProperty (\d) compare > 42


*Checking Existence*<br>
RULE_TYPE:target

*Examples for checking existence:*<br>
f:/etc/sshd_config<br>
d:/etc<br>
not p:sshd

*Checking Content*<br>
RULE_TYPE:target -> CONTENT_OPERATOR:value

*Examples for checking content:*<br>
f:/etc/ssh_config -> !r:PermitRootLogin

f:/etc/ssh_config -> !r:^# r:Protocol && r:Protocol && r:2<br>
**NOTE:** `r:` is used for Lightweight Regex Match

More examples and information are beyond the scope of this documentation, and can be found in the Wazuh documentation:<br>
[Creating Custom Policies](https://documentation.wazuh.com/current/user-manual/capabilities/sec-config-assessment/creating-custom-policies.html)
[Wazuh Regex Reference](https://documentation.wazuh.com/current/user-manual/reference/tools/wazuh-regex.html)