## Task 2: Initial Access via SSH
Much like with RDP on Windows, SSH is both powerful and often poorly defended - in fact, both protocols are tracked under the External Remote Services MITRE technique. A lot of threat groups run vast botnets to scan the Internet for systems with exposed SSH and access them in two primary ways: via a stolen key or a breached password. Let's see how it usually happens:

1. Common risks when using key-based authentication:
  * Threat actors access a service or source code where private SSH keys have been stored (Like a GitHub repository or Ansible automation server containing SSH credentials)
  * Threat actors steal SSH keys to a server by infecting an admin's laptop with a data stealer

2. Additional risks when using password-based authentication:
* An IT admin sets a weak SSH password for a quick test and forgets to revert the changes
* An IT support enables SSH for a contractor who sets the password to "12345678"
* A network engineer accidentally exposes an old, insecure SSH server to the Internet
Most real-world Linux attacks, such as those by the Outlaw(opens in new tab) group, start from one of the scenarios above. However, you should also be aware of more advanced risks like a vulnerability in the SSH server itself, notably Erlang/OTP or SSH session hijacking(opens in new tab), that you will learn in more advanced rooms.

### Lab
**Q1: When did the ubuntu user log in via SSH for the first time?**
<img width="1015" height="38" alt="image" src="https://github.com/user-attachments/assets/12b1c114-b9e0-4a6f-8855-39ee22d7e459" />

## Task 3: Detecting SSH Attacks
### SSH Breach Example
Now, imagine a common real-world scenario: An IT administrator enables public SSH access to the server, allows password-based authentication, and sets a weak password for one of the support users. Combined, these three actions inevitably lead to an SSH breach, as it's a matter of time before threat actors guess the password. The log sample below shows such a compromise: A brute force followed by a password breach. There are three indicators of malicious logins to pay attention to:
<img width="2080" height="440" alt="image" src="https://github.com/user-attachments/assets/bc64053e-0fbd-4580-858d-6f510d97213d" />

### Detecting SSH Attacks
On Linux, you don't need to learn a dozen fields like logon type to figure out what's going on, making log analysis more straightforward. Your starting point in detecting SSH attacks can be as simple as listing all successful SSH logins and analyzing a few fields. Let's imagine you queried the logs and found three successful SSH logins, each of which could indicate an attack. How would you distinguish bad from good?
* *ubuntu@thm-vm:~$ cat /var/log/auth.log | grep -E 'Accepted'*

**Login of Ansible**
The first login appears legitimate: It used public-key authentication from an internal IP, likely an Ansible automation account. Moreover, the login at exactly 14:00 matches periodic task behavior. But to be sure, you'd still need to verify that 10.14.105.255 is an Ansible server and review the following user's activity for signs of a breach.

**Login of Jsmith**
The two logins of jsmith are more interesting, as there are three red flags: Password-based authentication, logins from external IPs, and time difference between the logins (one of the logins must be at night for the user, right?). Still, to make a final verdict, you might need to investigate more details:
* Username: Who owns the user? Is it expected for them to log in at this time and from this IP?
* Source IP: What do TI tools and asset lookups say about the IP? Is it trusted or malicious?
* Login history: Was the login preceded by brute force or other suspicious system events?
* Next steps: Is the login suspicious? Should I analyze user actions following the login?

### Lab
**Q1: When did the SSH password brute force start?**
<img width="1680" height="81" alt="image" src="https://github.com/user-attachments/assets/8fffe9af-bbd3-4ddf-af30-43f92c0c22fe" />

**Q2: Which four users did the botnet attempt to breach?**
<img width="1423" height="376" alt="image" src="https://github.com/user-attachments/assets/f29d1d78-c774-459d-aa9b-e29bf2fbfd97" />

**Q3: Finally, which IP managed to breach the root user?**
<img width="1155" height="112" alt="image" src="https://github.com/user-attachments/assets/68c84836-415a-47fc-a04d-b046b9c80c03" />

## Task 4: Initial Access via Services
### Linux and Public Services
Linux systems often host public-facing services or applications such as web servers, email servers, databases, and various development or IT management tools. They also comprise the core of most firewall or VPN software. However, whenever one of these applications is compromised, the entire Linux host is at risk. This risk is covered with the T1190(opens in new tab) MITRE technique. Let's see a few real-world examples:
* CVE in Zimbra Collaboration(opens in new tab): Allowed the attackers to execute arbitrary OS commands
* Exposed Docker API port(opens in new tab): Acted as an entry point in a series of cloud infrastructure breaches
* CVE in Palo Alto firewalls(opens in new tab): Granted attackers full control over the Linux-based firewall's OS
* WordPress "plugins" feature(opens in new tab): Often abused to upload malware like web shells to the system
<img width="1190" height="190" alt="image" src="https://github.com/user-attachments/assets/526a3db7-2e32-4123-b646-89c036fd372f" />

### Using Application Logs
If you want to know whether your email server was breached, you naturally reach for the email logs. On the other hand, can you expect an application to log "I am being exploited with a zero-day right now"? Of course not. That’s the nature of application logs - they rarely tell the full story, but they can still provide unique artifacts for analysis. For example, you can:
* Use web logs to detect a variety of web attacks
* Use database logs to detect suspicious SQL queries
* Use VPN logs to detect abnormal VPN login events
* Refer to other logs for specific events like bank transactions

