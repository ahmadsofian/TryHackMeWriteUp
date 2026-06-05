### Task 1: Introduction
In this room, I learned the core email security frameworks controls, analyze SMTP network traffic & email content and explore anti-phishing protection measures.

### Task 2: Sender Policy Framework (SPF)
Used to authenticate the sender of an email. With an SPF record in place, Internet Service Providers can verify that a mail server is authorized to send email for a specific domain. 
An SPF record is a DNS TXT record containing a list of the IP addresses that are allowed to send email on behalf of your domain.”
<img width="824" height="412" alt="image" src="https://github.com/user-attachments/assets/8d0c3cd7-1401-4219-bd93-7fadab351941" />
spf record verification 
<img width="1240" height="266" alt="image" src="https://github.com/user-attachments/assets/e1a901e1-2886-4298-b4ce-d1257479debd" />

**SPF Records**
*v=spf1 ip4:127.0.0.1 include:_spf.google.com -all*
* *v=spf1* Signifies the start of the SPF record
* *ip4:127.0.0.1* Specifies which IP can send mail (IPv4 in this case)
* *include:_spf.google.com* Specifies which domain can send mail
* *-all* Non-authorized emails will be rejected

**Tools**
*SPF Surveyor:* tool from dmarcian enables us to gain a visual look at DNS records. It also helps ensure the record uses the correct syntax.


### Task 3: DomainKeysIdentified Mail (DKIM)
DKIM stands for DomainKeys Identified Mail and is used for the authentication of an email that’s being sent. 
Like SPF, DKIM is an open standard for email authentication that is used for DMARC alignment. A DKIM record exists in the DNS, but it is more complex than SPF. 
DKIM’s advantage is that it can survive forwarding, which makes it superior to SPF and a foundation for securing your email.”

**DKIM workflow diagram:**
<img width="847" height="453" alt="image" src="https://github.com/user-attachments/assets/111c36dd-273c-489b-b07e-3b9313f4de67" />

**DKIM Records**
*v=DKIM1; k=rsa; p=<public_key>*

* *v=DKIM1* Specifies the version of DKIM being used (optional)
* *k=rsa* The key type. The RSA encryption algorithm is standard
* *p=* This is the public key that will be matched to the private key to verify the DKIM signature


### Task 4: Domain-Based Message Authentication, Reporting and Conformance (DMARC)
“DMARC, an open source standard, uses a concept called alignment to tie the result of two other open source standards,  SPF (a published list of servers that are authorized to send email on behalf of a domain) 
and DKIM (a tamper-evident domain seal associated with a piece of email), to the content of an email.”

This means that DMARC ensures the sender's domain matches the domains verified by SPF and DKIM. If the alignment fails, 
DMARC instructs the recipient server on how to handle the email based on a policy specified in the record.

**DMARC Records**
*v=DMARC1; p=quarantine; rua=mailto:postmaster@website.com*

* *v=DMARC1:* The version of DMARC (required)
* *p=quarantine* The DMARC policy (quarantine = move to the spam folder)
* *rua=mailto:postmaster@website.com* An optional tag. In this case, aggregate reports will be sent to the email specified

### Task 5: Secure/Multipurpose Internet Mail Extensions(S/MIME)
a standard protocol for sending digitally signed and encrypted messages. It is based on public key cryptography, where the private key is never shared and the public key can be distributed openly. The two main components and security features of S/MIME are:

**Digital Signature**
Authentication, Non-repudiation and Data integrity

**Encryption**
sender encrypt messahe using recipient public key. maintain Confidentiality.

**S/MIME Flow**
<img width="804" height="278" alt="image" src="https://github.com/user-attachments/assets/8bd8a826-a7d8-4ea2-9698-7a40ee1aa1e3" />

### Task 6: Analyzing SMTP Responses
In this task, I analyzed a PCAP file with SMTP traffic.
1. Use *smtp.response.code* Wireshark filter to narrow down the results based on SMTP response codes.
2. Use *smtp.response.code eq 220* to find the total packet capture that contain SMTP response code 220 Service ready.
3. Set the filter as String to search for *spamhaus.org* and its response code.
4. The full response code for above just recite below of the response code
5. Use *smtp.response.code eq 552* to find the total blocked message for presenting potential security issues.

### Task 7: Inspecting Emails and Attachments
I utilize the Internet Message Format (IMF(opens in new tab)) to examine the inner details of emails, such as sender and recipient fields, content type, and attachments.

1. Simply enter *smtp* into the Wireshark filter to get the total SMTP packets available.
2. Press Ctrl + G to find a packet (or press Go -> Go to Packet). Enter 270 and press “Go to packet”.
3. In the Line-based text data we can find the attachment in the packet and the host IP that is not responding.
4. Filter on imf. Now, take a look at the 7 packets found. One of them mentions the attachment.scr file.
5. Just below the attachment we can identified the type of encoding used for this potentially malicious attachment.

### Task 8: How Organizations Stop Phishing
I explored some modern solutions that enhance email security and reduce phishing risks.

**Technical Defense**
* Email Filtering
* Secure Email Gateways
* Link Rewriting
* Sandboxing

**User-Facing Tools & Training**
* Trust & Warning Indicators
* Phishing Reporting
* User Awareness Training
* Phishing Simulation Exercises.


