## Task 2: What is Logged
### Logging Overview
* **Incident Response:** Logs can show when and how the attack occurred
* **Threat Hunting:** Logs allow you to search for signs of malicious activity
* **Alerting and Triage:** Logs are a building block of any alert or detection rule

### Anatomy of a Log Entry
Windows is an interesting OS as it has very powerful logging capabilities but requires a lot of knowledge to read and understand the logs. as they are stored in a binary format inside the *C:\Windows\System32\winevt\Logs folder*:
<img width="1683" height="424" alt="image" src="https://github.com/user-attachments/assets/6f78b7d1-e752-41d2-ae61-8263e9b30bae" />
Every EVTX file corresponds to a specific log category. For example, Application Logs contain events logged by user-mode applications like the IIS web server or MS SQL database, and Security Logs capture events like logon attempts, process activity, and user management.

### Reading Event Logs
We will use **Event Viewer** for this room, a built-in tool that allows you to view and manage event logs. To open Event Viewer, search for "Event Viewer" using Windows Search or press Win + R, type eventvwr, and press Enter. Once the tool is loaded, you may see all system logs parsed, grouped, and ready for analysis:

1. **Log Sources:** Every EVTX file corresponds to a single item on the left panel
2. **Log List:** Each row you see is a single event that contains a few properties you can sort by:
  * **Keywords:** For some events, indicates if the action was successful or not
  * **Date and Time:** The timestamp when the event occurred (system time, not UTC!)
  * **Event ID:** A unique number for the event name (e.g. a failed login is always 4625)
3. **Log Details:** The actual content of the log, in a plaintext or XML format ("Details" tab)
**Filters Menu:** Use the "Filter Current Log" and "Find" buttons to filter the logs
<img width="1280" height="398" alt="image" src="https://github.com/user-attachments/assets/8aaf7bc4-6ef9-4c11-abfc-b20f0fb79adc" />


## Task 3: Security Log: Authentication
<img width="868" height="333" alt="image" src="https://github.com/user-attachments/assets/f801bded-7008-4b63-844d-eb05d9284ec9" />

### Structure of 4624
A typical Windows server can generate tens of login events per minute, and every login event is often comprised of many different fields. Still, you can cover most L1/L2 cases just by checking a few core event fields in the image below. You can also read more about other fields and logon types in the Event ID Encyclopedia
<img width="1194" height="470" alt="image" src="https://github.com/user-attachments/assets/9f51490a-0ea3-4af3-b00a-ec0ed1ada020" />

### Usage of 4624/4625
**Detect RDP Brute Force**
1. Open Security logs and filter for 4625 event ID (Failed login attempts)
2. Look for events with Logon Type 3 and 10 (Network and RDP logins)
  * For most modern systems, the logon type will be 3 (since NLA(opens in new tab) is enabled by default)
  * For older or misconfigured systems, the logon type will be 10 (since NLA is not used)

**Analyse RDP Logons**
1. Open Security logs and filter for 4624 event ID (Successful logins)
2. Look for events with Logon Type 10 (RDP logins)
  * If NLA(opens in new tab) is enabled, every RDP logon event is preceded by another 4624 with logon type 3
  * To get a real Workstation Name, you need to check the preceding logon type 3 event
3. Your red flags are either a preceding brute force or a suspicious source IP / hostname
4. If you assume that the login was indeed malicious, find out what happened next:
  * Windows assigns a Logon ID to every successful login (e.g. 0x5D6AC)
  * Logon ID is a unique session identifier. Save it for future analysis!

**Q1: Open the "Practice-Security.evtx" file on the VM's Desktop.
Which IP performed a brute force of the THM-PC?**
Filter Event ID 4625 which indicate Logon Failure, give us clue that it is typical brue force.

<img width="489" height="475" alt="image" src="https://github.com/user-attachments/assets/0a7b0e7f-574b-4666-8d62-abeb8cc892ac" />

**Q2: Which user has been breached as a result of the attack?**

<img width="595" height="442" alt="image" src="https://github.com/user-attachments/assets/34f22401-4e5b-43cc-aa04-866560c429cd" />

**Q3: What was the Logon ID of the malicious RDP login? Note: The login you are looking for has a Logon Type 10.**

<img width="387" height="287" alt="image" src="https://github.com/user-attachments/assets/f532ab80-69fb-4a5f-ab73-97b9dbec8c80" />


## Task 4: Security Log: User Management

<img width="866" height="472" alt="image" src="https://github.com/user-attachments/assets/78b9429e-a99e-448f-8efb-750bfd35dc5d" />

### Structure of User Management Events
All user management events have a similar structure and can be split into three parts: who did the action (Subject), who was the target (Object), and which exact changes were made (Details):
1. **Subject:** The account doing the action. Note the Logon ID field - you can use it to correlate this event with the preceding 4624 login event!
2. **Object:** This can be named differently depending on an event ID (e.g. New Account or Member), but it always means the same - the target of the action.
3. **Details:** A target group for 4732 and 4733 events, or new user's attributes like full name or password expiration settings for the 4720 event.

