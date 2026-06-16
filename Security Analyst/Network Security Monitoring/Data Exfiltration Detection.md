### Task 1: Introduction
Data exfiltration is the unauthorized transfer of sensitive data from a computer or other device. It's a primary objective for attackers who have breached a network. As a SOC analyst, our job is to detect and stop this before sensitive information walks out the door.
In this room, i learnt the common methods used for data exfiltration, how to detect them using network traffic analysis, identify signs of exfiltration and correlate logs in SIEM.

### Task 2: Lab Connection

### Task 3: Data Exfil: Overview, Technique and indicators
**Techniques and Indicators**
Detecting exfiltration requires correlating host and network-level indicators such as unusually large or frequent outbound uploads (proxy/firewall), long or high-entropy DNS queries, suspicious process command-lines and network connections (Sysmon/EDR), cloud storage API activity, removable-media events, and effective SOC L1 triage focuses on source host/user, destination, transferred volume, process identity/command-line, and supporting evidence across proxy, DNS, flow, host, and cloud logs.
<img width="1240" height="635" alt="image" src="https://github.com/user-attachments/assets/87bfa7c7-99fe-49e6-b9e5-63c7fbe8b94c" />


### Task 4: Detection: Data Exfil through DNS Tunneling
DNS exfiltration abuses the DNS, a protocal normally allowed through networks to smuggle bytes encoded inside DNS queries/responses so firewalls and web proxies don't notic.
DNS typically allowed and often unfiltered or forwarded to public resolvers.

**DNS Tunneling**
DNS (Domain Name System) translates human-friendly domain names (e.g., example.com) into IP addresses and provides other record types (A, AAAA, TXT, MX, CNAME, etc.). 
Key points:
* DNS queries are ubiquitous: almost every host performs DNS lookups.
* DNS is normally allowed through firewalls and gateways, making it an attractive covert channel.
* DNS uses UDP (mostly) on port 53 for queries and responses; TCP is used for zone transfers or large responses.

Why attackers use DNS for exfiltration:
* Always-on service: DNS lookups are routine and often allowed outbound.
* High cover: queries look like normal requests unless inspected closely.
* Flexible payload: data can be encoded into the subdomain labels or TXT responses.

**Indicators of attack**
When analysing DNS traffic for possible indicators of data exfiltration, we should look for:
* Many DNS queries are sent to a single external domain, especially with very high counts compared to the baseline.
* Long subdomain labels or unusually long full query names (> 60–100 characters).
* High entropy or Base32/Base64-like patterns in the query name (lots of mixed case letters, digits, -, = signs for base64).
* Rare record types (TXT, NULL) or many large TXT responses.
* Unusual response behavior: frequent NXDOMAIN (if attacker uses exfil-by-query without answering), or TCP/large UDP fragments for DNS.
* Queries at regular intervals (beaconing behaviour).

**Detecting through wireshark**
Let's start by applying filter on dns traffic, using the following filter, as shown below:
* Filter: *dns*
Let's filter on the DNS queries with no response using the following filter:
* Filter: *dns.flags.response == 0*
Find log queries (suspicious subdomain lenghts)
* Filter: *dns && frame.len > 70*
Let's filter on the suspicious domain using the following filter:
* Filter: *dns && dns.qry.name contains "the domain name"*

**Investigating with Splunk**
Search Query: *index=data_exfil sourcetype=DNS_logs*
This search query will filter show the results matching sourcetype=DNS_logs. What to look for

Let's run the following search query to display the stats of DNS queries generated per source IP, as shown below:
* Search Query: *index="data_exfil" sourcetype="DNS_logs" | stats count by src_ip*
Let's now apply filter on the stats of the queries to identify the suspicious ones, as shown below:
* Search Query: *index="data_exfil" sourcetype="dns_logs" | stats count by query | sort -count*

**Long Query Names (subdomain encoding)**
Let's now apply the following filter on the queries with length over 30 and see if we can filter the odd-looking ones out, as shown below:
* Search Query: *index="data_exfil" sourcetype="DNS_logs" | where len(query) > 30*
With the resulted indicators, we were able to identify the data exfiltration attempts through DNS tunneling:
A Large number of DNS requests with no response.
Large length of the DNS query.

