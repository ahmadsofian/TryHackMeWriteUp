### Task: A Series of Suspicious Email
As a member of the IT department at SwiftSpend Financial, you are responsible for assisting employees with technical concerns. What initially appeared to be a routine day quickly escalated when multiple employees across different departments reported receiving a suspicious email. Several users noted unusual characteristics in the message, and unfortunately, some had already submitted their credentials and were no longer able to access their accounts. With the potential for a wider compromise, the incident has been escalated for investigation. 
The task is to analyze the available evidence, determine the scope of the attack, and uncover how the adversary operated.

**Q1: Which individual received the email regarding a Quote for Services Rendered?**
1. Look for it at sender and recipient details.

**Q2: What email address was used by the adversary to send the phishing emails?**
1. Look for it at the email content.

**Q3: Investigate the attachment in the email addressed to Zoe Duncan. What is the root domain of the redirection URL found within the file?**
1. Open the email addressed to Zoe Duncan.
2. The domain URL is stored in the attached HTML file.
3. Open the html file  via FireFox and inspects the content.

**Q4: Which company is the login page impersonating?**
1. Open the html file  via FireFox and inspects the content.

**Q5: Navigate to the /data directory. What is the name of the archive file?**
1. Navigate to the "domain name"/data directories.

**Q6: Download the phishing kit archive to your virtual environment. Using the sha256sum command, what is the SHA256 hash of the file?**
1. Perform command line as below:
2. <img width="721" height="180" alt="image" src="https://github.com/user-attachments/assets/f3a864db-c0c2-4009-a014-d9e25cdb7459" />

**Q7: Aside from phishing, what other threat category is assigned to the ZIP archive?**
1. Copy and paste the hash into the ViruTotal.
2. Look at the Detection section to find the threat categories.

**Q8: Review the VirusTotal Details page for the phishing kit. How many files are contained within the archive?**
1. Look it up at the Details section then Bundle Info.

**Q9: Let’s see if the attacker has exposed any captured credentials. Navigate to the /data/Update365/ directory and investigate the log file.What is the email address of the user who submitted their credentials more than once?**
1. Naviagate to the /data/Update365 directories.
2. Investigate the log.txt file.

**Q10: Extract the phishing kit archive and locate the submit.php file. What email address is used by the adversary to collect compromised credentials?**
1. Unzip the phishing kit and find out which file responsible for sending information to the adversary.
2. Open the submit.php file and navigate through it to find the email address.

**Q11: Return to the phishing URL and locate the flag.txt file. Using CyberChef (opens in new tab) to decode the flag, what is the secret value?**
1. Navigate to http://kennaroads.buzz/data/Update365/office365/flag.txt
2. Decode the flag in cyberchef to get the flag.
