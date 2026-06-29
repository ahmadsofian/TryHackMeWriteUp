## Task 2: Command and Control
### Attacks Without C2
In some cases, C2 is not needed at all. For example, threat actors can type their commands directly in the RDP session after an RDP breach. Since this method becomes unavailable as soon as RDP is closed or secured, most threat actors choose to still set up a C2 immediately after the breach.

### Simplest C2
For other Initial Access methods, threat actors can't simply use RDP every time they need to run a command, so they need some process that connects back to the attackers and waits for their commands 24/7. In the simplest case, the phishing attachment will be that process and establish the Command and Control channel, like on the CobaltStrike C2 screenshot below.
In more advanced cases, the attachment won't immediately connect back, but rather download an additional C2 malware, hide it in a folder like C:\Temp, and run it as a new stealthy process. This method is beneficial to keep the attack going if the victim decides to delete the original attachment. See how it worked in the recent ransomware cases and during the APT29 phishing campaign.

### Lab
**Q1: Which suspicious archive did the user download?**
<img width="1022" height="326" alt="image" src="https://github.com/user-attachments/assets/1fe349ee-690d-43c8-9ce6-835ba2a6d8c1" />

**Q2: Where did the attackers hide the C2 malware file?**
<img width="1021" height="262" alt="image" src="https://github.com/user-attachments/assets/02d148ba-c93a-4e3c-b8a0-5389177de373" />

**Q3: What is the domain of the Command and Control server?**
<img width="767" height="169" alt="image" src="https://github.com/user-attachments/assets/81f80cb8-d284-4a61-8109-e04b9e00a75e" />


## Task 3: Persistence Overview
Data stealer infections usually have a very short lifespan: they breach the victim, collect the data, exfiltrate it, and exit - all within minutes. However, for most other attacks, maintaining access to the victim for days or even months after the Initial Access is vital. The tactic of maintaining reliable, long-term access to the target that can survive reboots and password changes is called Persistence - a big and interesting topic that you will discover soon.

### Persisting via RDP
Many Windows breaches happen because of the exposed service: RDP with a weak password, a vulnerable mail server, or a misconfigured web app. For such scenarios, the threat actors can access the machine via the same exposed service over and over again until the vulnerability is fixed. Still, threat actors often deploy an additional Persistence method, for example:
* Create an additional hidden vulnerability in the breached service (e.g. a backdoor or a web shell)
* Create a new user (T1136), make it an administrator (T1098), and use it for further RDP logins
Let's focus on the second method now and see how you or threat actors can manage users on Windows. The first option is to use the graphical utility by searching for "Computer Management" or by launching lusrmgr.msc. The second option is to use a command line, like in the example below:
<img width="1233" height="236" alt="image" src="https://github.com/user-attachments/assets/f6ccf392-403f-49fe-98bb-50d6d950f79d" />

### Detecting Backdoored Users
It's time to go back to the Security event logs! Every user creation event is logged as Security event ID 4720, which you explored in the Windows Logging for SOC room. Since threat actors can be very creative with naming the backdoored accounts, you should not rely just on detecting suspicious names like "hacker" but rather investigate:
1. Who created the account? Can the person confirm the account creation?
2. What is the source IP and time of the creator's login? Is it expected?
3. Which other suspicious events can you see in the creator's session?

**Making Users Privileged**
Next, a new user by itself won't give the attacker much, as the default user permissions do not allow remote (RDP) logins or grant administrative privileges on the machine. To overcome this, threat actors will add their backdoored account to one of the privileged groups, which is tracked by Security event ID 4732. The most commonly exploited groups are Administrators and Remote Desktop Users.

**Resetting Passwords**
Lastly, in more advanced cases, threat actors may simply reset the password of some old or unused account and use it instead of creating a new one. You can detect it with Security event ID 4724. In summary, below you can see how the described event IDs look like:

<img width="1313" height="630" alt="image" src="https://github.com/user-attachments/assets/115c55cc-8d36-4571-be44-c767c137880e" />

### Lab
**Q1: How many times did the threat actor fail to log in to the Administrator?**
<img width="765" height="109" alt="image" src="https://github.com/user-attachments/assets/6d5b8ab5-51f7-4242-980c-ea2de61a4ebf" />

**Q2: After the successful login, which backdoor user did the attacker create?**
<img width="766" height="280" alt="image" src="https://github.com/user-attachments/assets/568306cf-e608-4781-8945-cd25df02cbcc" />

**Q3: Which privileged group was the backdoor user added to?**
<img width="768" height="351" alt="image" src="https://github.com/user-attachments/assets/80aa6b2d-9de2-4e53-965c-380124fba6c7" />


## Task 4: Persistence: Tasks and Services
### Malware Persistence
Persistence via a backdoored user works well if you can remotely log in to it via RDP, but if the attack started through a phishing attack or USB infection, that's not an option. For these scenarios, threat actors need an actively running malware that maintains a connection with their C2 server even after a system reboot. How could they achieve malware persistence?

### Services and Tasks
Unfortunately for defenders, there are literally a hundred or more methods to persist on a Windows machine. As a SOC L1 or L2 analyst, you don't need to know all of them, but let's start with the two common ones:
<img width="1232" height="256" alt="image" src="https://github.com/user-attachments/assets/f01517a6-8b23-48f0-9665-6d3d946511c2" />

