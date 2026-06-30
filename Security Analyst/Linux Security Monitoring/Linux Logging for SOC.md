## Task 2: Working With Text Logs
Unlike in Windows, Linux logs most events into plain text files. This means you can read the logs via any text editor without the need for specialized tools like Event Viewer. On the other hand, default Linux logs are less structured as there are no event codes and strict log formatting rules. Most Linux logs are located in the /var/log folder, so let's start the journey by checking the /var/log/syslog file - an aggregated stream of various system events:
* *root@thm-vm:~$ cat /var/log/syslog | head*

**Filtering Logs**
You will see thousands of events when reading the syslog file on the attached VM, but only a few are useful for SOC. That's why you must filter logs and narrow down your search as much as possible. For example, you can use the "grep" command to filter for the "CRON" keyword and see only the cronjob logs:
* *Or "grep -v CRON" to exclude "CRON" from results*
* *root@thm-vm:~$ cat /var/log/syslog | grep CRON*

**Discovering Logs**
Lastly, let's say you hunt for all user logins, but don't know where to look for them. Linux system logs are stored in the /var/log/ folder in plain text, so you can simply grep for related keywords like "login", "auth", or "session" in all log files there and narrow down your next searches:
* *root@thm-vm:~$ ls -l /var/log*
* *root@thm-vm:~$ grep -R -E "auth|login|session" /var/log*

### Lab
**Q1: Use the /var/log/syslog file on the VM to answer the questions.
Which time server domain did the VM contact to sync its time?**
* The question asks for the time server domain, so the obvious place to look is system logs (usually /var/log/syslog on Debian/Ubuntu).
* The pattern to search for is any NTP-related entry.
* Use command *grep -i ntp /var/log/syslog*

<img width="1349" height="125" alt="image" src="https://github.com/user-attachments/assets/e6731ec0-26cc-4996-b4ec-2f54026c7587" />

**Q2: What is the kernel message from Yama in /var/log/syslog?**
* Yama is a Linux Security Module (LSM) that enforces stricter ptrace and debugging policies.
* Use command *grep -i Yama /var/log/syslog*

## Task 4: Authentication Logs
The first and often the most useful log file you want to monitor is /var/log/auth.log (or /var/log/secure on RHEL-based systems). Although its name suggests it contains authentication events, it can also store user management events, launched sudo commands, and much more! Let's start with the log file format:
<img width="1138" height="253" alt="image" src="https://github.com/user-attachments/assets/35374abd-4f7b-4172-a2db-7b20d3985822" />

### Login and Logout Event
There are many ways users authenticate into a Linux machine: locally, via SSH, using "sudo" or "su" commands, or automatically to run a cron job. Each successful logon and logoff is logged, and you can see them by filtering the events containing the "session opened" or "session closed" keywords:
* Command: *root@thm-vm:~$ cat /var/log/auth.log | grep -E 'session opened|session closed'*
In addition to the system logs, the SSH daemon stores its own log of successful and failed SSH logins. These logs are sent to the same auth.log file, but have a slightly different format. Let's see the example of two failed and one successful SSH logins:
* Command: *root@thm-vm:~$ cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'*


