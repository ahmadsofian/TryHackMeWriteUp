## Task 2: Client-Side Attacks
* **Client-side** attacks rely on abusing weaknesses in user behavior or on the user’s device. These attacks often exploit vulnerabilities in browsers or trick the user into performing unsafe actions to gain access to accounts and steal sensitive information.
* Valuable data can be stored on a user’s device, so a successful client-side attack can result in data loss. With the demand for dynamic and versatile web applications rising, companies are integrating more third-party plugins, broadening the attack surface in browsers, and opening more opportunities for client-side attacks.

### Common Client-Side Attacks
* **Cross-Site Scripting (XSS)** is the most common client-side attack, in which malicious scripts are run in a trusted website and executed in the user’s browser. If your website has a comment box that doesn’t filter input, an attacker could post a comment like: Hello <script>alert('You have been hacked');</script>. When visitors load the page, the script runs inside their browser, and the pop-up appears. In a real attack, instead of a harmless pop-up, the attacker could steal cookies or session data.
* **Cross-Site Request Forgery (CSRF):** The browser is tricked into sending unauthorized requests on behalf of the trusted user.
* **Clickjacking:** Attackers overlay invisible elements on top of legitimate content, making users believe they are interacting with something safe.


## Task 3: Server-Side Attacks
* **Server-side** attacks rely on exploiting vulnerabilities within a web server, the application’s code, or the backend that supports a website or a web application.
* While client-side attacks manipulate users’ interaction with a site, server-side attacks focus on taking advantage of the systems themselves.
* By exploiting flaws and vulnerabilities, server logic, misconfigurations, or input handling, attackers can gain access, steal information, and cause damage to running services.

### Common Server-Side Attacks
* **Brute-force** attacks occur when an attacker repeatedly attempts different usernames or passwords in an attempt to gain unauthorized access to an account. Automated tools are often used to send these requests quickly, allowing attackers to go through large lists of credentials and common passwords. T-Mobile faced a breach in 2021 that stemmed from a brute-force attack, allowing attackers access to the personally identifiable information (PII) of over 50 million T-Mobile customers.
* **SQL Injection (SQLi)** relies on attacking the database that sits behind a website and occurs when applications build queries through string concatenation instead of using parameterized queries, allowing attackers to alter the intended SQL command and access or manipulate data. In 2023, an SQLi vulnerability in MOVEit, a file-transfer software, was exploited, affecting over 2,700 organizations, including U.S. government agencies, the BBC, and British Airways.
* **Command Injection** is a common attack that occurs when a website takes user input and passes it to the system without checking it. Attackers can sneak in commands, making the server run them with the same permissions as the application.

## Task 4: Log-Based Detection
* Logs can be a valuable tool for detecting web attacks. Every request sent to a web server can leave evidence in its access and error logs. Defenders can spot patterns that reveal scanning, exploitation attempts, or other attacks by reviewing log entries.

### Access Log Format
<img width="651" height="597" alt="image" src="https://github.com/user-attachments/assets/2883a640-808f-48d5-ae0b-f7de3443bbdb" />


### Attacks in Logs
*  The sequence of log entries will only display the attacker's requests; in a real-world scenario, however, benign traffic would make up the vast majority of log entries, making it essential to develop a sharp eye for spotting malicious patterns amid normal activity. It is also important to note that in the example below, the query string in the SQLi attack is logged, so you will be able to see the full SQLi payloads.
  1. The attacker tests for potential directories and forms to exploit with a directory fuzz. The 200 response codes signify valid finds for the attacker.
  2. Next, the attacker exploits the login.php form found with a brute-force attack. Note the repeated POST requests in quick succession. The last POST request has a different response code, 302 Found, which, in this case, signifies a successful login attempt. The page then redirects to the user's account page at /account.
  3. Once the attacker gains access to the account, they attempt two SQLi payloads, ' OR '1'='1 and 1' OR 'a'='a, on the /search form. If the application builds SQL queries dynamically instead of using parameterised queries, the attacker could dump the database.
<img width="568" height="280" alt="image" src="https://github.com/user-attachments/assets/f6f5a99d-acd2-4392-af70-a1eaf2dfb7b8" />

### Log Limitations
While access logs are useful, they do not always capture the full contents of a request, particularly the body of POST or GET requests. For example, a login attempt may appear in access logs as:
* "*10.10.10.100 [12/Aug/2025:14:32:10] "POST /login HTTP/1.1" 200 532 "/home.html" "Mozilla/5.0"*"


### Investigation (Lab)
* Begin by opening up the access.log file on the desktop. The mission is to analyze the logs and retrace the attacker’s steps to reveal how the breach unfolded.
<img width="720" height="425" alt="image" src="https://github.com/user-attachments/assets/761cdd93-3858-4a39-a519-21b69717b2f8" />

**Q1: What is the attacker’s User-Agent while performing the directory fuzz?**
Answer:  FFUF v2.1.0

