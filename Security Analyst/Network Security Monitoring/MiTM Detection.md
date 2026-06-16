### Task 1: Introduction
Man-in-the-middle (MITM) attacks represent one of the most insidious threats in network security. In these attacks, attackers position themselves between legitimate communication endpoints to intercept, modify, or redirect traffic. From a blue team perspective, detecting these attacks requires a multi-layered approach combining network monitoring, certificate validation, and behavioral analysis. 

### Task 2: Lab Connection

### Task 3: MITM Attacks Overview
A Man-in-the-Middle (MITM) attack is a cyberattack where an attacker secretly intercepts and potentially alters communication between two parties, such as a user and a service, without their knowledge. The attacker may eavesdrop to steal sensitive data like credentials and credit card info or inject malicious content. These attacks can target any organization or individual, especially where encryption or authentication is weak.

**How MITM Attack Work**\
* Interception: The attacker inserts themselves into a communication stream, often by exploiting weaknesses in network protocols or by using techniques like ARP, DNS, or IP spoofing.
* Manipulation/Decryption: The attacker tries to access or modify the communication, decrypting encoded data or injecting harmful content, such as altered website responses or fake login forms

**Commom Types of MITM Attacks**
* Packet sniffing: Capturing unencrypted data packets exchanged over a network, often on open Wi-Fi.
* Session hijacking: Stealing and using session tokens to impersonate users.
* SSL stripping: Downgrading HTTPS connections to insecure HTTP to steal or alter data transferred.
* DNS spoofing: Redirecting legitimate website traffic to fraudulent domains by manipulating DNS responses.
* IP spoofing: Crafting malicious IP packets that appear to come from trusted systems.
* Rogue Wi-Fi access point: Creating fake networks to intercept user traffic.

**MITM and Cyber Kill Chain**
* Reconnaissance: The adversary gathers intelligence on the target to identify vulnerabilities.
* Weaponization: The adversary couples an exploit with a malicious payload into a deliverable package.
* Delivery: The adversary transmits the weaponized package to the targeted environment.
* Exploitation: The adversary's code is triggered, and it takes advantage of a software, hardware, or human vulnerability to gain initial access.
* Installation: The adversary installs malware or establishes a persistent backdoor on the compromised asset.
* Command & Control (C2): The adversary establishes a covert channel to communicate with and control the compromised asset.
* Actions on Objectives: The adversary executes their ultimate goals, such as data exfiltration, destruction, or lateral movement.

### Task 4: Detecting ARP Spoofing
**What is ARP Protocol**\
ARP (Address Resolution Protocol) maps IP addresses to MAC addresses in a local network. When a device wants to send data to another IP, it first asks: "Who has this IP?” The correct device replies with its MAC address.

**ARP Spoofing**\
In ARP spoofing, an attacker sends fake ARP replies to trick devices into associating the attacker’s MAC address with a legitimate IP, usually the default gateway. This allows the attacker to intercept, modify, or redirect traffic.

**Why ARP Spoofing Works**\
ARP has no authentication. Any device can send unsolicited is-at messages. An attacker exploits this vulnerability by sending fake ARP replies to victims and gateways:
* 192.16.10.100 is at 02:fe:BB:cd:55:55 attacker claims to be the gateway.
Results:
* The Victim's ARP cache becomes poisoned.
* All traffic intended for the gateway flows through the attacker first (MITM).

**Indicators of the Attack**
* Duplicate MAC-to-IP Mappings: Multiple MAC addresses claiming the same IP address. Indicates impersonation.
* Unsolicited ARP Replies: High number of ARP replies without matching requests ("gratuitous ARP").
* Abnormal ARP Traffic Volume: A Large number of ARP packets in short intervals.
* Unusual Traffic Routing: Traffic rerouted through the attacker’s MAC.
* Gateway Redirection Patterns: Multiple destination MACs for the same gateway IP.
* ARP Probe / Reply Loops: Many ARP requests with Who has 192.168.1.x? Tell 192.168.1.y patterns.

