# Wazuh Vulnerabilities
Wazuh can track down vulnerabilities. 

Using your Wazuh Manager’s terminal, open the global configuration file, and scroll down about 117 lines:<br>
`sudo nano /var/ossec/etc/ossec.conf`

Toggle on the vulnerability detector:
```
<vulnerability_detector>
	<enabled> no </enabled>
</vulnerability_detector>
```
-> &nbsp; and change to &nbsp; ->
```
<vulnerability_detector>
	<enabled> yes </enabled>
</vulnerability_detector>
```