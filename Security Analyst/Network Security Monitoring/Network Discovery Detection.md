### Task 1: Introduction
This room aimed to understand network discovery, why attackers perform network discovery, types network discovery and the techniques

### Task 2: Network Discovery
**Attackers and Network Discovery**
During the discovery phasem attacker attempts to ascertain some information:
* What assets can be accessed by the attacker?
* What are the IP addressess, ports, OS and services running on these assets?
* What versions of services are running? Does any service have a vulnerability that can be exploited?

**Defenders and Network Discovery**
During this activity, defenders want to achieve the following goals:
* Inventory the organization's assets and ensure all assets are documented.
* Ensure no unnecessary IP, port or service is open
* Ensure vulnerabilities are patched, or at least the exploitable vulnerabities are patched.

### Task 3: External vs Internal Scanning
**External Scanning Activity**
This type of scan indicates that the attack is still in the Reconnaissance(opens in new tab) phase of the MITRE ATT&CK lifecycle. The attacker does not have a foothold inside the network and is doing initial reconnaissance to identify opportunities to gain initial access to the network.

**Internal Scanning Activity**
This type of scan indicates that the attack has progressed to the Discovery(opens in new tab) phase of the MITRE ATT&CK lifecycle. In some other frameworks, it might also be called internal reconnaissance, indicating that the attacker has a foothold in the network and is now gearing up for lateral movement. 

**Identifying External and Internal Scanning (Lab Tasks)**
Q1: Which file contains logs that showcase internal scanning activity?
1. Navigate to the directory where the log file is located by using *cd /home/ubuntu/Downloads/logs*
2. Use the *ls* command to list the files in the directory.  We noticed there are 3 log csv files.
3. We were told in the reading that an internal IP of a target is likely to be a private one, and private IPs usually start with a 192.168…. and are non-routable addresses, meaning they are only used for internal communications within a local network (like your home Wi-Fi) and are not accessible directly from the public internet.
4. We use the command *head -n2 “log csv file”* for each log file to verify which one has a private IP
<img width="801" height="213" alt="image" src="https://github.com/user-attachments/assets/66f783cb-5b87-4acb-a292-964f6a96c85a" />

Q2: How many log entries are present for the internal IP performing internal scanning activity?
1. Since we already know which file is from the question above, we will have to use the command *cat log-session-2.csv |cut -d “,” -f3 |uniq -c*.
2. Cut command with a -d “,” telling the cut command to use the comma as the delimiter. -f3 also tells the cut command to extract only the 3rd field.  uniq -c is saying remove or detects duplicate lines.
 <img width="798" height="124" alt="image" src="https://github.com/user-attachments/assets/c0981f2c-ea55-4e3c-8837-c223fffa963a" />

Q3: What is the external IP address that is performing external scanning activity?
1. If we return to the directory from question one and examine the other log files, we will notice a different source IP scanning our destination IP, which we identified in the log file from the first question. This IP is 192.168….
<img width="805" height="236" alt="image" src="https://github.com/user-attachments/assets/83d98a3c-8d50-4816-9d15-c76a772059bf" />


### Task 4: Horizontal vs Vertical Scanning
Once the attackers know what hosts are present on a network, they often want to identify what ports are open on these hosts. This is called a port scan, and can be of the following types:

**Horizontal Scanning**
* Horizontal scans are performed to identify which hosts expose a particular port.
* We can detect a horizontal scan in the logs if we see the same source IP, a single destination port, but multiple destination IP addresses across multiple events.

**Vertical Scanning**
* Vertical scanning occurs when a single host IP address is scanned across multiple ports. Attackers perform vertical scanning to footprint a host and identify its exposed services.
* We can detect a vertical scan in the logs if we observe the same source IP, the same destination IP, but multiple destination ports across multiple events.

**Lab Tasks**
Q1: One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X
1. Upon reviewing all the logfiles, the one that indicates a possible horizontal scanning is log.session-2.csv.
2. To have a view of all the log files, we will send just the 3 to the 6 field to the csv2.txt file using the command cut -d’,’ -f3,4,5,6 log-session-2.csv > csv2.txt, and we read the text file with the cat csv2.txt.
<img width="684" height="477" alt="image" src="https://github.com/user-attachments/assets/4332673c-40c7-44cb-982d-52d9b0231f13" />

Q2: In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?
1. From what we know about vertical scan. Still referring to the above output, scroll up and retrieve the IP.
   
Q3: On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.
1. Examining the head of the output log file, we can see the various ports it is using.
<img width="679" height="359" alt="image" src="https://github.com/user-attachments/assets/1134e5ea-da5b-46b2-9ccb-091db6fd663c" />

### The Mechanics of Scanning

**1. Ping Sweep**
* Ping sweeps are generally used to identify hosts present (and online) on a network.
* This scan is run by sending an Internet Control Message Protocol (ICMP) packet to the host.
*  If the host is online, it will reply with an ICMP packet of its own. However, it is often blocked by security controls in some organisations nowadays, making it easier to defeat this type of scanning activity.

**2. TCP SYN Scans**
* A TCP connection is initiated by a three-way handshake, following the steps SYN, SYN-ACK, ACK to establish the connection.
* Network scanners can sometimes use this functionality of the TCP handshake to identify online hosts and their open ports.
* The scanner sends a SYN request to the recipient. If a SYN-ACK response is received, it means that the host is online and the port on which the SYN connection was sent is also open.

**3. UDP Scan**
*  If the port is closed, the host sends back an ICMP port unreachable reply. This signifies that the port is closed, but the host is online.
*   In some cases, the scanner will not receive any response until a set timer is reached. If no response is received, the scanner will mark the port as open (but this is not a clear evidence of the port being open).
*   In rare cases, the scanner might receive a UDP packet in response, which is evidence of the port being open.

**Identifying Scan Types (Lab Tasks)**
Q1: Which source IP performs a ping sweep attack across a whole subnet?
1. From the Kibana page showing the logs, you will need to add the source IP and network protocol as columns and then filter out TCP to get only ICMP connections, as that’s what a ping sweep uses.
<img width="780" height="567" alt="image" src="https://github.com/user-attachments/assets/b025f073-14ca-4084-9841-5eeb1f27ea3c" />

Q2: The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145
1. Adding that given field as a column, we can see the connection saying TCP, which definitely tells us the type of scan.
<img width="855" height="574" alt="image" src="https://github.com/user-attachments/assets/c72a5b1e-f3ca-4727-8ed9-5683b1294f35" />

Q3: Is there any UDP scanning attempt in the logs? Y/N
1. This question builds on the previous one, meaning we are examining those two IP addresses. Scrolling all the way down, we cannot find any UDP connection.




