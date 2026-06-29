## Task 2: Discovery Overview
### Situational Awareness
After the criminals pass through the front door, do they know what's behind the door? Most do not, so they will start searching the rooms: maybe there is a hidden treasure, but maybe a trap ready for action.

### Discovery Commands
<img width="1239" height="476" alt="image" src="https://github.com/user-attachments/assets/c69fa309-1d35-48cf-813c-62007c5297a0" />


### Discovery Process
Recall the phishing attack chain from the previous room. After the attachment is launched, it runs basic discovery commands to identify the victim or even delete itself if a specific antivirus is installed or a victim is not from a targeted company or country. Then, it connects back to the threat actor, giving full control over the victim. From there, human attackers can type additional Discovery commands if required.
**How Attackers Control the Victim**
<img width="1507" height="485" alt="image" src="https://github.com/user-attachments/assets/e4f0db8a-f24d-4721-bd57-26b4afc1a77b" />

### Lab
**Q1: Open CMD and type "net user Administrator". Which privileged group does the user belong to?**
<img width="474" height="422" alt="image" src="https://github.com/user-attachments/assets/1560d630-4462-4da5-8024-f39938a49d7b" />

**Q2: Open Event Viewer and try to find your command in Sysmon logs. What is the "Image" field of the net command you just run?**
<img width="647" height="377" alt="image" src="https://github.com/user-attachments/assets/3762e7ec-8364-478f-9bae-8857bd6ae23b" />



## Task 3: Detecting Discovery
### Discovery via CMD
Discovery via the command line is the most common and easiest method available for threat actors. This is because it simply uses the existing commands like "whoami" or "ipconfig" that are available on all Windows machines by default; check out this articlefor a real-world attack example. Luckily for the defenders, most of the launched commands are logged as new processes, like on the process tree below:
<img width="1236" height="331" alt="image" src="https://github.com/user-attachments/assets/eea158ed-3562-4800-a87a-f8cf5df67777" />

### Discovery via GUI
In cases where threat actors log in to the system interactively, like after the RDP breach, they are not limited to console commands (but they often use them anyway as a habit). With access to the graphical interface, nothing prevents attackers from using the same toolkit as you do: Apps & Programs, System Settings, Disk Management, or even Event Viewer. In this scenario, you won't see typical "whoami" commands but rather a process tree that looks like this:
<img width="1236" height="263" alt="image" src="https://github.com/user-attachments/assets/4c1b1066-f682-4f38-8a65-79c767ee0ab5" />

### Detecting Discovery
The first task to detect a potential Discovery is to find a Discovery command, or better, a sequence of commands run during a short period of time. You will see them as process creation events tracked by Sysmon event ID 1 or as new rows in the PowerShell history file. There are lots of Discovery commands, so be prepared to use the search engine if you are not sure what the command means.
Next, it is important to find out where the commands are coming from. Commands like "ipconfig" are often used by IT departments and legitimate tools, and you don't want to create panic just because your coworker checked their IP. For this room, you can build the process tree using Sysmon logs: filter for process creation events and correlate ProcessId and ParentProcessId fields, like in the example below:
<img width="1211" height="285" alt="image" src="https://github.com/user-attachments/assets/0729dabd-f495-486e-9b14-9913834db4ae" />

### Lab 
**Q1: Looking at Sysmon logs, what is the first command the invoice.pdf.exe executes?**
<img width="643" height="394" alt="image" src="https://github.com/user-attachments/assets/042bbb3b-3861-4f34-8d43-2d1ad468cd0e" />

**Q2: Which command did the malware use to check the presence of MS Defender EDR?**
<img width="666" height="325" alt="image" src="https://github.com/user-attachments/assets/7485a971-0686-4c63-bdf8-3c9b3be6a8f0" />

**Q3: To which domain did the malware send the discovered data?**
<img width="675" height="311" alt="image" src="https://github.com/user-attachments/assets/7930ebb8-a995-42d0-87e8-78d0a53442ea" />


