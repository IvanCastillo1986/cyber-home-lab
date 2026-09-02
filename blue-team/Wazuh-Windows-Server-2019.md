# Wazuh alerts on Windows Server 2019
This article takes an investigative look at Wazuh alerts on Windows Server 2019 via Sysmon.
The following are elements from an alert triggered by a Wazuh rule detecting possibly anomolous behavior from a Windows Sysmon event.

`data.win.eventdata.image = C:\Windows\System32\svchost.exe`<br>
This is the executable that ran.

`data.win.eventdata.commandLine = C:\Windows\System32\svchost.exe -k netsvcs -p -s PushToInstall`<br>
When Windows creates a process, a program is launched with a command line. The command line contains: the executable being run, and any arguments or options passed to it.

`data.win.eventdata.processId = 6488`<br>
The Windows PID assigned to that process. The important thing to remember is that PIDs are temporary and eventually get reused. PID 6488 doesn't permanently define this process.

`data.win.eventdata.processGUID = {2fb31f8d-132b-6a97-1901-000000001f00}`<br>
Sysmon generates this identifier specifically to make it easier to correlate process activity.


## We can imagine this alert as having 3 layers

### Layer 1:  Individual searchable fields
```
data.win.eventdata.image
data.win.eventdata.commandLine
data.win.eventdata.user
```
These are convenient for searching and filtering.

### Layer 2:  Original Windows/Sysmon messages
```
data.win.system.message
```
This is essentially the human-readable event content.

### Layer 3:  `full_log`
This contains the larger raw/JSON representation that Wazuh received and processed.

So the data isn't necessarily being generated three separate times. Wazuh is preserving multiple representations of the same event.


`data.win.eventdata.commandLine = C:\Windows\System32\svchost.exe -k netsvcs -p -s PushToInstall`<br>
Let's break down each part of this command.

`C:\\Windows\\System32\\svchost.exe` is the executable.<br>
`-k netsvcs` is the service group.<br>
`-p` means Protected-process-related service hosting option.<br>
`-s PushToInstall` is the specific service that svchost.exe is hosting.<br>
The full command is telling you `svchost.exe` was launched with these specific arguments to host the `PushToInstall` service.

`C:\\Windows\\System32\\svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc`<br>
`-k LocalNetworkRestricted` is the name of the service group that this svchost.exe instance belongs to.<br>
`LocalSystem` indicates the service runs under the highly privileged LocalSystem security context.<br>
`NetworkRestricted` indicates that Windows applies network restrictions to the service/group. This is part of Windows service isolation and hardening.<br>
`-p` indicates that the service is running in a more isolated svchost.exe process rather than necessarily being grouped with a bunch of unrelated services.


## Investigation Chain
Sysmon Alert  ->  CommandLine: -s StorSvc  ->  Confirm StorSvc exists  ->  Check its service configuration  ->  Find Service DLL  ->  Verify DLL location  ->  Verify Microsoft signature  ->  Check whether hashes/configuration look as expected

## Command Line Investigation Workflow
`C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc`

### `-s StorSvc`
Check whether Windows Recognizes that service:  `Get-Service StorSvc | Format-List \*`<br>
Look for things such as Name, DisplayName, Status, StartType.

Check the service configuration:  `sc.exe qc StorSvc`<br>
Compare it against the Sysmon command line. Notice that `sc.exe qc` might not show `-s StorSvc`. This argument is added when Windows starts that specific service.

### Find the DLL that the service actually loads
The executable isn't the whole story, since `svchost.exe` is only the host. You want to find the service DLL.
```
Get-ItemProperty `
    "HKLM:\\SYSTEM\\CurrentControlSet\\Services\\StorSvc\\Parameters"
```
Look for `ServiceDLL`.<br>
You might get something like `C:\\Windows\\System32\\StorSvc.dll`.<br>
Save that path mentally for next step.

`Test-Path "C:\Windows\System32\StorSvc.dll"`<br>
The expected result is `True`.

Then inspect the file:
```
Get-Item "C:\\Windows\\System32\\StorSvc.dll" |
Format-List Name,FullName,Length,CreationTime,LastWriteTime
```
A file outside normal Windows locations would deserve more scrutiny.

### Check the DLL's Digital Signature:
```
Get-AuthenticodeSignature `
    "C:\\Windows\\System32\\StorSvc.dll"
```

To be sure it was signed by Microsoft:
```
Get-AuthenticodeSignature `
    "C:\\Windows\\System32\\StorSvc.dll" |
    Format-List Status,StatusMessage,SignerCertificate
```

Verify `svchost.exe` itself:
```
Get-AuthenticodeSignature `
    "C:\\Windows\\System32\\svchost.exe" |
    Format-List Status,SignerCertificate
```

### Also verify its hash
```
Get-FileHash `
    "C:\\Windows\\System32\\svchost.exe" `
    -Algorithm SHA256
```
Compare this against the SH-256 hash reported by Sysmon.

### Check which process is actually running the service.
Your Sysmon alert gave you `ProcessId: 5680`.<br>

Check that specific process:<br>
`Get-Process -Id 5680`

You can also list all `svchost.exe` processes:<br>
`Get-Process svchost`

Print even more information on all processes, and the services they run:<br>
`tasklist /svc`<br>
You can narrow it down to just your process:<br>
`tasklist /svc /fi "PID eq 5680"`

### Check the actual command line of the running process
You can retrieve the command line through CIM:
```
Get-CimInstance Win32\_Process -Filter "ProcessId = 5680" |
Select-Object ProcessId,Name,CommandLine
```
Compare it against your Sysmon alert:
`C:\\Windows\\System32\\svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc`

Check the service's Registry configuration
Inspect the entire service registry key:
```
Get-ItemProperty `
    "HKLM:\\SYSTEM\\CurrentControlSet\\Services\\StorSvc"
```
And its parameters:
```
Get-ItemProperty `
    "HKLM:\\SYSTEM\\CurrentControlSet\\Services\\StorSvc\\Parameters"
```

Check the DLL's hash
Calculate a SHA-256 hash:
```
Get-FileHash `
    "C:\\Windows\\System32\\StorSvc.dll" `
    -Algorithm SHA256
```
Submit the DLL hash to a site like VirusTotal to determine if it's legitimate.

## Summary of investigation workflow
Wazuh Alert  ->  CommandLine `svchost.exe -k LocalSystemNetworkRestricted -p -s StorSvc`  ->  Extract service name `StorSvc`  ->  Extract service name `StorSvc`  ->  `Get-Service StorSvc`  ->  `sc.exe qc StorSvc`  ->  Find ServiceDll `HKLM:\\SYSTEM\\CurrentControlSet\\Services\\StorSvc\\Parameters`  ->  Verify DLL exists `Test-Path`  ->  Inspect DLL Get-Item  ->  Verify Signature `Get-AuthenticodeSignature`  ->  Verify svchost.exe signature + hash  ->  Connect PID to service `tasklist /svc`  ->  Compare live command line `Get-CimInstance Win32\_Process`  ->  Inspect Registry configuration
