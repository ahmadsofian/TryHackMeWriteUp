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
<img width="677" height="270" alt="image" src="https://github.com/user-attachments/assets/95207896-a291-48c9-bd6c-0e8bcd7d21f1" />

### LNK Attachments
To avoid AV detection, threat actors may prefer attaching PowerShell, Visual Basic, or BAT scripts over binaries. A popular way to make the scripts look trustworthy is to hide them behind LNK shortcuts - the same files you have on your Desktop that point to real executables somewhere in the Program Files folder. \

Imagine receiving an email from a local PC store announcing major discounts and asking you to review the details in an attached archive. As in the screenshot below, the Discounts.zip contains two files: a PDF and a shortcut to the website. You carefully analyze the PDF and see that it is just a poster with the latest discounts. Engaged by the news, you rush to open the shortcut, only to find out that it points to a PowerShell command instead of the legitimate website.

<img width="679" height="268" alt="image" src="https://github.com/user-attachments/assets/9826e4f8-1c27-4a18-adc9-8f2b4a2b477b" />

Threat actors can include any command inside the LNK "Target" field, as well as set any shortcut icon. You can verify it by right-clicking the LNK file, selecting "Properties", and viewing the "Shortcut" tab. The case shown above, for example, downloads and executes a simplified version of RemcosRAT - malware used in many attacks on major companies and government agencies. The terminal below shows a full LNK payload:

<img width="684" height="201" alt="image" src="https://github.com/user-attachments/assets/6c9726f6-87d2-4f5a-8d99-c8a83788e1ae" />

### Lab
**Q1: Let's play the role of the untrained user and mindlessly open the COM file.
Run the www.skype.com file from the Phishing Case 1 folder, which flag do you get?**
<img width="619" height="199" alt="image" src="https://github.com/user-attachments/assets/8da692cc-c081-4b05-adc9-15842f3f3356" />

**Q2: Continue with the second attachment from the Phishing Case 2 folder.
From which URL does the malicious LNK download the next stage malware?**
<img width="731" height="375" alt="image" src="https://github.com/user-attachments/assets/7e51eed8-600e-4692-8bd6-e4fd001a161c" />

**Q3: Finally, move on to the Phishing Case 3 folder and review its content.
What is the name of the double-extension file you see there?**
<img width="766" height="248" alt="image" src="https://github.com/user-attachments/assets/046319f1-0c77-4a3c-9a51-fe5f263f456a" />

## Task 5: Continuing Phishing Topic
### Detecting Malicious Download
It is relatively simple to hunt for malicious downloads if you know how the victim sees it. First, the user uses a web browser or desktop application to open a phishing attachment. In the simplest case, it would be a direct .exe malware download, but you are far more likely to see an archive attachment like .zip or .rar containing the malware. In this case, Sysmon can greatly help you detect every attack stage:
<img width="678" height="658" alt="image" src="https://github.com/user-attachments/assets/9abccd78-5946-4fb1-9f1c-bbc8c645c6d6" />

### Notes on LNK Attachments
Unlike with binary attachments, LNK files have a very interesting and important capability - they leave little execution trace. Consider the case on the screenshot below, where a user downloaded LNK file that looks like a Google Chrome shortcut, but in fact runs some PowerShell payload.

After the user launches the shortcut - Windows Explorer reads the "Target" field of the LNK and makes it look like explorer.exe launches PowerShell directly. Still, you can identify if it was indeed LNK phishing or another attack vector by looking for the preceding file creation events - LNK files must have appeared somewhere in Downloads before:
<img width="2732" height="1022" alt="image" src="https://github.com/user-attachments/assets/5f58bd89-e507-4295-a493-ffcce54fb430" />

### Lab

**Q1: Which file did the user download via the web browser?**
<img width="481" height="298" alt="image" src="https://github.com/user-attachments/assets/09c1ae02-ed63-423b-b80e-5a4c309c4405" />

**Q2: In which folder did the user unarchive the suspicious file?**
<img width="768" height="288" alt="image" src="https://github.com/user-attachments/assets/b2a2931e-6efd-491e-bbb4-b69f53546596" />

**Q3: What is the process ID of the launched phishing malware?**
<img width="766" height="299" alt="image" src="https://github.com/user-attachments/assets/f792f324-34df-400e-b12b-1bf4ccc7eb27" />

**Q4: Finally, which malicious domain did the malware try to connect to?**
<img width="767" height="280" alt="image" src="https://github.com/user-attachments/assets/163afa40-9aa0-47e6-bc67-869ed9220997" />


## Task 6: Initial Access via USB
### Risk of Removable Media

Although some may believe that days of infected USB flash drives are long gone and cloud services have replaced them completely, threat actors will disagree, as proven by Camaro Dragon or Raspberry Robin attacks. Moreover, Initial Access via an infected USB bypasses firewalls, much like phishing, and can start the attack chain even without Internet access and continue spreading without user interaction.

**USB Delivery Case**
Imagine working for TryHatMe Inc. and receiving a delivery package with a fancy hat and a USB labelled as "A gift from HR". You plug it in, a harmless GIF pops up, and you call HR to thank them for the present. But while the HR figures out what you meant, the malware from the USB has already spread to your laptop. (Real-World Case)

**Print Service Case**
Another common scenario involves third-party entities like a print service. Suppose you visit one and hand over your USB to print a document. Their system, already infected with a worm, passes the malware onto your flash drive. Then, you bring the malware back to your home PC, and the infection chain continues. Now, let's learn how to detect this before it's too late! (Real-World Story)

### Detecting an Infected USB
Although there are many advanced techniques on how to run the malware from USB automatically as soon as the flash drive is plugged in, the majority of cases occur just because the user launches malware themselves. For example:

* Malware hides all legitimate files on USB and creates a malicious "RECOVERY.lnk" file
* Malware creates a "Photos.exe" binary and sets its icon to look like a simple folder
* Malware creates double-extension copies of all files, like "photo_2024_1_12.jpg.exe"
Interestingly, the detection of Initial Access via USB looks very similar to the phishing attachments. Since both methods rely on a user running malicious binary via a graphical interface (explorer.exe), you may have a hard time understanding how exactly the attack started. Still, in some cases, you may find evidence of execution from external drives like "E:\malware.exe":
<img width="1155" height="530" alt="image" src="https://github.com/user-attachments/assets/a6b2e2ff-3302-4932-9515-d0027c52633c" />

### Lab
**Q1: Which USB file was launched by the user?**
<img width="767" height="217" alt="image" src="https://github.com/user-attachments/assets/219ed354-b38e-46c5-8756-a27a0567860a" />

**Q2: Which suspicious file did the malware drop to the disk?
(Format: full path to the file, e.g. C:\file.txt)**
<img width="766" height="198" alt="image" src="https://github.com/user-attachments/assets/637e9997-2635-4ab8-b9e6-f52528cb31ea" />

**Q3: To which other USB did the malware propagate?
(Format: just the letter, e.g. X:)**
<img width="767" height="201" alt="image" src="https://github.com/user-attachments/assets/906393a0-51b7-443a-ab32-e5cb0700b831" />



















