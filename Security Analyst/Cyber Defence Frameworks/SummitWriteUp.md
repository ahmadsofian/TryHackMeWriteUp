## 📔Summit Challenges Writeup

### Blocking sample1.exe
1. Navigate to the Malware Sandbox page and submit the sample1.exe for analysis.
2. Among results listed there are hashes of the file. This sounds like a unique way to distinguish this malware.
3. Navigate to the Manage Hash page. Paste the MD5 hash.
4. Effectively block the sample1.exe to get the flag for this challenge.

### Blocking sample2.exe
1. Navigate to the Malware Sandbox page and submit the sample2.exe for analysis.
2. Noticed that the malware sends a HTTP GET requests to 154.35.10.113 over port 4444. Presumably, this IP address points to Sphinx's C2C server.
3. Navigate to Firewall Manager and create new firewall rule.
4. In the firewall rule, we specify type *Engress*, source IP *Any*. destination IP 154.35.10.113 and action *Deny*.
5. Effectively block the sample2.exe to get the flag for this challenge.

### Blocking sample3.exe
1. Navigate to the Malware Sandbox page and submit the sample3.exe for analysis.
2. Found out that Sphinx has relied on the domain *emudyn.bresonicz.info*.
3. Indicate that Sphinx use DNS to map the regularly-rotated IP address of the C2 server to this name, so we need to block the domain.
4. Navigate to DNS Filter and create new rule for this.
5. Specify the domain name *emudyn.bresonicz.info* and action as *deny*
6. Effectively block the sample3.exe to get the flag for this challenge.

### Blocking sample4.exe
1. Navigate to the Malware Sandbox page and submit the sample4.exe for analysis.
2. There is indeed some registry activity reported. The first modification is listed as *DisableRealtimeMonitoring*.
3. This align with the defence evasion of MITRE ATT&CK tactic TA0005, Sphinx is disabling the detection measure provided by Windows Defender.
4. Navigate to Sigma Rule Builder to create new signature to detect this activity in future.
5. Select "Sysmon Event Longs", then select "Registry Modifications". Paste the registry key and the registry name with value 1 and the ATT&CK ID (TA005).
6. Effectively block the sample4.exe to get the flag for this challenge.

### Analysing outgoing_connections.log
1. Examine the *outgoing_connections.log* reports. Notice that the log reports from 10.10.15.12 to various endpoints, including a lot of what seems to be the same traffic to 51.102.10.19. I say that it seems to be the same traffic based on the size of the packets: each is 97 bytes.
2. Examining the timestamps of this traffic, we find that this traffic occurs every 30 minutes exactly.
3. This looks like it’s beaconing to Sphinx’s command and control infrastructure.
4. Navigate to the Sigma Rule Builder, select "Sysmon Event Logs", then "Network Connections".
5. Detect connections for remote IP *Any* since Sphinx is known hop to different IP Addresses. Same goes to the remote port *Any*. Include the ATT&CK ID *Command and Control(TA0011)*. Validate then the flag will be given.

### Analysing commands.log
1. Examine the *commands.log* file, found out it describes several command that discover and output various information about the host and network connections to a hard-coded filename as *%temp%\exfiltr8.log.*
2. This behaviour corresponds to the MITRE ATT&CK tactic Discovery (TA0007).
3. Navigate to the Sigma Rule Builder, select *Sysmon Event Logs*, then *File Creation and Modification*
4. Put the file path as *%temp%*, filename as *exfiltr8.log* and ATT&CK ID  is Discovery TA0007. Validate then the flag will be given.