## Task 4: Collection Overview
### Searching Secrets
Continuing our scenario, what would the criminals do after they explored all rooms in the apartment, found out who the owner is, what valuables they hide, and what traps are in place? They grab the actual treasure - something that can be sold or is valuable for the threat actors. This process involves three more MITRE tactics: Collection, Credential Access, and Exfiltration (To make it simpler, let's treat Credential Access as a part of Collection).

### Collection Targets
The targets drastically differ depending on the attackers' goals. Some adversaries hunt personal info like images or chat conversations; others look for crypto wallets, gaming, or banking accounts; and advanced groups just use the victim to access the corporate network, hoping for a full-scale ransomware encryption.
Note that while most of the sensitive data is stored as simple files, the secrets can also be hidden in the registry or in process memory. You can review the common Collection targets in the code block below:
<img width="1222" height="291" alt="image" src="https://github.com/user-attachments/assets/e352dbf5-5de0-4539-a425-c7a2215ee4a2" />

### Exfiltrating Data
Data collection can be performed automatically via scripts or manually by human threat actors. For scripts, the whole process usually takes less than a minute, but it may take hours for the attacker to find and review the interesting files. Still, both methods should eventually end with exfiltration - uploading stolen data to a controller server. Here, threat actors can be very creative - to avoid detection, they often:

* Exfiltrate stolen data to DropBox, Mega, Amazon S3, or other trusted cloud storage services
* Exfiltrate stolen data to known code repositories like GitHub or messengers like Telegram
* Or just create a trustworthy-looking domain like "windows-updates.com" and send the data there

### Lab
**Q1: What is the Facebook password that the user saved in Chrome?
(Chrome menu > Passwords and autofill > Password Manager)**
<img width="720" height="368" alt="image" src="https://github.com/user-attachments/assets/0f0bdcf1-bca0-4846-a370-98abd9b61b12" />

**Q2: Which interesting SSH key does the user store on disk?
(Start your search from C:\Users\Administrator\)**
<img width="765" height="170" alt="image" src="https://github.com/user-attachments/assets/859f55ec-a926-451e-8c3f-b4b8821abc93" />

**Q3: What is the secret PDF file explaining TryHackMe's internal network?
(Look for the file on the Desktop, Downloads, and Documents)**
<img width="690" height="157" alt="image" src="https://github.com/user-attachments/assets/75a1434f-7567-426b-8057-dedc1d658abf" />


## Task 5: Detecting Collection
### Detecting Collection
Same as with Discovery, threat actors can use both command-line and graphical interface options to review sensitive files. However, in Collection, threat actors don't just check a system configuration but rather look for specific files and folders shown in the previous task. Thus, you can detect access to the files by tracking commands like:
<img width="1240" height="453" alt="image" src="https://github.com/user-attachments/assets/abc6a8b0-ac80-43ec-a54f-32dd2138916f" />

### Collection Examples
During manual collection or when using scripts, you will see basic commands and processes covered in the previous task. In this incident(opens in new tab), threat actors simply used Notepad and Wordpad to open files of interest and then used 7-Zip to archive all files at once. As you may see from the screenshot, the actions were easily detected with Sysmon event ID 1:
<img width="1450" height="125" alt="image" src="https://github.com/user-attachments/assets/42ff4fc4-f71d-4b66-b23c-157656102398" />

**Data Stealers**
Collection performed by human threat actors is typical for breaches of big networks, where the attacker knows their target and spends much time looking for data to steal. However, attacks targeting simple personal workstations rarely involve human attacker and data collection is performed by a data stealer - specialized malware to automate collection and exfiltration.

For example, Gremlin data stealer, a single malicious file, steals VPN profiles, cryptocurrency wallets, web browser sessions, Steam, Discord, and Telegram data, and even takes screenshots of the victim's host. You can read the details in this Unit42 blog post(opens in new tab). Note that data stealers rarely use CMD or PowerShell commands but rely on their own code, making it harder to understand which exact data was accessed or stolen:
<img width="1536" height="278" alt="image" src="https://github.com/user-attachments/assets/31fe86bc-9e91-41fb-8fb8-8615cb7ea6df" />

### Lab
**Q1: Looking at Sysmon logs, what directory does the stealer create?**
<img width="649" height="322" alt="image" src="https://github.com/user-attachments/assets/63c52613-886e-47c2-a150-e3f5a38f1b6b" />


**Q2: Which three file extensions does the malware search for? Format: Separate by comma in alphabetic order (e.g. bat, txt)**
<img width="590" height="187" alt="image" src="https://github.com/user-attachments/assets/455eaedc-0cac-4ae6-b5fe-17ec598aaa5b" />

**Q3: Which PowerShell cmdlet does the malware use to get clipboard content?**
<img width="671" height="311" alt="image" src="https://github.com/user-attachments/assets/d71bf2a2-8f6d-47a1-b79c-53dcdad389dc" />

**Q4: Which domain does the malware exfiltrate the data to?**
<img width="653" height="266" alt="image" src="https://github.com/user-attachments/assets/311958bc-d06c-4031-a1ed-6fe4738b4358" />


## Task 6: Ingress Tool Transfer
### Ingress Tool Transfer
Recall the previous room explaining how attacks start: not from a fully functional malware, but from a tiny phishing attachment or from an RDP session without any red team tools. Thus, at some attack stages, threat actors may need to download more tools to achieve their goals, for example:

* A script to automate Discovery and find common vulnerabilities like Seatbelt(opens in new tab)
* A tool to extract saved passwords or OS credentials like Mimikatz(opens in new tab)
* A fully functional Remote Access Trojan (RAT) like Remcos RAT(opens in new tab)
* Finally, a ransomware binary to encrypt the system after the data is stolen
The process of downloading additional malware to the breached system is mapped to the MITRE Ingress Tool Transfer(opens in new tab) technique, and it is used in the majority of breaches. You have already seen an example where a LNK attachment used PowerShell to download additional malware, but there are many other ways to transfer malware even without PowerShell!

### Common Transfer Methods
Why can't the threat actors just include all they need into a single phishing attachment, you may ask. There can be different reasons, but the common ones are to bypass antivirus by splitting the malware into multiple parts and to minimize exposure of their tools/exploits in case they're caught right in the beginning.
<img width="1233" height="324" alt="image" src="https://github.com/user-attachments/assets/18eb8063-f36b-4613-92cc-22e03de23175" />

### Detecting Tool Transfer
Since a transfer requires a network connection, your best option would be to track a network connection or a DNS request from the suspicious process. Note, however, that threat actors often try to avoid detection by downloading the tools from legitimate services like GitHub, so make sure to analyze which process is making the connection, the destination domain, and the file being downloaded. The screenshot below shows a complete event chain
<img width="1046" height="474" alt="image" src="https://github.com/user-attachments/assets/3275c748-a05f-41e1-ae64-978df9b8248a" />

### Lab
**Q1: Open the Chrome browser on the VM and navigate to the URL.
What is the flag in the response?**
<img width="533" height="167" alt="image" src="https://github.com/user-attachments/assets/7a37216f-d7ef-4b4c-8878-ca8349cd29c0" />

**Q2: Next, open CMD and download the file from the same URL using curl.exe.
What is the flag in the response?**
<img width="746" height="152" alt="image" src="https://github.com/user-attachments/assets/059d4fdf-6396-408b-ad4c-8535ccb8211c" />


**Q3: Continue with the same CMD and URL, but now using certutil.exe.
What is the flag in the response**
<img width="765" height="195" alt="image" src="https://github.com/user-attachments/assets/9ab88036-acc7-43be-acdc-3abe9ce5d0c4" />


**Q4: Finally, download the same file using PowerShell IWR.
What is the flag in the response?**
<img width="767" height="71" alt="image" src="https://github.com/user-attachments/assets/e0543070-85d2-4c26-9c04-70d09ac66bce" />


