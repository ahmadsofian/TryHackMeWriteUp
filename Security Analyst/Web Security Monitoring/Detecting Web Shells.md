## Task 2: Web Shell Overview
### What is a Web Shell?
* A web shell is a malicious program uploaded to a target web server, enabling adversaries to execute commands remotely. Web shells often serve as both an initial access(opens in new tab) method (via file upload vulnerabilities) and a persistence(opens in new tab) mechanism.
* Once access has been gained on a compromised server, attackers can use a web shell to move through the kill chain, performing reconnaissance(opens in new tab), escalating privileges(opens in new tab), moving laterally(opens in new tab), and exfiltrating data(opens in new tab).
* Below is a simple example of a web shell named awebshell.php that can run commands remotely through the web interface. Note that the web shell is located in the /uploads directory of the target server, 10.10.10.100.
<img width="3956" height="2188" alt="image" src="https://github.com/user-attachments/assets/5a47cb3b-82cd-4538-abab-f61af94bd433" />

### Web Shell Deployment
* For an attacker to upload and execute a web shell, a file upload vulnerability, misconfiguration, or prior access to the system is required. These vulnerabilities arise when an application fails to validate file type, extension, content, or destination. While web shells are used as an initial access vector, they can also serve as a persistence mechanism if the attacker has already compromised the system and wants to maintain long-term access.
* Imagine a simple website that allows you to upload your pet photos. The website is intended for storing images. However, if developed insecurely, an attacker may upload a web shell like shell.php or mydog.aspx and gain command execution on the server.


## Task 3: Anatomy of a Web Shell
Web shells rely on the abuse of legitimate functions within programs.
System execution functions in PHP, such as shell_exec(), exec(), system(), and passthru(), can be abused to gain command execution. \
Here is a simple web shell written in PHP. Let's take a look at its functionality.
1. Checks if the cmd parameter is present in the URL ?cmd=whoami
2. Stores the user supplied command in the variable $cmd
3. Executes the command using shell_exec()
4. Displays the output
5. HTML for the user interface
6. Command to execute
7. Output
<img width="1330" height="380" alt="image" src="https://github.com/user-attachments/assets/cf62dfcc-5d57-42dc-b837-f5ae6ba244c4" />

### A Web Shell in Action (Lab)
**Q1: Access the shell and determine which account you have access to by running the whoami command.**
<img width="720" height="230" alt="image" src="https://github.com/user-attachments/assets/29743665-f995-4428-b51f-ae7dbd77bc1a" />

**Q2: List the directory contents and find the flag using the ls and cat commands.**
<img width="720" height="246" alt="image" src="https://github.com/user-attachments/assets/9b0bcd71-ed36-4959-a681-826386db159f" />

## Task 4: Log-Based Detection
### Web Server Logs
* Web shells rely on the abuse of web servers, so web server logs are a natural place to start our hunt for evidence. We will explore what to look for in popular web servers like Apache & Nginx. Understanding the difference between normal and suspicious behavior can help uncover malicious activity.
* While the format of web server logs varies depending on the service, access logs generally follow a similar structure and include the following information.
* The remote log name field is typically represented by a hyphen (-), as it is a legacy field that is rarely used today. However, it still appears in access logs for compatibility. Similarly, the authenticated user field is usually shown as a hyphen as well, unless the server required prior authentication, in which case it may contain the actual username.
<img width="1400" height="260" alt="image" src="https://github.com/user-attachments/assets/7b9e094e-bd05-4c83-8670-65392d18a2fb" />


### Web Indicators
**Unusual HTTP Methods & Request Patterns**
* Repeated GET requests in quick succession could mean an attacker is probing for a valid place to upload a shell
* POST requests to valid upload locations following repeated GET requests
* Repeated GET or POST requests to the same file could indicate web shell interaction
<img width="647" height="556" alt="image" src="https://github.com/user-attachments/assets/ee35f370-46ba-42a2-ad54-e6f576ae99ec" />
Let’s walk through a potential web shell attack sequence using the indicators we've discussed so far.
Note that each request shares the same client IP and user agent. Response codes and timestamps should also be noted.
* 200 OK
* 404 Not Found
<img width="1244" height="590" alt="image" src="https://github.com/user-attachments/assets/d5576ce9-ff8d-4845-9022-737ef31982f9" />

**Suspicious User-Agents & IP Addresses**
The User-Agent identifies the client making requests to the web server and provides information about the browser, device, and operating system.
* Altered User-Agents: Mozilla/4.0+(+Windows+NT+5.1) shortened to Mozilla/4.0
* Outdated User-Agents: Mozilla/4.0 (compatible; MSIE 6.0) MSIE 6.0 released in 2001
* Blacklisted User-Agents: curl/1.XX.X or wget/1.XX.X for example
* Suspicious IP Addresses: A network normally only sees internal traffic so an outside IP address would be suspicious

**Query Strings**
Part of the URL that associates values with a parameter. example.php?query=somequery
* Abnormally long or suspicious query strings, especially containing keywords like cmd= or exec=
* Encoded query strings. ?query=whoami becomes ?query=d2hvYW1p when Base64 encoded. Cyberchef(opens in new tab) is an excellent tool for decoding Base64 and many other forms of encoding and obfuscation.

**Missing Referrer**
The referrer shows the URL the users visited before being linked to the current page.
* A missing referrer can be potentially indicative of web shell activity
* There are valid reasons why a referrer might be missing (e.g., browsers blocking them for privacy, if a URL is directly accessed)