### Web as Initial Access
Any publicly exposed application can lead to a Linux breach, especially vulnerable web servers. Let's see an example: The IT team creates a simple web application called TryPingMe, where you can ping the specified IP online. Internally, the app runs a system command ping -c 2 [YOUR-INPUT] to test the connection, without any input filtering. The attackers would easily spot a command injection there, but can you spot the exploitation in the TryPingMe web logs?
* *ubuntu@thm-vm:~$ cat /var/log/nginx/access.log*

**Web Logs Analysis**
The requests coming from 10.14.105.255 seem odd. Instead of IPs, the client puts Linux commands inside the query parameters - a clear sign of command injection! Although now you will need to start a deep investigation to unravel the whole story, from web logs alone you can assume that:
* 10.14.105.255 is likely the attacker's IP
* The /ping page is vulnerable and allows code execution
* The attacker executed OS commands like whoami and ls
* The entire system is now at risk because of the TryPingMe vulnerability

### Lab
**Q1: What is the path to the Python file the attacker attempted to open?**
<img width="1571" height="300" alt="image" src="https://github.com/user-attachments/assets/4e82eec3-91e8-444f-8aaa-9766bd26ce3a" />

**Q2: Looking inside the opened file, what's the flag you see there?**
<img width="572" height="462" alt="image" src="https://github.com/user-attachments/assets/8c2f2282-3e6d-4362-9a85-b6e8a9aac348" />

## Task 5: Detecting Service Breach
### Building Process Tree
One way to detect a service breach is to use application logs, like you did in the previous task. But remember, application logs are not always available or helpful. Instead, most SOC teams rely on process tree analysis - a universal approach to unwrapping the Initial Access. For example, in this report(opens in new tab), Wiz used a process tree to visually highlight how exactly Selenium servers were breached and how to spot it in process creation logs.
A common SOC scenario is when you receive an alert about a suspicious command, let's say whoami. Why was it executed - due to IT activity, or maybe a service breach? To answer this, all you need is to build a process tree and trace the command back to its parent process, as shown in the image below:
<img width="2080" height="610" alt="image" src="https://github.com/user-attachments/assets/288f5279-d9af-4f68-9483-e1b1970f8e9b" />

### Auditd and Process Tree
Continuing the example, you begin by locating the suspicious command in the logs with ausearch -i -x whoami. Next, you walk up the process tree using the --pid option until you reach PID 1, the OS process. The tree eventually shows that whoami was launched by a Python web application (/opt/mywebapp/app.py). This immediately raises the question: Was the application breached and used as an entry point?
* *ubuntu@thm-vm:~$ ausearch -i -x whoami # -x filters the results by the command name*
* *ubuntu@thm-vm:~$ ausearch -i --pid 3905 # 3905 is a parent process ID of whoami*
* *ubuntu@thm-vm:~$ ausearch -i --pid 3898 # 3898 is a grandparent process ID of whoami*

Next, you might wonder if whoami is simply part of the application's normal behavior. Maybe so, but that question would require web logs analysis, external research, or communication with the developers. What you can do instead is use the process tree to look for other, more dangerous commands launched by the app. By listing all child processes of /opt/mywebapp/app.py, you may find clearer evidence of the app's breach, like a malicious curl command!
* *ubuntu@thm-vm:~$ ausearch -i --ppid 3898 | grep 'proctitle' # Use grep for a simpler output*

### Lab
**Q1: What is the PPID of the suspicious whoami command?**
<img width="1862" height="132" alt="image" src="https://github.com/user-attachments/assets/db72d72b-b370-4bd2-a4e1-7e37efcade0f" />

**Q2: Moving up the tree, what is the PID of the TryPingMe app?**
<img width="1848" height="138" alt="image" src="https://github.com/user-attachments/assets/de41757f-3725-4ddc-a537-d27c2520d4f5" />

**Q3: Which program did the attacker use to open a reverse shell?**
<img width="1035" height="65" alt="image" src="https://github.com/user-attachments/assets/c53c9ecc-c0a3-4caf-85d5-3921eb555b5e" />

## Task 6: Advanced Initial Access
### Human-Led Attacks
In the previous tasks, you explored Initial Access via SSH and exposed services. But what about phishing and USB attacks, so commonly seen in Windows environments? Since Linux primarily is a server OS operated by technical people, it is harder to trick system owners into running phishing malware or inserting a malicious USB. Still, the risk remains, for example:
<img width="1235" height="230" alt="image" src="https://github.com/user-attachments/assets/37746a14-cdcb-4c09-9d80-668ef917fad0" />

### Supply Chain Compromise
While not unique to Linux, you should also be aware of Supply Chain Compromise(opens in new tab). These attacks breach a software first, and then infect all its users with the malicious update. Since a typical Linux server uses hundreds of software dependencies maintained by different developers, the attack can come from anywhere, anytime. Let's see some examples:
* A backdoor in the XZ Utils(opens in new tab) library that is a part of SSH nearly led to a breach of millions of Linux servers
* A breach of the tj-actions(opens in new tab) resulted in a leak of thousands of secrets, like SSH keys and access tokens

### Detecting the Attacks
All Initial Access techniques described in this room can be uncovered through a process tree analysis. You start with a trigger, such as a SIEM alert on a suspicious command or a connection to a known malicious IP. From there, you build a process tree to trace which application or user initiated the events - a web server, an internal application, or an IT administrator’s SSH session. Finally, you determine whether the activity is legitimate or an indicator of malicious behavior:
<img width="2050" height="430" alt="image" src="https://github.com/user-attachments/assets/cdffc765-d973-4b6d-9d8c-6d69275ed7da" />