<img width="1313" height="630" alt="image" src="https://github.com/user-attachments/assets/1fe66b28-6198-4634-83fb-359b1db530d8" />

### Usage of User Management Events
**Hunt for Backdoor Users**
1. Open Security logs and filter for 4720 / 4732 event IDs
2. Manually review every event; your red flags are:
  * No one from your IT department can confirm the action
  * Changes were made during non-working hours or on weekends
  * The subject user's name is unknown or unexpected to you
(e.g. "adm.old.2008" creating new Windows users)
  * The target user's name does not follow a usual naming pattern
(e.g. "backup" instead of "thm_svc_backup")
3. If you confirmed that the action was malicious, find out the login details:
  * Copy the Logon ID field from your 4720 / 4732 event
  * Find the corresponding login event with the same Logon ID
  * Refer to the workbooks from the previous task for further analysis

**Q1: Continue with the "Practice-Security.evtx" file on the VM's Desktop.
Which user was created by the attacker soon after the RDP login?**

<img width="535" height="300" alt="image" src="https://github.com/user-attachments/assets/58a669fe-54fe-41d7-a680-f0c361a7c9c5" />

**Q2: Which two privileged groups was the backdoor user added to?
(Answer in alphabetical order, e.g. "Administrators, Power Users")**

<img width="493" height="306" alt="image" src="https://github.com/user-attachments/assets/fea279fe-db1d-4ab6-8d7d-bc0b1261e55e" /> <img width="530" height="314" alt="image" src="https://github.com/user-attachments/assets/48d82a65-c07b-48a0-a39d-04319c313309" />


## Task 5: Sysmon: Process Monitoring
<img width="868" height="292" alt="image" src="https://github.com/user-attachments/assets/e2e712d6-d129-4baa-9367-01b31d81f43c" />

### Sysmon vs Security Log
Sysmon is a free tool from the Microsoft Sysinternals suite that became a de facto standard for advanced monitoring in addition to the default system logs. \

So, if I were to choose between enabling the basic, noisy 4688 event ID or spending some time installing Sysmon to receive more powerful and flexible logs, I would proceed with Sysmon, and you are encouraged to do the same! Once installed, Sysmon logs are found in Event Viewer under Applications & Services -> Microsoft -> Windows -> Sysmon -> Operational.
<img width="1400" height="498" alt="image" src="https://github.com/user-attachments/assets/50bab8f5-90cb-41e8-8fba-6aa571df5a4a" />


### Sysmon Event ID 1 in Action
As you can see on the screenshot above, event ID 1 has a lot of different fields, the most important of which can be grouped as:

* **Process Info:** Context of the launched process, including its PID, path (image), and command line
* **Parent Info:** Context of the parent process, very useful to build a process tree or an attack chain
* **Binary Info:** Process hash, signature, and PE metadata. You will need it for more advanced rooms
* **User Context:** A user running the process and, most importantly, Logon ID - same as in the Security logs \

Since almost any attack works on the endpoint level and requires at least some process to be launched to breach the system or exfiltrate the data from it, process monitoring is the most important log source for any SOC team. Use the following workbook to perform a basic analysis of any process launch: \

**Analyze Process Launch**
1. Open Sysmon logs and filter for event ID 1
2. Review the fields from the process and binary info groups. The red flags are:
  * Image is in an uncommon directory like C:\Temp or C:\Users\Public
  * Process is suspiciously named like aa.exe or jqyvpqldou.exe
  * Process hash (MD5 or SHA256) matches as malware on VirusTotal(opens in new tab)
3. Review the fields from the parent process group. The red flags are:
  * Parent matches red flags from step 2 (suspicious name, path, or hash)
  * Parent is not expected (e.g. Notepad launching some CMD commands)
4. If still in doubt, go up the process tree until you are confident in your verdict:
  * Find the preceding event where ProcessId equals ParentProcessId in your event
  * Analyze it by following steps 2 and 3 (suspicious parent, name, path, or hash)
5. Finally, trace the attack chain by filtering all Security and Sysmon events with the same Logon ID

**Q1:Open the "Practice-Sysmon.evtx" file on the VM's Desktop.
Which web browser does Sarah use to browse the web?**
<img width="754" height="331" alt="image" src="https://github.com/user-attachments/assets/44b78b36-22b9-4845-875a-72461ac84aa7" />

**Q2:Which file did Sarah download from the browser?**
<img width="768" height="337" alt="image" src="https://github.com/user-attachments/assets/c536cfe6-4199-40a8-8796-abf8c8c0e52e" />

**Q3:Which URL was the file downloaded from?
Note: Use other Sysmon events to find out!**
<img width="767" height="510" alt="image" src="https://github.com/user-attachments/assets/3f1f306d-5a0a-40da-a1e8-2cf0e316ede2" />