**Sample suspicious web request including some of the above indicators.**
1. Known malicious or untrusted IP address.
2. Abnormal timestamp. Perhaps outside of normal business hours.
3. POST request with a search query string to a malicious file.
4. No referrer. So this page was accessed directly. (not always a valid indicator)
5. A suspicious User-Agent string that is not typically associated with a web browser.
<img width="1400" height="130" alt="image" src="https://github.com/user-attachments/assets/d2b8ce9d-4628-4b63-bb37-0df8977b7f6e" />

### Auditd
A native Linux utility that tracks and records events, creating an audit trail. Rules can be created for auditd, which determine what is logged in the audit.log. Rules can be highly configured to match specific conditions, such as when certain programs are run or files are modified in a particular directory. In the example below, ausearch is used to search for any logs matching the web_shell rule.

### Web & Auditd Correlation
Detecting web shells effectively requires correlating multiple log sources. Combining web access and error logs with auditd provides more insight and can confirm if a file was created, modified, or executed, and by which user or process. A suspicious POST request in web logs can be linked to an audit event that includes a creat or execve syscall, showing a script wrote a file or ran commands. Combining this information helps us build a clearer picture of the attack sequence.

### Leverage SIEM Platforms
Some benefits of Security Information and Event Management (SIEM) platforms include: 
* Centralized log collection and correlation, which is especially useful when dealing with many different log types.
* Targeted queries can be created to uncover signs of malicious activity, including web shells.
* Allows analysts to search and analyze logs more efficiently.


## Task 5: Beyond Logs
### File System Analysis
An attacker's web shell must be stored somewhere. Analyzing web server files is crucial in identifying uploaded web shells or locating files modified to include a web shell payload.
It should be noted that some platforms like WordPress and Django store page content in a database rather than a file system, so malicious code may be injected into posts, themes, or settings and won't appear in normal file system searches.

Here are some common web server directories where web shells are typically placed:

* Apache: */var/www/html/* Default on most Linux distros
* Nginx: */usr/share/nginx/html/* Default on many Linux setups

Even if a custom root path is configured, attackers can guess or scan for common upload directories, like /uploads/, /images/, or /admin/ like we saw in Task 3.
Temporary directories like /tmp can also be abused if file permissions are not configured securely. \

**Suspicious or Random File Names**
Can be used by attackers to evade detection. Be on the lookout for names that deviate from standard application files. 
* Monitor files with executable extensions like .php & .jsp
* Be on the lookout for double extensions to disguise malicious files image.jpg.php \

**Helpful Commands**
You can use find to search for recently modified scripts. In the example below, it is used to search the /var/www directory for .php files modified between two specific dates using the -newerct option. Another helpful tool, grep, can be used to track down suspicious functions like eval( within files. In the example below, we use it to search the WordPress directory wp-content.

### Network Traffic Analysis
Network traffic analysis allows analysts to go beyond logs by examining the data exchanged between a client and a server. By inspecting packet payloads, it becomes possible to observe attacker behavior on a more detailed level. \

Many of the indicators that analysts need to be on the lookout for in log analysis can be applied to network traffic analysis as well.
* Unusual HTTP Methods & Request Patterns
* Suspicious User-Agents & IP Addresses
* Encoded Payloads
* Malicious Code or Commands in Request Bodies
* Unexpected Protocols or Ports
* Unexpected Resource Usage
* Web Server Processes Spawning Command Line Tools
Packet captures provide important details that can be critical in confirming an attack. For example, in a PCAP showing a POST /upload.php request, you might see a PHP web shell (webshell.php) being uploaded directly with the shell's source code visible in the payload. This level of detail helps further validate exploitation attempts. \

Some useful Wireshark http filters.
* *http.request.method == “METHOD”*
Hunting for repeated or unusual requests can be useful
* *http.request.uri contains “.php”*
Can be helpful in finding suspicious or modified files
* *http.user_agent*
Used to locate unusual or outdated User-Agents
<img width="2080" height="504" alt="image" src="https://github.com/user-attachments/assets/49617b96-7265-4801-a445-51a5344f9290" />
<img width="2080" height="1352" alt="image" src="https://github.com/user-attachments/assets/56c74d4c-3aaa-4473-b9ec-313333db8585" />

## Task 6: Investigation (Lab)
**Q1: Which IP address likely belongs to the attacker?**
<img width="720" height="178" alt="image" src="https://github.com/user-attachments/assets/d851b23b-8408-4051-90b5-75065b555f4f" />  \

**Q2: What is the first directory that the attacker successfully identifies?**
<img width="720" height="159" alt="image" src="https://github.com/user-attachments/assets/3558da34-3c92-40d6-a34f-526de314e7d5" />  \

**Q3: What is the name of the .php file the attacker uses to upload the web shell?**
<img width="720" height="126" alt="image" src="https://github.com/user-attachments/assets/9a99109b-9c9f-4501-827a-e9de39f08fdb" /> \


**Q4: What is the first command run by the attacker using the newly uploaded web shell?**
<img width="720" height="100" alt="image" src="https://github.com/user-attachments/assets/d20aa9f2-8261-4d88-bd2e-a11a2f4b5b38" /> \

**Q5: After gaining access via the web shell, the attacker uses a command to download a second file onto the server. What is the name of this file?**
<img width="720" height="71" alt="image" src="https://github.com/user-attachments/assets/2d72fed3-ec82-40d6-ab44-a55f1e9bc211" />

**Q6: The attacker has hidden a secret within the web shell.
Use cat to investigate the web shell code and find the flag.**
<img width="720" height="158" alt="image" src="https://github.com/user-attachments/assets/29b6f90a-c739-46ca-b767-f988334fc7f5" />