### Task 5: Detection: Data Exfil through FTP
FTP (File Transfer Protocol) is one of the oldest protocols for transferring files between a client and server over a TCP/IP network. Attackers use it to move large amounts of data off a network, sometimes via compromised credentials, misconfigured servers, or ephemeral accounts. Detection relies on a mix of packet inspection (FTP only), server logs, SSH session metadata, and network flow/size/pattern analysis.

**How adversaries use FTP for exfiltration**
* Use legitimate FTP servers (public or misconfigured internal servers) to stage/transfer data.
* Use compromised credentials (service accounts, user creds).
* Use non-standard ports or tunneling to blend with other traffic.

**Indicators of attack**
What to look for:
* USER and PASS commands (cleartext credentials).
* STOR (upload) and RETR (download) commands: repeated or large transfers.
* Large data connections to unusual external IPs, especially outside business hours.
* Data channel openings on ephemeral ports (PASV) paired with large payloads.

**Examining the FTP exfil file**
Isolate FTP control & data
* First, we will look for FTP Sessions using the *ftp || ftp-data filter*
Look for credentials
* Filter to show only login attempts with USER/PASS:*ftp.request.command == "USER" || ftp.request.command == "PASS"*
From the output, we can look for suspicious usernames or weak passwords.
Look for Anomalies in Filenames or Credentials
* Filter: *ftp contains "STOR"*
* Right-click a *packet* → *Follow* → *TCP Stream*
We can look at suspicious files by filtering on the file extensions like PDF, csv, TXT etc. Let's apply a filter on ftp packets containing the term csv in it, as shown below:
* Filter: *ftp contains "csv"*
The results show a suspicious IP connected as Guest account has transferred some sensitive csv files to a supicious external IP.

**Identifying the traffic with large payload size**
Look at the traffic with a large length.
* Filter *ftp && frame.len > 90* and check out the content in TCP Stream

**Question**
Q1: How many connections were observed from the guest account?
1. Filter: *_ws.col.info == "Request: USER guest"*

Q2: Apply the filter; what is the name of the customer-related file exfiltrated from the root account?
1. Filter: packet -> Follow --> TCP Stream

Q3: Which internal IP was found to be sending the largest payload to an external IP?
1. Analyze the IP payload for each

Q4: What is the flag hidden inside the ftp stream transferring the CSV file to the suspicious IP?
1. Go through the packet -> FTP -> DATA


### Task 6: Detection: Data Exfil via HTTP
Data exfiltration via HTTP is when an attacker moves sensitive data out of a target network using HTTP as the transport. HTTP is commonly abused because it blends with normal web traffic, can traverse firewalls and proxies, and can be obfuscated (encoding, encryption, tunneling).

**Why it matter**
* HTTP is very common; attackers hide exfiltration in the noise of legitimate web usage.
* Successful detection stops data breaches and helps trace attacker activity post-compromise.
* Organizations must detect and respond to protect sensitive data and meet compliance requirements.

**How adversaries use HTTP for data exfiltration**
* POST uploads to external servers: Bulk data is sent to attacker-controlled hosts or cloud storage in POST request bodies.
* GET requests with encoded data: Attacker squeezes small chunks into query strings or path segments (useful for low-and-slow exfiltration).
* Use of common services / CDN: Exfiltration disguised as uploads to popular services or attacker-controlled subdomains under reputable domains.
* Custom headers: Data placed in headers (e.g., X-Data: <base64>) may bypass some string-based DLP.
* Chunked transfer / multipart: Large payloads split into multiple requests to avoid size thresholds.
* HTTPS/TLS tunneling: The encrypted channel hides the payload; detection requires TLS inspection, SNI analysis, or metadata-based detection.
* Staging via cloud services: The attacker uploads to Dropbox/GitHub/Gist and then fetches externally.

