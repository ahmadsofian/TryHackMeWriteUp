## Task 2: Intro to Initial Access
### Initial Access
No matter what the final goal is, the first step of a threat actor is to get through the front door, and the moment an attacker successfully gets in is known as Initial Access.

### Exposed Services

<img width="970" height="285" alt="image" src="https://github.com/user-attachments/assets/d8f916a0-b413-4121-9333-ad88522f5744" />

Putting a Windows server directly on the Internet is a common task for IT teams - corporate websites require an open HTTP port to show content, a mail server can't handle emails without an active SMTP port, and IT admins need RDP to manage the machine remotely. However, every exposed service introduces major security risks. Within minutes, your exposed system can be scanned by automated bots looking for open ports, weak passwords, or unpatched vulnerabilities. And if something is not protected enough, threat actors will use their chance, as proven by these MITRE techniques:

* **T1133 / External Remote Services:** Threat actors will look for exposed RDP/VNC/SSH with weak passwords to get remote access to the machine
* **T1190 / Exploit Public-Facing Application:** Threat actors will look for misconfigured or vulnerable websites and applications


### User-Driven Methods
<img width="970" height="285" alt="image" src="https://github.com/user-attachments/assets/828a1b7c-24ed-432b-a29a-95952a7707a5" />

But how can the laptop be infected if it is not Internet-exposed? Indeed, unless you help the threat actors yourself, it is very hard to infect your laptop. However, people often help threat actors by clicking on malicious links, launching phishing attachments, using pirated software, picking up unknown USB devices, and so on. And since humans are still the weakest link in cyber security and Windows is the most popular OS for user workstations, you are very likely to handle user-driven Windows Initial Access alerts frequently. The methods are covered by these MITRE techniques:

* **T1566 / Phishing:** Threat actors employ different phishing techniques, tricking users into launching the malware themselves
* **T1091 / Removable Media:** Threat actors infect a USB device and hope that users will use the USB on multiple PCs



## Task 3: Initial Access via RDP
### Risks of Exposed RDP
As a SOC analyst, you should know that if you expose RDP to the world and set a "12345678" password, your host will be breached within a few days. However, not everyone understands the security risks of an exposed RDP. According to Censys Search(opens in new tab), there are over 5,000,000 RDP-enabled machines right now, and many of them are already under threat actors' control. The problem is so widespread that defenders often call RDP the Ransomware Deployment Protocol, emphasizing how often an RDP breach directly results in a ransomware attack.

### Detecting RDP Breach
<img width="679" height="812" alt="image" src="https://github.com/user-attachments/assets/d7fa646d-b0d4-40cb-adf3-b26507a9f7d1" />

### Logging Brute Force
Interestingly, it is not that hard to spot an exposed RDP just from the Security logs. If you would assign a public IP to your server, disable the firewall, enable RDP, and wait around an hour - you would see the logs just like on the screenshot. Botnets around the world will immediately start brute forcing your server, generating hundreds of 4625 events that you won't miss! Note, however, that RDP can be breached without a brute force if threat actors knew the credentials in advance(opens in new tab), but that is a topic for another room.

<img width="1639" height="384" alt="image" src="https://github.com/user-attachments/assets/1be07867-3b60-479f-9d3a-9ca85893c6c1" />


### Lab
**Q1: Which user seems to be most actively brute-forced by botnets?**
<img width="610" height="308" alt="image" src="https://github.com/user-attachments/assets/7a68b851-7f0e-40f3-ab07-e6e0543fe809" />

**Q2: Which IP managed to breach the host via RDP (Logon Type 10)?**
<img width="613" height="411" alt="image" src="https://github.com/user-attachments/assets/8018bd2d-3588-4ad5-a543-9267007eaf25" />

**Q3: Can you get the real Workstation Name (hostname) of the threat actor?**
<img width="767" height="457" alt="image" src="https://github.com/user-attachments/assets/d6a50542-33ae-45cc-b3f1-886e04bd0b72" />

## Task 4: Initial Access via Phishing
### Current State of Phishing
Phishing attacks are still a major threat as they can't be mitigated as easily as blocking RDP access. If users have access to the Internet, they will eventually bring malware to their laptops, bypassing the firewall entirely. According to the HoxHunt Phishing Trends Report for 2025(opens in new tab), phishing attacks have increased 41 times since the release of ChatGPT in 2022. Even more, the success rate of these campaigns remains high, meaning users are still falling for them. In this task, we'll focus on two phishing techniques that lead to Windows breaches: malicious binaries and LNK attachments

### Binary Attachments
In Windows, there are lots(opens in new tab) of executable extensions, and while most people know not to open untrusted .exe files, they are usually less cautious about .com, .scr, or .cpl files. However, all of them can contain the malware inside. For example, users are very likely to open the attached "tryhatme.com" file name assuming it to be a link to a meeting invite, not a malicious binary. \

To make it worse, Windows hides known file extensions by default, meaning that the file "program.exe" will be shown to you just as "program". Threat actors often abuse it(opens in new tab) by naming their viruses like "invoice.pdf.exe" or "cat.png.com" and changing the icons to fit the topic. See the screenshots below to understand how the malicious file looks for the common user:

### LNK Attachments
To avoid AV detection, threat actors may prefer attaching PowerShell, Visual Basic, or BAT scripts over binaries. A popular way to make the scripts look trustworthy is to hide them behind LNK shortcuts - the same files you have on your Desktop that point to real executables somewhere in the Program Files folder. \

Imagine receiving an email from a local PC store announcing major discounts and asking you to review the details in an attached archive. As in the screenshot below, the Discounts.zip contains two files: a PDF and a shortcut to the website. You carefully analyze the PDF and see that it is just a poster with the latest discounts. Engaged by the news, you rush to open the shortcut, only to find out that it points to a PowerShell command instead of the legitimate website.