### Detecting Services
Many critical Windows components like DNS client or Security Center are services. You can view services by launching services.msc or searching for "Services", but you need administrative privileges and the sc.exe command to create or modify one.
Threat actors can create their own malicious services that will run the specified program on startup, and they do it very often, as you can read in the MITRE examples. In logs, you can detect malicious services in three ways:
1. Detect the launch of the sc.exe create command via Sysmon event ID 1
2. Detect service creation via Security event ID 4697 or System event ID 7045
3. Detect suspicious processes with a services.exe parent process
<img width="1175" height="370" alt="image" src="https://github.com/user-attachments/assets/07907139-d573-4695-8356-36f311cd9836" />

### Detecting Tasks
Scheduled tasks are another Windows feature heavily used by both the OS and external apps (e.g. to check for updates or make a backup every hour). From GUI, you can manage tasks by launching taskschd.msc or searching for "Task Scheduler". From the command line, you can use the schtasks.exe command.
Unlike services, scheduled tasks are very easy to configure and hide, which is why they are the most common persistence method by threat actors, like in these APT28 and APT41 attacks. Similar to services, you can detect scheduled tasks in three ways:
1. Detect the launch of the schtasks.exe /create command via Sysmon event ID 1
2. Detect and analyze scheduled task creation events via Security event ID 4698
3. Detect suspicious processes with a svchost.exe [...] -s Schedule parent

<img width="1175" height="370" alt="image" src="https://github.com/user-attachments/assets/da84757e-2fc3-4d62-90db-1125a92b1f5e" />

### Lab
**Q1: Which Windows service was created to persist the Nessie malware?**
<img width="767" height="277" alt="image" src="https://github.com/user-attachments/assets/055e257d-b2be-4a39-ba55-500ff0d0e416" />

**Q2: Which scheduled task was created to persist the Troy malware?**
<img width="766" height="330" alt="image" src="https://github.com/user-attachments/assets/529f1e02-86e2-4076-a3ca-65d13d720e33" />

**Q3: What flag do you get after finding and running the Troy malware?**
<img width="684" height="349" alt="image" src="https://github.com/user-attachments/assets/bc09caab-93c6-432f-b957-b422d3a56a3a" />


##  Task 5: Persistence: Run Keys and Startup
### Run Keys and Startup
Services and scheduled tasks are typically run on system boot and require administrative privileges to configure. However, what if a program has to run only when a specific user logs in? For such cases, Windows provides a few per-user persistence methods that are actively used by both legitimate tools and malware:
<img width="1235" height="230" alt="image" src="https://github.com/user-attachments/assets/5cc45aa4-ee11-476c-9c3a-060bc8aab307" />


### Detecting Startup
The startup folder was meant to be an easy way for inexperienced users to configure programs to run on login. You simply open the startup folder, move your program or program shortcut there, and see how it automatically starts upon your future logins. You can access your startup folder via the path below:
* *C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\**
* *Or for all users: C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp*
<img width="1175" height="370" alt="image" src="https://github.com/user-attachments/assets/3c659a6c-2413-41be-a1e2-9ad7947470d9" />

### Detecting Run Keys
Run key persistence is very similar to the startup folder; they even share a single MITRE technique(opens in new tab)! The only major difference is how the entries are added there. Instead of just copying the program to the startup folder, you need to create a new value in the "Run" Windows registry and put the path to your program there:
* *HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run*
* *Or for all users: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run*

To view the "Run" entries, you can launch the regedit.exe or search for "Registry Editor" and go to the path shown above. To detect the malicious entry from logs, you can monitor registry change events (Sysmon Event ID 13) affecting the Run keys:
<img width="1175" height="370" alt="image" src="https://github.com/user-attachments/assets/e3a02324-00c5-48da-9def-f1caefce5075" />

### Lab
**Q1: What is the parent process image of the "Odin" malware?**
* c:\windows\explorer.exe

**Q2: What is the last line that the "Odin" malware outputs?**
<img width="765" height="249" alt="image" src="https://github.com/user-attachments/assets/61239de6-4dc8-4b3b-958a-b4aa6fe22f93" />

**Q3: What flag do you get after finding and running the "Kitten" malware?**
<img width="361" height="155" alt="image" src="https://github.com/user-attachments/assets/f6815a32-7f86-4400-b402-73004efa9a3d" />


## Task 6: Impact and Threat Detection Recap
### Need for Persistence
In the previous tasks, we have learned how threat actors can remain active on the systems. But why would they need it? Why not just steal the data and exit the system before detection? There can be multiple reasons, but the main ones are:
1. Add the host to a botnet and use it for further attacks
  * Like how the Kraken Botnet combines crypto miner, data stealer, and C2 capabilities
2. Spy on the victim as a part of a state-sponsored campaign
  * Like how Volt Typhoon stayed undetected in the US electric grid for nearly a year
3. Use the victim as an entry point to the network, breaching which could take months
  * Like in the case where threat actors spent 29 days breaching a full network

### Active Directory and Ransomware
Let's take a closer look at the third point. In most cases, a Windows network means a large Active Directory that brings its own attacks, detections, and threats - the main one being ransomware. Ransomware scares businesses the most. Why? Because it can bring entire companies to a halt, as it did for McLaren hospitals, affecting 743,000 patients(opens in new tab). Just imagine seeing your servers encrypted, data stolen, and ransom notes automatically printed on all office printers:

### Threat Detection Recap
Active Directory and ransomware are complex topics, but all complex attacks start from a simple single breach. In the Windows Threat Detection rooms, you explored how breaches begin, how the attackers steal data, and how they remain undetected for years. You are now ready to use the acquired knowledge to detect and stop the attacks before ransomware causes a disastrous Impact(opens in new tab), preferably right after Initial Access. Here is a quick recap of what you've learned so far (highlighted in yellow):






