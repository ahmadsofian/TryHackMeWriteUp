### Task 1: Introduction
In this i have learnt about network and its components, explored the concept of network perimeter, identified key perimeter threats and examined firewall logs to monitor normal and suspicious logs.


### Task 3: A Network - Overview
**User Workstation (Endpoints)**
* Most common entry point for attackers, often via phishing emails or malicious downloads.
* Endpoints log can reveal malicious processess but network logs mat first show C2 connections.

**File & Database Server**
* Store business most important assests (data).
* Compromising these means attacker can access to valuable or sensitive data.(Ransomware or Data Exfiltration campaign)

**Application Servers (Web, Email, VPN etc.)
* Web Servers: Host company website and web applications.
* Email Server: Handle corporate communications.
* VPN Gateways: Allow secure remote access to internal resources.
* These are externally facing which makes them high-value targets. Attackers scan them for software vulnerabilites or weak configurations, Exploiting one provide foothold into the internal network.
* From security perspective we need to monitor logs, firewall alerts and IDS signatures.

**Active Directory (AD) / Authentication Servers**
* AD manages users, groups computers and access rights.
* AD is the main component that controls all user account and systems within network.
* Commonly targeted for privilege escalation, persistence and lateral movement.
* A single compromised domain admin account can bring down the entire enterprise.

**Router & Switches (Network Infrastructure)**
* Exposed to intercept and manipulate traffic, create backdoors and open hidden channels to the internet.

**Firewalls / Perimeter Devices**
* Firewall to protects enterprise from direct exposure to the internet.
* Prevent unauthorized access to internal services (databases. RDP).
* Logs every connection attempt, successful or blocked. These logs are often earliest indicators of attack such as port scan, brute-forces attempts or exploitation.


### Task 4: Network Visibility
Effective visibility allows security analysts to detec anomalies, investigate incidents, hunt for threats and ensure compliance.

**Host-Centric Logs**
These logs give detailed of view of whats happening on a specific machine.
* Operating System Logs: Record events like user logons, process creation, service startups and failed login attempts.
* Application Logs: Logs from software running on the hosts (web servers (Apache, Nginx), databases, etc).
* Security Tool Logs: Logs from antivirus software, EDR agents and HIDS.

**Network-Centric Logs**
* Show source and destination IPs,  ports, protocols and action taken.
* Reveal attacker initial reconnaissance attempts, lateral movement between systems or attempt to exfiltrate data.
* Firewalls, IDS/IPS, Router & Switches, Web Proxies and VPN.


### Task 5: Network Perimeter
The network perimeter is the boundary that separates an organization's internal network (trusted zone, like employees, servers, business applications) from the external Internet (untrusted zone). 
It’s the point where data enters or leaves the network.

**The Perimeter**
The perimeter is defined by hardware devices at the edge of the network. However, it also includes virtual gateways, cloud connections, and remote access points in modern environments.
* Firewalls: Gatekeepers that filter traffic between internal and external networks.
* Routers/Gateways: Devices that route traffic and enforce access rules.
* Demilitarized Zone (DMZ): A buffer network segment where public-facing servers (web, mail, VPN) are placed.
* Remote Access Gateways / VPNs: Secure entry points for employees working outside the office.


**Importance of Network Perimeter**
* The network perimeter acts as a gatekeeper, controlling what goes in and out of the network. A collection of security controls and network components.
* Attackers always start probing from outside. Perimeter is the first line of defense and first place SOC analyst will see signs of malicious activity.
* If perimeter is weak or misconfigured, attackers can exploit expose services, perform scanning, launch brute-force and perform data exfiltration.

**Network Perimeter in a Small Enterprise**
* Firewall sits between the Internet and internal LAN.
* Web server placed in DMZ so customers can access the website,
* Internal servers (AD, file, database) live behind firewall and accessible only to employees.
* Employees outside the office connect through a VPN gateway.

<img width="717" height="379" alt="image" src="https://github.com/user-attachments/assets/eac36fda-e4ef-4c2b-94a3-702d5cfb79ac" />


**Why does it matter**
The perimeter is the first line of defense against external threats. Attackers scan perimeter IPs loong for open port and vulnerabilities. 
Misconfigured or weakly protected can lead to:
* unauthorized access
* data breaches
* malware infiltration

**Role of Security Analyst**
* Reviewing firewall logs for blocked/allowed connections
* Identifying scanning attempts or brute-force login attempts
* Flagin unusual outbound traffic that may indicate malware beaconing or exfiltration
* Understand what should and shouldnt be exposed at the perimeter.