**Netowork Traffic Analysis (LAB)**\
1. Lets begin examine the ARP traffic and seeing how can we identify and detect ARP spoofing in the network traffic.\
As the ARP is the protocol of interest, we can start by isolating it using the following filter. This will allow us to inspect requests/replies and notice abnormal volume or patterns. 
* Filter: *arp*
2. Let's look at the ARP requests.
* Filter: *arp.opcode == 1*
3. Forged ARP poisoning typically uses unsolicited is-at replies (gratuitous/unasked replies). These are strong indicators. Let's look at the ARP response, as shown below:
* Filter: *arp.opcode == 2*
4. A suspicious host sends many unsolicited (gratuitous) ARP replies, especially to multiple destinations. Repeated gratuitous ARPs can indicate an attacker maintaining their poison state. We can also filter on the gratuitous packets, as shown below:
* Filter: *arp.isgratutious*
5. We have the information about the IP and the MAC address associated with the gateway. Let's apply the following filter to examine the ARP traffic associated with the gateway, as shown below:
* Filter: *arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01*
6. Looking at the output, we can see only the ARP responses. Let's now narrow down on the Gateway IP to further probe about the MAC addresses associated with the IP, using the following filter as shown below:
* Filter: *arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1*
7. Looking closely, we see some ARP replies pointing the Gateway's IP to the suspicious MAC address. The frequency of these ARP replies indicates that this is indeed an ARP spoofing. Let's confirm by applying another filter, as shown below:
* Filter: *arp.opcode ==2 && _ws.col.info contains "192.168.10.1 is at"*
8. Now that we have identified ARP spoofing in action, let's narrow down on the attacker's MAC address that has associated itself with the Gateway's IP address, using the following filter:
* Filter: *arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:fe...*
9. The last thing we can check and confirm is by filtering on the Check for duplicate MAC address mappings to a single IP address. 
* Filter: *arp.duplicate-address-detected || arp.duplicate-address-frame*

### Task 5: Unmasking DNS Spoofing
**DNS Protocol Simplified**/
DNS translates human-friendly website names (like www.google.com) into computer-friendly IP addresses.
**DNS Spoofing** (or DNS Cache Poisoning) is when an attacker corrupts this system. They give your computer the wrong "phone number" for a website you're trying to visit.\

This is a powerful technique for launching a Man-in-the-Middle (MITM) attack. Here's how it works:
1. The Victim tries to visit their bank at my-real-bank.com.
2. The Attacker, who is already on the local network (perhaps via ARP spoofing), intercepts the victim's DNS query.
3. The Spoof: The attacker quickly sends a fake DNS response to the victim. This fake response says, my-real-bank.com is at my IP address: ATTACKER_IP.
4. The Interception: The victim's computer trusts this fake response and saves it in its DNS cache. When the victim tries to connect to their bank, they unknowingly connect directly to the attacker's server, which might host a perfect replica of the real banking site.
<img width="742" height="502" alt="image" src="https://github.com/user-attachments/assets/16e9af84-00c0-4b1e-b007-d72b931b3907" />

**Indicators of the Attack**
* Multiple DNS responses for the same query: A legitimate resolver and a forged responder reply to the same query. This is the single most reliable indicator.
* DNS response from an unexpected source: A DNS reply arrives from an IP address that does not match any configured resolver (like 8.8.8.8 or your DNS server).
* Suspiciously short TTL (Time-To-Live) values: Attackers use very low TTLs (1 - 30s) to keep poisoned entries short-lived and reassert control.
* Unsolicited DNS responses: A DNS reply appears without a corresponding DNS request from the victim.

**Network Traffic Analysis (LAB)**\
1. As the DNS is the protocol of interest, we can start by isolating it using the following filter. This will allow us to inspect requests/replies and notice abnormal volume or patterns. 
* Filter: *dns*