## Task 6: Sysmon:Files and Network
<img width="867" height="274" alt="image" src="https://github.com/user-attachments/assets/17e63654-3954-44bf-aedf-3aca60497750" />

### Structure of Sysmon Events
<img width="1430" height="320" alt="image" src="https://github.com/user-attachments/assets/436ab9f5-7f3b-4968-a8ca-2dfd7232d377" />

### Usage of Sysmon Events
Although process creation events provide enough context to detect common breach scenarios, additional logs can be vital to reconstruct the full attack chain and ensure nothing is missed. For example, you need network logs to identify where the data was exfiltrated to, and registry change logs to check which system configuration was modified by threat actors. \

**Analyze Process Activities**
1. Copy the ProcessId field from the event ID 1
2. Search for other Sysmon events with the same ProcessId
3. Your red flags for network connection events are:
  * Connection to external IPs on port 80 or on non-standard ports like 4444
  * Connection to known malicious IPs (e.g. by checking on VirusTotal(opens in new tab))
  * DNS queries to suspicious domains (*.top, *.click, or hpdaykfpadvsl.com)
4. Your red flags for file and registry changes are:
  * Files dropped to staging directories like C:\Temp or C:\Users\Public
  * Dropped file is a script (.bat or .ps1) or an executable file (.exe or .com)
  * Created files or registry keys are used for persistence (soon on it later!)

**Q1:Continue with the "Practice-Sysmon.evtx" file on the VM's Desktop.
Which file was created by the downloaded malware to persist on the host?**
<img width="767" height="475" alt="image" src="https://github.com/user-attachments/assets/0d36f250-f7e0-4008-a627-8dbe68065f89" />

**Q2:What is the Command & Control server malware connected to?
(Answer in format IP:Port, e.g. 1.1.1.1:80)**
<img width="769" height="673" alt="image" src="https://github.com/user-attachments/assets/e7cab784-ede6-4eff-96ab-e8a4f8af977f" />

**Q3:Finally, which domain does the malicious IP correspond to?**
<img width="682" height="471" alt="image" src="https://github.com/user-attachments/assets/9261a750-ecb5-45ab-a9e5-d78e55d1d2e6" />


## Task 7: PowerShell: Logging Commands
PowerShell is a powerful tool built into Windows that attackers love to abuse. Mainly because it is both trusted and capable of malware download, system discovery, data exfiltration, and even advanced techniques like process injection. However, you won't capture its commands by just using process creation logs like the Sysmon event ID 1. Take a look at the command prompt below:
<img width="864" height="167" alt="image" src="https://github.com/user-attachments/assets/96a24e4f-b133-4e05-864f-51e8eccd68ec" />

Here, the threat actor managed to read a sensitive file, view local users and groups, and even download malware to the Temp directory. Still, you will see a single event ID 1 stating that powershell.exe was launched, with no information about the executed commands.


### How it Works
Every program has a specific purpose: firefox.exe is a web browser, notepad.exe is a text editor, and whoami.exe simply outputs your username. If you're just browsing the web, you might only create a single Firefox process. However, with every out-of-scope task like RDP access or photo editing, you will have to open new programs and create additional logs. \

PowerShell, on the other hand, is a powerful all-in-one tool for managing the system. Once you launch powershell.exe, you can run hundreds of different commands within the same terminal session without creating new processes for each action. This is why Sysmon is not very helpful here, and you'll need to find an alternative logging approach. \

### PowerShell History File
There are at least five methods to monitor PowerShell, each with its own pros and cons. While you can check out the Logless Hunt room and research AMSI and Transcript Logging topics, in this room, we will focus on a simple but effective way to track PowerShell commands - the PowerShell history file:
* *C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt*
The PowerShell history file is a plain text file automatically created by PowerShell. It simply records every command you type into a PowerShell window and is immediately updated when you press Enter to submit a command:
<img width="864" height="281" alt="image" src="https://github.com/user-attachments/assets/a8123aa5-4119-471e-82ef-134b217f5ab7" />

**Q1: Review the Administrator's PS history on the attached VM.
Which PowerShell command was executed first?**
<img width="196" height="187" alt="image" src="https://github.com/user-attachments/assets/862bdef7-e3f6-4afb-9f28-3320a258b612" />

**Q2: When did the Administrator run the first PS command? (Format: April 18, 2025)
Note: You might need to right-click the history file and open "Properties" to get the answer!**
<img width="363" height="510" alt="image" src="https://github.com/user-attachments/assets/7e51c6ca-8e27-4e32-979a-0c6a66c97624" />

**Q3: Can you find the flag stored in the PowerShell history? (Format: THM{...})
Note: You might want to check the PS history of other local users!**
<img width="311" height="243" alt="image" src="https://github.com/user-attachments/assets/92c55537-881d-4569-8d3d-2d3a04979808" />