### Task 6: Network Perimeters: Monitoring and Protecting
#### Monitoring the perimeter
Monitoring the perimeter means using firewalls, intrusion detection/prevention systems (IDS/IPS), and access control to examine and limit exposure and enforce security rules. This  allows the security analysts to:
* Spot early-stage attacks like port scans or brute-force attempts.
* Detect misconfigurations that leave sensitive services exposed.
* Identify outbound traffic anomalies that may indicate malware or data exfiltration.
* The perimeter logs used in the following scenarios can be found at */Perimeter_logs/task6/* folder on the Desktop.

#### Monitoring the Perimeter in Action
**Scenario 1: Probing for Ports (Port Scanning)**
An attacker is testing your network to see which ports are open or closed, while the firewall is doing its job and blocking them.
<img width="650" height="213" alt="image" src="https://github.com/user-attachments/assets/5bd1b447-2996-420b-85c8-0c4df1eb8e38" />
* The same external IP (203.0.113.10) is trying to connect to multiple ports on the same internal machine quickly.
* Analyst's Verdict: This is a classic port scan. The attacker is looking for an open service to target.

**Scenario 2: Attacking the Web Server (SQL Injection)**
The company website is being attacked. An intrusion detection system (IDS) provides more details than a firewall, identifying the type of attack.
<img width="870" height="480" alt="image" src="https://github.com/user-attachments/assets/ee6359a1-c634-404d-ae93-2d6f19596a47" />
* This log shows a mix of ALLOW and BLOCK actions. An analyst can filter for action=BLOCK to immediately find threats. The WAF does the hard work by blocking the request and telling us why.
* The attack_type="SQL Injection" alert shows the attacker trying to dump database information.
* The attack_type="XSS" alert is an attempt to inject malicious scripts.
* The attack_type="Directory Traversal" alert shows an attempt to read sensitive server files.
* Analyst's Verdict: The WAF successfully identifies and blocks multiple web attack types from suspicious IP. This is a high-confidence alert that an attacker is actively targeting the website.

**Scenario 3: Guessing the Password (VPN Brute-Force)**
An attacker is trying to guess a user's password to gain remote access to the network. This creates a lot of noise in the authentication logs.
<img width="808" height="141" alt="image" src="https://github.com/user-attachments/assets/d8967fde-c87b-41dd-8caa-f59641e27f7a" />
* The log is flooded with FAILED_AUTH and SUCCESS_AUTH events.
* A few successful logins from various IPs are normal.
* The problem is the massive volume of failures.
* To find the attack, an analyst would filter or group the logs by source IP.
* Doing so would immediately reveal that a suspicious user is responsible for hundreds of failed login attempts in a very short period.
* Analyst's Verdict: One suspicious IP is observed conducting a brute-force attack against the VPN gateway. The attacker is attempting to use a list of common usernames ('admin', 'root', 'test', etc.) to find a valid account to compromise. The scattered successful logins are benign traffic from legitimate employees.


### Task 7: Perimeter Logs: Investigating the Breach
#### Method 1: Manual Log Analysis
* Commandline: *head firewall.log*
* Commandline: *head ids_alerts.log*
* Commandline: *head vpn_auth.log*

**Reconnaissance Attempt**
Analyzing the blocked requests in the firewall logs, as below:
* Commandline: *cat firewall.log | grep "BLOCK" | head*
Use the filtering to understand which IP is responsible for the maximum BLOCK requests:
* Commandline: *cat firewall.log | grep "BLOCK" | cut -d' ' -f5 | cut -d: -f1 | sort -nr | uniq -c*

Find if our firewall has allowed any requests to the suspicious IP, as  below:
* Commandline: *cat firewall.log | grep [REDACTED] | grep "ALLOW"*

**VPN Brute-force / Credential Acceess**
Examine the number of failed attempts, using the following command, as shown below:
* Commandline: *cat vpn_auth.log | grep FAIL | cut -d' ' -f3 | sort -nr | uniq -c*
Narrow down on the suspicious IP, as below:
Commandline: *cat vpn_auth.log | grep [REDACTED]*

**Lateral Movement**
Let's filter through the firewall logs and see if we can find the footprints of any lateral movement from the compromised host IP REDACTED.
* Commandline: *cat firewall.log | grep [REDACTED] | grep "ALLOW" | head*  
Let's now pivot to ids_alerts logs, and filter through the compromised IP and see if we can find any intrusion rules triggered, as shown below:
* Commandline: *cat ids_alerts.log | grep [REDACTED] | head*
One of the IDS alerts indicates SMB exploit, which look interesting. Let's narrow down our search by using the following search query:
* Commandline: *cat ids_alerts.log | grep -n [REDACTED] | grep 'SMB' | cut -d' ' -f6,7,8,9,10,19,21 | head*

**C2 Beaconing**
Now that, we have an evidence of the lateral movement of the attacker. Let's hunt for any indicator of C2 communication. If we look at the IDS alerts, we can find a specific alert related to C2 Beaconing, indicating possible C2 communication. Let's use the following search query to see the results:
* Commandline: *cat ids_alerts.log | grep C2 | head*
Let's slice and dice through the results, to filter against the compromised IP responsible for C2 beaconing, as shown below:
* Commandline: *cat ids_alerts.log | grep -n [REDACTED]   | cut -d' ' -f6,7,8,9,10,19,22,23 | head -n 15*
This further affirms that, the infected host is further performing susicious activities. We can use the following command to show the stats of the alerts triggered against infected host.
* Commandline: *cat ids_alerts.log | grep -n [REDACTED] | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head* 
We now have the external IP address acting as a C2 server, recieving the C2 beacons from our compromised host.
*Commandline: *cat ids_alerts.log | grep -n [REDACTED]   | cut -d' ' -f6,7,8,9,10,19,22,23 | uniq -c | sort -nr | head*

**Data Exfiltration Attempt**
Now that, we have identified the C2 communication and examined other alerts as well against suspicious Is, let's now investigate, if there are any indicators of data being exfiltrated out of our network. We will apply filter on the compromised hosts, and examine the traffic originating from those to an external destination IP, as shown below:
* Commandline: *cat firewall.log | grep [REDACTED]  | cut -d' ' -f5,6,7  |   uniq  | sort*
The output clearly shows, the compromised host REDACTED is sending extensive amound of traffic on external IP address. We can also filter on IDS logs, to see the alerts being triggered on these activities from the internal IP.
We have found evidence of data exfiltration attempts. If we dig deep into the ids alerts and pivot correlate the alerts through other log files, we can find more suspicious activities as well.


#### Method 2: Analyzes using Splunk
