# YARA

Workflow:
1. FIM (syscheck) monitors directories on the agent and detects file creation or modification.
2. A FIM alert is generated (for example, file added or modified).
3. A custom rule on the manager matches that FIM alert.
4. That rule triggers an Active Response.
5. The Active Response executes `yara.sh` (or `yara.bat`) on the agent.
6. YARA scans the file and writes results to `active-responses.log`.
7. Custom decoders and rules on the manager parse that output and generate the final YARA alert.

Because YARA is integrated into Wazuh via the Active Response module, it does not occupy a `<wodle>` block in the `ossec.conf` file.
