### Email Addresss
1. **Username:** User mailbox that identifies the specific recipient’s mailbox on the email server
2. **@ symbol:** Separates the username from the domain and tells the system where to route the email
3. **Domain name:** Specifies the mail server responsible for receiving the message

### Email Delivery
1. **Simple Mail Transfer Protocol (SMTP):** Sends emails
2. **Post Office Protocol (POP3):** Downloads emails to a device
3. **Internet Message Access Protocol (IMAP):** Syncs emails across devices

**POP3**
* Email are downloaded and stored on a single device.
* Sent messages are stored on the single device from which the email was sent.
* Emails can only be accessed from the single device to which the emails were sent
* Emails are typically removed from the server after download

**IMAP**
* Emails are stored on the server and can be downloaded to multiple devices
* Sent messages are stored on the server
* Syncs messages across multiple devices
* Emails remain on the server unless explicitly deleted

**Understanding the email protocol**
1. User sends an email: The sender’s email client sends the message to their mail server using SMTP
2. Mail server queries DNS: The sending server asks DNS for the recipient domain’s mail server
3. DNS responds: DNS returns the address of the recipient’s mail server
4. Email is delivered: The message is sent across the Internet to the recipient’s server
5. The recipient checks their mailbox: The recipient’s email client connects to their mail server
6. Email is retrieved: The message is downloaded (POP3) or synced (IMAP) to the recipient’s device

### Email Headers
* Analyze the phishing email sample headers by navigating to the *View* then *Message Source*. Got to identify the sender IP Address and the Email Header Details.

### Email Body
**Viewing HTML Source Code**
* Analyze the raw html from the sample email by using the Thunderbird *Message Source*.

**Reconstructing Attachments**
* Content-Type indicates the file type application/pdf.
* Content-Disposition specifies that the file is an attachment and includes its filename.
* Content-Transfer-Encoding shows that the file is base64 encoded.
* Challenge --> Analyze the sample email2, decode the email by using base64 string using CyberChef. Make sure to save the output as a file to view the flag.

### Types of Phising
* **Spam**: Unsolicited bulk emails sent to a large number of recipients. A more malicious form of spam is often called malspam.
* **Phishing**:Emails that impersonate a trusted entity to trick recipients into revealing sensitive information.
* **Spear Phishing**:A targeted form of phishing aimed at a specific individual or organization, often using personalized information.
* **Whaling**:A type of spear phishing that specifically targets high-level executives (CEO, CFO) to obtain sensitive data or financial access.
* **Smishing**:Phishing attacks conducted via SMS or text messages, targeting users on mobile devices.
* **Vishing**:Phishing attacks carried out through voice calls, where attackers use social engineering over the phone.

**Anatomy of Phishing Email**
* Spoofed From Address: The sender’s email is spoofed to appear as a trusted entity (noreply@microsof.com)
* Urgent Subject or Message: The email creates a sense of urgency (“Your account will be locked in 24 hours”)
* Brand Impersonation: The email is designed to mimic a legitimate organization (logos or colors matching a real company)
* Grammar & Spelling Issues: The message may contain errors, though with AI these are now less common (awkward phrasing or unnatural wording)
* Generic Content: The message lacks personalization (“Dear Customer” instead of your name)
* Hidden or Shortened Links: Hyperlinks may disguise their true destination (bit.ly/secure-login)
* Malicious Attachments: Attachments are included and disguised as legitimate files (invoice.pdf.exe)

**Analyzing sample email3.eml**
1. Idenitfy the organization and its sender's email address.
2. View the message source from Thunderbird to idenfity the originatig IP.
3. Defang the originated IP Address using CyberChef.
4. From the message source we can identify the authentication results header to find the mail server that generated the email.