2. Legitimate DNS servers (like Google’s 8.8.8.8) respond from a known external IP address. By filtering responses from this IP, we can see what normal answers look like for comparison.
* Filter: *dns.flags.response == 1 && ip.src == 8.8.8.8*

3. Let's look at the DNS responses, using the filter below. 
* Filter: *dns.flags.response==1*
In the output, we can hunt for responses from IP addresses other than the usual DNS server. Looking closely, we find one DNS response from an IP other than 8.8.8.8. This is an interesting find. Though we have discovered an odd DNS request indicating a potential DNS spoofing attempt, we will note this and return to it later.

4. The following filter will show the DNS responses from the DNS server. 
* Filter: *dns.flags.response == 1 && ip.src == 8.8.8.8*

 5. Let's now take a curious look at the DNS traffic for our domain of interest, corp-login.acme-corp.local, using the following filter:
* Filter: *dns && dns.qry.name == "corp-login.acme-corp.local"*

6. Let's now filter on the DNS request for our domain of interest, originating from the DNS server, using the filter, as shown below:
* Filter: *dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"*

7. In the previous results, we explored the DNS traffic originating from the DNS servers for our domain of interest. Let's apply the following filter to see if any DNS requests are coming from servers other than the DNS server 8.8.8.8:
* Filter: *dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"*

### Task 6: Spotting SSL Stripping in Action
SSL stripping is a man-in-the-middle technique in which an attacker intercepts and modifies traffic to remove or prevent TLS encryption between a client and a server. This causes the client to communicate over HTTP instead of HTTPS. The attacker retains a secure (HTTPS) session with the server while relaying plain HTTP to the victim, enabling eavesdropping and credential capture.

**How It Works**
1. The victim initiates an HTTPS request to a website.
2. The attacker intercepts the request using ARP spoofing or a rogue access point.
3. The attacker connects to the website over HTTPS but relays the response to the victim through HTTP.
4. The victim unknowingly interacts over HTTP, exposing sensitive data in plaintext.

**Indicators of SSL stripping**
* Initial Request vs. Response: The user's initial request may be for HTTPS (port 443), but the subsequent packets immediately shift to unencrypted HTTP (port 80) for the same domain.
* Redirects/Link Rewriting: Monitoring for redirects (HTTP Status Codes 301, 302) that persistently direct an HTTPS request to an HTTP resource.
* Certificate Errors: Although the attacker usually tries to hide this, the initial TLS/SSL Handshake may fail or display a self-signed certificate if the attacker uses a more direct proxying technique.

**Network Traffic Analysis (LAB)**
1. For this task, HTTPS is the protocol of interest. We start by isolating it using the following filter. This will allow us to inspect SSL requests and notice abnormal volumes or patterns. 
* Filter: *tls || ssl*

2. Let's apply the following filter to show the established TLS handshakes to our server, which proves the site normally uses TLS for communication.
*Filter: *tls.handshake.type == 1 && tls.handshake.extensions_server_name == "corp-login.acme-corp.local"*

3. Our hypothesis about SSL stripping is that it is done after the attacker has successfully performed DNS spoofing, sending the victim to the attacker's IP. In the previous task, we identified the attacker's IP address performing the DNS spoofing. Let's isolate DNS responses from the attacker to show the victim was pointed to the attacker's IP, as shown below:
* Filter: *dns.flags.response == 1 && ip.src == 192.168.10.55 && dns.qry.name == "corp-login.acme-corp.local"*
* The result shows two incidents in which the attacker's IP sent spoofed DNS responses for the domain of interest.

4. One of the main indicators of SSL stripping is that, after the spoof, the domain no longer performs TLS handshakes to the legitimate server, which validates the absence of TLS traffic from the victim to the real server. Let's apply the following filter on the server IP and the attacker IP, as shown below:
* Filter: *http && ip.src == 192.168.10.10 && ip.dst == 192.168.10.55*
* Click the packet -> Follow -> HTTP Stream