### Miscellanous Events
You can also use the same log file to detect user management events. This is easy if you know basic Linux commands: If useradd(opens in new tab) is a command to add new users, just look for a "useradd" keyword to see user creation events! Below is an example of what you can see in the logs: password change, user deletion, and then privileged user creation.
* Command: *root@thm-vm:~$ cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['*
Lastly, depending on system configuration and installed packages, you may encounter interesting or unexpected events. For example, you may find commands launched with sudo, which can help track malicious actions. In the example below, the "ubuntu" user used sudo to stop EDR, read firewall state, and finally access root via "sudo su":
* Command: *root@thm-vm:~$ cat /var/log/auth.log | grep -E 'COMMAND='*

### Lab
**Q1: Continue with the VM and use the /var/log/auth.log file.
Which IP address failed to log in on multiple users via SSH?**
* Filter SSH-related messages for failures
* Use command: *cat /var/log/auth.log | grep "Failed"*

<img width="1020" height="127" alt="image" src="https://github.com/user-attachments/assets/e642f5e2-c5e2-4e10-ab30-35f2066caf67" />

**Q2: Which user was created and added to the "sudo" group?**
* User management is logged in the same auth log. Search for useradd and usermod:
* Use Command: *cat/var/log/auth.log | grep group*

<img width="767" height="81" alt="image" src="https://github.com/user-attachments/assets/e20fe216-7327-40c3-b469-e89181129c24" />

## Task 4: Common Linux Logs
### Generic System Logs
Linux keeps track of many other events scattered across files in /var/log: kernel logs, network changes, service or cron runs, package installation, and many more. Their content and format can differ depending on the OS, and the most common log files are:
* /var/log/kern.log: Kernel messages and errors, useful for more advanced investigations
* /var/log/syslog (or /var/log/messages): A consolidated stream of various Linux events
* /var/log/dpkg.log (or /var/log/apt): Package manager logs on Debian-based systems
* /var/log/dnf.log (or /var/log/yum.log): Package manager logs on RHEL-based systems

### App-Specific Logs
In SOC, you might also monitor a specific program, and to do this effectively, you need to use its logs. For example, analyze database logs to see which queries were run, mail logs to investigate phishing, container logs to catch anomalies, and web server logs to know which pages were opened, when, and by whom. You will explore these logs in the upcoming modules, but to give an overview, here is an example from the typical Nginx web server logs:
* Command: *root@thm-vm:~$ cat /var/log/nginx/access.log*

### Bash History
Another valuable log source is Bash history - a feature that records each command you run after pressing Enter. By default, commands are first stored in memory during your session, and then written to the per-user ~/.bash_history file when you log out. You can open the ~/.bash_history file to review commands from previous sessions or use the history command to view commands from both your current and past sessions:
* *ubuntu@thm-vm:~$ cat /home/ubuntu/.bash_history*
* *ubuntu@thm-vm:~$ history*
Although the Bash history file looks like a vital log source, it is rarely used by SOC teams in their daily routine. This is because it does not track non-interactive commands (like those initiated by your OS, cron jobs, or web servers) and has some other limitations. While you can configure it to be more useful, there are still a few issues you should know about:

### Lab
**Q1: According to the VM's package manager logs,
which version of unzip was installed on the system?**
* Use command: *cat /var/log/dpkg.log | grep unzip*

<img width="730" height="174" alt="image" src="https://github.com/user-attachments/assets/c2673c22-78f7-4d06-9a99-c11013a2295e" />

**Q2: What is the flag you see in one of the users' bash history?**
* A user’s Bash history is stored in ~/.bash_history, but not every account uses Bash (some use /bin/sh) and not every user leaves a history file.
* Use command: *cat .bash_history*

<img width="456" height="446" alt="image" src="https://github.com/user-attachments/assets/79ca9c99-557b-48b4-8d09-46794bef5648" />

## Task 5: Runtime Monitoring
Up to this point, you have explored various Linux log sources, but none can reliably answer questions like "Which programs did Bob launch today?" or "Who deleted my home folder, and when?". That's because, by default, Linux doesn't log process creation, file changes, or network-related events, collectively known as runtime events. Interestingly, Windows faces the same limitation, which is why in the Windows Logging for SOC room we had to use an additional tool: Sysmon. In Linux, we'll take a similar approach.

### System Calls
Before moving on, let's explore a core OS concept that might help you understand many other topics: system calls. In short, whenever you need to open a file, create a process, access the camera, or request any other OS service, you make a specific system call. There are over 300(opens in new tab) system calls in Linux, like execve to execute a program. Below is a high-level flowchart of how it works:

<img width="1152" height="213" alt="image" src="https://github.com/user-attachments/assets/7c240378-ac22-4fbe-a594-017fec7de216" />

## Task 6: Using Auditd
### Audit Daemon
Auditd (Audit Daemon) is a built-in auditing solution often used by the SOC team for runtime monitoring. In this task, we will skip the configuration part and focus on how to read auditd rules and how to interpret the results. Let's start from the rules - instructions located in /etc/audit/rules.d/ that define which system calls to monitor and which filters to apply:
<img width="1138" height="270" alt="image" src="https://github.com/user-attachments/assets/fadcd4e8-c797-493a-82ac-da93aa8389e7" />



### Using Auditd
You can view the generated logs in real time in /var/log/audit/audit.log, but it is easier to use the ausearch command, as it formats the output for better readability and supports filtering options. Let's see an example based on the rules from the example above by searching events matching the "proc_wget" key:
* *root@thm-vm:~$ ausearch -i -k proc_wget*

The terminal above shows a log of a single "wget" command. Here, auditd splits the event into four lines: the PROCTITLE shows the process command line, CWD reports the current working directory, and the remaining two lines show the system call details, like:
* *pid=3888, ppid=3752:* Process ID and Parent Process ID. Helpful in linking events and building a process tree
* *auid=ubuntu:* Audit user. The account originally used to log in, whether locally (keyboard) or remotely (SSH)
* *uid=root:* The user who ran the command. The field can differ from auid if you switched users with sudo or su
* *tty=pts1:* Session identifier. Helps distinguish events when multiple people work on the same Linux server
* *exe=/usr/bin/wget:* Absolute path to the executed binary, often used to build SOC detection rules
* *vkey=proc_wget:* Optional tag specified by engineers in auditd rules that is useful to filter the events

**File Events**
Now, let's look at the file events matching the "file_sshconf" key. As you may see from the terminal below, auditd tracked the change to the /etc/ssh/sshd_config file via the "nano" command. SOC teams often set up rules to monitor changes in critical files and directories (e.g., SSH configuration files, cronjob definitions, or system settings)
* *root@thm-vm:~$ ausearch -i -k file_sshconf*

### Auditd Alternatives
You might have noticed an inconvenient output of auditd - although it provides a verbose logging, it is hard to read and ingest into SIEM. That's why many SOC teams resort to the alternative runtime logging solutions, for example:
* **Sysmon for Linux:** A perfect choice if you already work with Sysmon and love XML
* **Falco:** A modern, open-source solution, ideal for monitoring containerized systems
* **Osquery:** An interesting tool that can be broadly used for various security purposes
* **EDRs:** Most EDR solutions can track and monitor various Linux runtime events

## Lab
**Q1: When was the secret.thm file opened for the first time? (MM/DD/YY HH:MM:SS)
Note: Access to this file is logged with the "file_thmsecret" key.**
* Search the audit logs (human-readable output via ausearch -i), or directly inspect /var/log/audit/audit.log for the file_thmsecret key:
* Use command: *ausearch -i | grep "secret.thm"*

**Q2: What is the original file name downloaded from GitHub via wget?
Note: Wget process creation is logged with the "proc_wget" key.**
* Audit logs record process invocation, including the wget command. Grep for wget in auditd records:
* Use command: *ausearch -i | grep "github"*

**Q3: Which network range was scanned using the downloaded tool?
Note: There is no dedicated key for this event, but it's still in auditd logs.**
* Auditd records process invocations for ./naabu and capture arguments. Search for naabu in the audit log:
* Use command: *ausearch -i | grep naabu*








