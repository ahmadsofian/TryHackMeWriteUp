### Task 1: Introduction
In this room i learned to identify common social engineering tactics used in phishing, anakyze red flags, detect link manipulation & tracking pixels and deconstruct credential harvesting and attachment manipulation.  


### Task 2: Cancel Your Order
In this, i examined sample email designed to mimic official transaction from PayPal.

**Phishing Techniques Used**
* Spoofed email address
* URL shortening
* Branded HTML

**First Observation**
1. The subject line uses a fake transaction to create a sense of urgency, prompting you to react in haste.
2. The sender details service@paypal.com do not match the actual address gibberish@sultanbogor.com.
3. This is an unusual email recipient address and not a normal Yahoo domain.

**Email Body Analysis**
1. We can see that this is a receipt for a purchase of gift cards. The email is designed to appear as a legitimate email from PayPal.
2. There aren't any attachments associated with this email, and the only interactive element in this email is the Cancel the order button.

**Button Investigation**
1. By inspecting the raw source of the email, we can further investigate the hyperlinks and underlying HTML that form the message.
2. The Cancel the order button leads to the shortened URL.
3. As a rule of thumb, we should never interact with buttons or links without first confirming exactly where they lead.


### Task 3: Track Your Package
We will investigate mimics a formal shipping notification to trick recipients into a sense of urgency.

**Phishing Technique Used**
* Spoofed email address
* Pixel tracking
* Link manipulation

**First Observation**
1. The subject line uses a fake tracking number to create a sense of urgency, prompting the recipient to click to see their package status.
2. This is an immediate red flag, as the display name Distribution Center does not match the actual sender address contact@beginpro.club.
3. The link in the email body matches the subject line appears to be a hyperlink.

**Hyperlink Tracking**
1.  Checked out the source of the email message to get a closer look at the tracking number hyperlink.
2.  There is an image file named *Tracking.png*. These trackers send information back to the spammer's server.
3.  There we can finde the hyperlink destination and its domain name.


### Task 4: Download Document Here
We will analyze a phishing campaign(opens in new tab) that utilizes a multi-stage redirection chain to harvest user credentials.

**Phishing Techniques Used**
* Artificial urgency
* Brand impersonation
* Link redirection
* Credential harvesting

**First Observation**
1.  The email was sent on Thursday, July 15th, 2021.
2.  A sense of urgency is introduced in this email. Notice that the link to download the fax document expires on the same day.
3.   There is an action to perform. In this case, a button to download the fax.

**Download the Document Here**
1. After clicking the Download Document Here button, the user is redirected to a landing page that mimics a legitimate OneDrive share.
2.  The URL is highly suspicious, and the provided directions are nonsensical.
3.  These inconsistencies lead to a credential-harvesting portal that asks the victim to sign in with their email provider to view the document.

**Logging In**
1. The victim attempts to log in using Outlook.
2. Even if they had entered valid credentials, the result would be the same: a generic error message.
3. The page isn't actually authenticating the user to their mail service; it's simply a front used to send the credentials directly to the attacker's server.


### Task 5: Your Account is on Hold
We will again examine an email claiming to be from a trusted, household brand that demands immediate action from the recipient.

**Phishing Techniques Used**
* Spoofed email address
* Sense of urgency
* Brand impersonation
* Poor grammar and typos
* Attachments

**First Observations**
1. The receiver's ID was suspended and must act quickly.
2. The display name does not match the user and domain.
3. The email utilizes rendered HTML to impersonate Netflix.

**Email Body and Attachment Analysis**
1. We can see that the user is informed that there is an issue with their billing information.
2. In order to update their account, they must open the attached PDF file.
3. The attachment contains an embedded link titled Update Payment Account which directs to a URL not associated with a legitimate Netflix domain.

*note*
* The use of an atypical phone number format is a red flag.
* Using a legitimate Netflix help center domain to build a false sense of trust.


### Task 6: You Recent Purchase
We will analyze a phishing attempt that masquerades as a billing notification from a major service provider.

**Phishing Techniques Used**
* Spoofed email address
* Recipient is BCCed
* Urgency
* Poor grammar and typos
* Attachments

**First Observations**
1. The recipient is told they must act quickly to resolve an unauthorized purchase, creating a false sense of urgency.
2. The display name, Apple Support, does not match the user and domain. There are also typos in the From and To addresses.
3. The recipient was not directly emailed, instead Blind Carbon Copied(opens in new tab) (BCC).

**Analyzing the Attachment**
1. The body of the email is completely blank, which is suspicious.
2. The only content present is an attachment in the form of a .dot file.
3. When the user interacts with the large image embedded in the document, they are redirected to a phishing site.
4. Although the URL includes familiar terms like apps and ios to appear legitimate, its excessive length and complexity are strong indicators of a malicious redirection attempt.

### Task 7: Scheduled Shipment
We analyze a phishing attempt that masquerades as a global shipping notification using spoofed addresses and branded HTML to appear legitimate.

**Phishing Techniques Used**
* Spoofed email address
* Brand impersonation
* Attachments

**First Observations**
* Gives the impression that DHL will be shipping a package.
* The display name, DHL Express, does not match the user and domain.
* The HTML in the email body is designed to look like it is sent from DHL.

**Email Body and Attachment**
1. There is attached .xlsx file.
2. Opening the document reveals several glaring inconsistencies.
3. The sender uses a German domain, the invoice is addressed to a city in India, yet the document content itself contains Mandarin.
4. The document contains a single clickable link designed to move the attack to the next stage.
5. When the link within the Excel document is clicked, it attempts to download and execute a malicious payload named regasms.exe.
6. This execution results in a system error.
7. While the error prevents the payload from running in this environment, it indicates the attacker’s intent to execute code directly on the victim’s machine.
8. IF successful, the attacker could:
     a) establish persistence
     b) exfiltrate data
     c) deploy ransomware