**Q2: What is the name of the page on which the attacker performs a brute-force attack?**
Answer: /login.php

**Q3: What is the complete, decoded SQLi payload the attacker uses on the /changeusername.php form?**
1. On the log, copy the located SQLi payload and decode it in Cyberchef.
2. Answer: %’ OR ‘1’=’1


## Task 5: Network-Based Detection
**Network traffic analysis** allows analysts to examine the raw data exchanged between a client and a server. By capturing and inspecting packets, analysts can observe attack behavior on a more detailed level, including the underlying transport protocols and the application data itself. Network captures are more verbose than server logs, revealing the data behind every request and response.

### Attacks in Network Traffic
Let's begin by reviewing the attack sequence from the previous task and comparing how it appears in Wireshark. Many different filters are available to you to highlight the fields you're interested in. Below, we have filtered for the destination IP address 10.10.20.200 and User-Agent using the http.user_agent filter.

Recall the sequence of events
1. Directory fuzz to find valid directories or forms
2. Brute-force attempt with packet 13 being a successful login
3. SQL injection attempts
<img width="1336" height="448" alt="image" src="https://github.com/user-attachments/assets/da7a0dc0-ced2-4389-99da-118249147afc" />
Now that we have examined the sequence of events, let's examine the packet details to gather further evidence of the attack. First, looking at the brute force attempts. If you recall, the logs showed us repeated POST requests to the login.php form. Now we will be able to see the actual username and password that the attacker attempted to log in with. Checking out the last request to the form (packet 13), which we know was successful, we can see that the attacker found the valid password password123. Not a very secure password, especially for an admin account!
<img width="1000" height="446" alt="image" src="https://github.com/user-attachments/assets/6edbe908-8b31-4849-9717-c68d47e63551" />
Next, let's look at the first SQLi attempt from the attack in much more detail. Inspecting the HTTP packet allows us to clearly see the payload used by the attacker and the results of the attack. In this case, the ' OR '1'='1 payload allowed the Users table to be dumped, displaying First name and Surname in clear text for the attacker! Note that MySQL protocol traffic can also be analyzed using Wireshark and show the payload and returned result.
<img width="1009" height="520" alt="image" src="https://github.com/user-attachments/assets/6ac207d8-f464-43d5-8fbd-1b3ea9f9c213" />

### Investigation Continued (Lab)
**Q1: What password does the attacker successfully identify in the brute-force attack?**
Since we know that successful logins give a response code of 302, just filter it on wireshark. Then follow the stream.
<img width="720" height="53" alt="image" src="https://github.com/user-attachments/assets/57e5c055-86bd-42c7-9732-4359220ca0ff" />
<img width="632" height="640" alt="image" src="https://github.com/user-attachments/assets/33a33dc2-42d0-4ad6-9975-eec5937652c9" />

**Q2: What is the flag the attacker found in the database using SQLi?**
Add the User-Agent as a column to make it easier to check each log.
<img width="720" height="287" alt="image" src="https://github.com/user-attachments/assets/03ca2502-6f0a-47ec-bb44-5747704f362e" />
Sqlmap is very common to use for SQLi, so I just filted it to find the log.
<img width="720" height="48" alt="image" src="https://github.com/user-attachments/assets/e186136e-de41-41af-84fb-20eedaf5f3eb" />
Follow the http stream again.
<img width="720" height="449" alt="image" src="https://github.com/user-attachments/assets/c52a3847-f8e9-4099-a4d2-c4fc8d9a8bf8" />


## Task 6: Web Application Firewall
**Web Application Firewalls (WAFs)** are often the first line of defense for websites and web applications. So far in this room, we have focused on detecting and analyzing malicious activity. Now, we will shift our focus to one of the most effective mitigation tools available. WAFs act as gatekeepers for your web applications, inspecting full request packets, similar to Wireshark but with the ability to decrypt TLS traffic and filter it before it reaches the server.

### Rules
<img width="647" height="431" alt="image" src="https://github.com/user-attachments/assets/94de4364-5b2b-4c57-95fb-a0a4acc805c3" />
Imagine you notice repeated GET requests to /changeusername with the User-Agent string sqlmap/1.9. SQLMap is an automated tool for detecting and exploiting SQL injection vulnerabilities. After reviewing the network traffic, you see that the requests include SQLi payloads.
You might create a rule to block any User-Agent string matching sqlmap:
* If User-Agent contains "sqlmap"
then BLOCK

### Challenge-Response Mechanisms
WAFs don't always need to block suspicious requests outright. For example, they can challenge requests with CAPTCHA to verify whether they come from a real user rather than a bot. This capability is extremely valuable considering that malicious bot traffic makes up 37%(opens in new tab) of global web traffic. This approach is beneficial for firewall rules with a higher chance of blocking legitimate web traffic.
<img width="1250" height="481" alt="image" src="https://github.com/user-attachments/assets/34229b71-bf7c-458c-bfd0-210b553ede5e" />