**Indicators of Attack (IoAs)**
* Unusually large HTTP POST requests to external/unexpected hosts.
* HTTP requests to domains with low reputation / rarely seen in baseline traffic.
* Frequent small requests (beaconing) to the same host, followed by large uploads.
* Chunked or multipart transfers where multiple requests compose a larger file.

**Analyze Logs in Splunk**
To start, use the following search query to get the http_logs and make sure to select Time range as All Time, as shown below:
* Search Query: *index="data_exfil" sourcetype="http_logs"*
As we understand, the exfiltration attempts can be done via POST requests. We will apply a filter on the POST method to further narrow down our results, as shown below:
* Search Query: *index="data_exfil" sourcetype="http_logs" method=POST*
Let's now look at the average amount of bytes sent out to various domains, as shown below:
* Search Query: *index="data_exfil" sourcetype="http_logs" method=POST | stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain | sort - count*
Let's filter out benign traffic and isolate POSTs with large payloads.
* Search Query: index="data_exfil" sourcetype="http_logs" method=POST bytes_sent > 600 | table _time src_ip uri domain dst_ip bytes_sent | sort - bytes_sent
Our analysis so far points to one suspicious entry, with a large chunk of data uploaded to an external source. Let's correlate this with the pcap to extend our analysis further.

**Analyze in Wireshark**
Apply a filter on the HTTP traffic, as shown below:
* Filter: *http*
We can see HTTP traffic with both GET and POST requests. Filter on POST requests, as shown below:
* Filter: *http.request.method == "POST*
Filter on the traffic with the frame length of more than 500, as shown below:
* Filter: *http.request.method == "POST" and frame.len > 500*
We still have a lot of noise. Let's increase the size to 750 and see if we can further filter down the result.
* Filter: *http.request.method == "POST" and frame.len > 750*
This request shows only one entry, which looks the same as the one we found in Splunk. Let's go to the HTTP stream to see the file's content, as shown below:

### Task 7: Detection: Data Exfiltration via ICMP
ICMP is a network-layer protocol used for diagnostics and control (e.g., ping, TTL exceeded). Because it is commonly allowed through firewalls and typically inspected less strictly than TCP/UDP, attackers sometimes abuse ICMP to tunnel and exfiltrate data.

**Common Techniques**
* ICMP echo (type 8) / reply (type 0) tunneling: attackers place encoded (base64, hex) chunks of files inside ICMP payloads. The remote server collects and decodes them.
* Custom ICMP types/codes: using uncommon ICMP types or non-zero codes to avoid signature-based detections.
* Fragmentation and reassembly: large payloads are split across multiple packets.
* Encryption/obfuscation: Encrypting or encrypting payloads (base64 is common) to look like random data.

**Indicators**
* Persistent ICMP sessions to an external host not used for legitimate monitoring.
* Unusually large ICMP payloads or frequent ICMP with payload > typical ping size.
* ICMP payloads that contain high-entropy data or patterns consistent with base64/hex.
* Bursts of ICMP are immediately followed by no other legitimate application traffic from the same host.

**Indicators of attack in Wireshark**
Look for the following in a pcap when inspecting in Wireshark:
* ICMP packet volumes: a single host sending many ICMP echo requests to an external IP.
* Large frame.len or icmp.payload: pings with payloads much larger than typical (e.g., > 64 bytes).
* ICMP type/code unusual values: e.g., unusual use of timestamp(13/14) or custom codes.
* Regular timing (periodicity): evenly spaced ICMP packets carrying similar-sized payloads.
* Fragments with reassembly: multiple ICMP fragments from the same src/dst pair.

**Lab Task**
**Filter All ICMP Traffic**
The filter below isolates all ICMP packets. Look for unusually frequent or large ICMP Echo Requests/Replies.
* Filter: *icmp*

**Isolate Echo Requests**
Let's apply the filter to isolate ICMP Echo Request packets, as shown below:
* Filter: *icmp.type == 8*

**Examine Large ICMP Packets**
Let's now apply the filter on the ICMP requests and focus on the frame length over 100, as shown below:
* Filter: *icmp.type == 8 and frame.len > 100*
Flags packets with unusually large payloads. Normal pings are ~74 bytes total. Anything over 100 is suspicious.




  









