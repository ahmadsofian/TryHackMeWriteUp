### Task: A Message That Doesn't Add Up
A sales executive at Greenholt PLC has reported a suspicious email received from a known customer. The message raised several red flags: a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment. According to the employee, this behavior does not align with the customer’s usual communication style. Concerned that the email may be malicious, the message has been escalated to the SOC (Security Operations Center) for further investigation.
The goal is to analyze the provided email sample and determine whether it is legitimate or part of a phishing attempt.

**Q1: What is the Transfer Reference Number listed in the email's Subject line?**
1. The Transfer Reference Number located at the details include in the email.\

**Q2: What is the display name of the sender?**
1. Included in the email.\

**Q3: What is the sender's email address?**
1. Included in the email.\

**Q4: What email address will receive a reply to this email?**
1. Included in the email.\

**Q5: What is the originating IP address of this email**
1. Open View then message source to get into the Thunderbird
2. The originating IP can be found at the line "Received:"

**Q6: Who is the owner of the originating IP?**
1. Perform IP lookup or WHOIS to find the organization correspond to the IP address.
<img width="873" height="773" alt="image" src="https://github.com/user-attachments/assets/3d778489-f1c0-4d09-8cd8-0380ce709566" />\

**Q7: What is the full SPF record for this domain?**
1. Use mxtoolbox and copied the email header and pasted in for analysis.
2. Able to get the SPF information in easy-to-read format.
3. DMARC needed for this task.

**Q8: What is the complete DMARC record for this domain?**
1. The result from steps above included DMARC too.

**Q9: What is the file name of the attachment found in the email?**
1. Filename can be found in the email itself and the Thunderbird.

**Q10: Using the sha256sum command, what is the SHA256 hash of the file?**
1. Run the command: *sha256sum "file name"*

**Q11: What is the attachment's file size in KB (e.g., 122.31 KB)?**
1. Copied the hash value and look it up at VirusTotal and found the size.
<img width="927" height="505" alt="image" src="https://github.com/user-attachments/assets/a91b6dc4-5f73-47ca-a98b-d0458b5cfc49" />\
**Q12: What is the actual file type of the attachment?**
1. File type is also in the above question  details section.
