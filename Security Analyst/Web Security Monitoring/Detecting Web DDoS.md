## Task 2: DoS and DDoS Attacks
Denial-of-Service (DoS) attacks are meant to overwhelm a website or app so that people can’t use it. When this happens, customers can’t log in, shop, or access services, and businesses lose money and trust.

### DoS
Any DoS attack, whether small or large in scale, is considered successful if it prevents a web service from functioning as intended. Let’s begin by looking at how even a simple, targeted attack can cause a website to become unavailable.
<img width="760" height="220" alt="image" src="https://github.com/user-attachments/assets/adb3fcd8-4051-4be2-bdea-89d6bb91750b" />

### DDoS
To scale up, attackers turn to Distributed Denial-of-Service (DDoS) attacks and utilize botnets, an army of compromised devices under their control. The computers, IoT devices, and even servers are often infected with malware and secretly controlled by the attacker. When instructed, the bots flood the target website or web application with traffic, overwhelming it much more effectively than a single machine could.
<img width="821" height="221" alt="image" src="https://github.com/user-attachments/assets/c36aa00c-abca-4ed1-b641-296ca89bff14" />

#### Types of DoS
<img width="651" height="557" alt="image" src="https://github.com/user-attachments/assets/7289e266-d76e-437f-8c4a-9d10905971ee" />

## Task 3: Attack Motivates
### Possible Attack Motives
<img width="649" height="675" alt="image" src="https://github.com/user-attachments/assets/0eed16e9-e64c-4d06-b938-e4745e8c1a7e" />

## Task 4: Log Analysis
* Web server logs are a valuable source of evidence when investigating denial-of-service attacks. Every major web service, whether Apache, NGINX, or Microsoft IIS, records web requests in a somewhat standardized log format.
* By examining these logs, analysts and responders can uncover patterns that help distinguish between normal user traffic and malicious activity.

Indicators as below:
<img width="653" height="798" alt="image" src="https://github.com/user-attachments/assets/ba6131a5-a5c8-489c-bb30-b380f30cb5ea" />
Analysts should look for multiple, layered signals forming a picture of a DDoS attempt. For example, imagine an attacker controlling a worldwide botnet aimed at a single site. You might see requests from a wide range of IP addresses across different geographic regions. These requests could hammer several resource-intensive endpoints with the same User-Agent string or a variety to appear more legitimate. Maintaining a watchlist of common indicators to be on the lookout for can be a valuable tool in an analyst's arsenal. \

### Targeted Resources
If an attacker aims to disrupt a web service like we've discussed, they will likely focus on endpoints that consume the most server resources per request or are most critical to maintain site functionality. Pages like /login or search forms are prime targets because each request forces the server to query a database, validate input, and return results. This makes the requests far more expensive to process than static content like product pages or images. \

Commonly targeted endpoints and reasoning: 
* /login - involves authentication processes
* /search - requires complex database queries
* /api endpoints - critical for dynamic content delivery
* /register or /signup - requires database writes and validation
* /contact or /feedback - requires database entries and can trigger email notifications
* /cart or /checkout - requires session management, inventory checks, and payment processing

### Log Sample
Let's examine a sample condensed access log to see how a DoS attack might appear in a post-incident scenario. 

1. **Normal User Traffic** - Every few seconds, a user requests a page and receives a response as expected
2. **DoS Attack** - Beginning at 10:01:10, you can see the IP address 203.0.113.55 begin to send repeated GET requests to /login.php
3. **Web Server Down** - Users are requesting pages and receiving 503 responses indicating the service is unavailable

This log snippet is highly condensed, and a DoS or DDoS may have hundreds or thousands of requests flooding the logs simultaneously.
<img width="690" height="330" alt="image" src="https://github.com/user-attachments/assets/a1984a31-b86c-472e-b0d9-5c230b1747e2" />

### Hands On (Lab)
**Q1: What is the attacker’s IP address?**
<img width="763" height="144" alt="image" src="https://github.com/user-attachments/assets/a7b7b444-e714-4cc4-983d-dfce62128719" />
\
**Q2: Which page is repeatedly targeted by the attacker’s requests?**
<img width="482" height="302" alt="image" src="https://github.com/user-attachments/assets/1456df64-fb1d-4c87-ae45-ef0000eff7ad" />
\
**Q3: After the attack, what error code do legitimate users receive?**
<img width="765" height="101" alt="image" src="https://github.com/user-attachments/assets/c0744208-5c04-4e0d-a9cc-63a72d16f0a5" />


## Task 5: Leveraging SIEMs
**Security Information and Event Management (SIEM)** platforms make log analysis far more efficient by providing the ability to combine multiple log sources and query for useful fields. Instead of scrolling through endless raw entries in a log file, you can filter and sort logs by IP address, user agent, or response code. This ability makes identifying patterns in traffic much easier. \

In the screenshot below, you can see a timechart created in Splunk based on all requests to a server over a period of ten minutes.

1. **Normal User Requests** - A few requests to various pages every minute
2. **DoS Attack** - 1,000 requests to /login.php within a one-minute timeframe
3. **Requested Pages**
<img width="520" height="174" alt="image" src="https://github.com/user-attachments/assets/3532858c-9450-47ec-8428-33b731ebbe29" />
Looking over the same requests but filtering by the user agent (useragent) and IP address (clientip) fields enables you to see more details about where the requests originated.
<img width="520" height="232" alt="image" src="https://github.com/user-attachments/assets/ec65f186-f7b4-42bd-9add-2f7a3d0d557d" />

### Hands On (Lab)
**Q1: What was the most frequently requested uri?**
<img width="582" height="248" alt="image" src="https://github.com/user-attachments/assets/25cafd3d-0b30-4236-aa27-33a76e81dac1" />
\
**Q2: Which clientip made the most requests to the target uri?**
<img width="389" height="139" alt="image" src="https://github.com/user-attachments/assets/d52c7d6b-9e89-43bd-bea0-46a57634134a" />
<img width="767" height="126" alt="image" src="https://github.com/user-attachments/assets/5878e2d4-7e90-4d6b-b5aa-aa4746072b04" />
\
**Q3: How many IP addresses were part of the botnet that attacked your website?**
<img width="551" height="269" alt="image" src="https://github.com/user-attachments/assets/d858008f-2e8d-4d3b-8470-754dd30cc3c6" />
\
**Q4: Which useragent was most commonly used by the attacking traffic?**
<img width="768" height="279" alt="image" src="https://github.com/user-attachments/assets/eb94e73c-b681-41b2-8e6a-bfa8795088f8" />
\

**Q5: Use the timechart command to visualize the requests.
What is the peak number of requests made per second during the attack?**
<img width="767" height="276" alt="image" src="https://github.com/user-attachments/assets/bf5dd9f0-0fc6-447b-9458-d1a157c369fa" />

## Task 6: Defense
### Application Level Defense
**Secure Development Practices**
* A secure site starts with secure code. Search fields and forms must validate input so they can’t be abused. Think of a search form like a librarian who looks up books on request. If the librarian has clear rules, like “only search for titles under 50 characters”, they can respond quickly. Without rules in place, someone could ask the librarian to search for an overly long or strange title with special characters, slowing them down and delaying everyone else's requests. In the same way, web applications need input validation to stop attackers from submitting specialized queries aimed at overloading the system. \

**Challenge**
* One way to stop automated traffic is to require a challenge before granting access. This could be a CAPTCHA, where the user solves a puzzle, like clicking images or checking a box. For humans, it’s a small step, but for bots, it can block or slow down an attack.
* Websites can also use JavaScript challenges, which run quietly in the background to confirm if a visitor is a real user or automated traffic. Legitimate users usually don’t notice them, but automated tools and botnets often fail, making these challenges an effective filter against malicious traffic.
* <img width="616" height="164" alt="image" src="https://github.com/user-attachments/assets/371b5632-0d22-403d-b6fc-50c03d7b4fcf" />

### Network and Infrastructure Defenses
**Content Delivery Network (CDN)**
CDNs help manage server load by caching and serving content from edge servers closest to users. This reduces latency and allows the origin server to handle only a fraction of requests, while the CDN serves the majority. As a result, CDNs take on much of the burden of mitigating DDoS attacks. They also provide load-balancing to distribute traffic across servers, ensuring no single server is overloaded and rerouting requests if one becomes unavailable. \

Here’s an example of a 16 TB DDoS attack displayed in the Cloudflare CDN dashboard.
1. Total bandwidth over a 30-day period - This shows the entire traffic volume over the last 30 days. If your site normally sees a few hundred gigabytes a month, suddenly seeing 16 TB indicates something is abnormal.
2. Total cached bandwidth by CDN edge servers - This represents how much traffic was successfully delivered by the CDN's edge servers. Nearly all of the bandwidth being cached indicates that Cloudflare absorbed the attack traffic before it could overwhelm the backend.
3. Traffic spike from DDoS attack - The spike in the graph is the signature of the DDoS attack. Without a CDN, the flood of requests would have hit the origin server.
<img width="520" height="224" alt="image" src="https://github.com/user-attachments/assets/d7016d55-4b3b-4a4a-90b8-751abf38f07d" />
Beyond absorbing traffic, CDNs also provide analysts with powerful visibility, including helpful visuals and diagnostics of website traffic. This lets you quickly break down requests by geographic location, volume, and source patterns, helping distinguish malicious traffic from legitimate users.
<img width="520" height="248" alt="image" src="https://github.com/user-attachments/assets/ed3ce796-ba14-480f-85d8-1bad08841307" />


**Web Application Firewall (WAF)**
* CDNs typically integrate WAFs in an effort to shield their customers' servers. They inspect incoming traffic and either allow, challenge, or block requests. WAFs work off of rules that integrate known attack indicators and threat intelligence. Modern WAF solutions are already very good at mitigating DoS and DDoS attacks because they know what to look out for. Custom rules can also be developed to assist in defending against targeted threats.
* For example, you might implement a rate-limiting firewall rule that limits requests to /login.php to five per minute. If the originating IP requests the page more than five times, it would be blocked from making future requests for a period of time or provided with a challenge to prove it is a human-made request. 
<img width="1357" height="240" alt="image" src="https://github.com/user-attachments/assets/19edfa57-1ff3-46fd-962b-4e1872125278" />

### Large-Scale Mitigation
Modern defense solutions can block denial-of-service attacks and also leverage their vast distributed infrastructure to absorb and balance large volumes of traffic. In 2023, Google(opens in new tab) mitigated a large-scale DDoS that peaked at 398 million requests per second. Cloudflare(opens in new tab) claims to have mitigated the largest DDoS ever recorded, which peaked at 11.5 Tbps and lasted only 35 seconds. These examples demonstrate how providers use global networks and traffic filtering to keep even the largest attacks from taking critical services offline.

### Bypassing Security Measures
While CDNs, caching, and WAFs provide strong protection, attackers often attempt to bypass these defenses. One technique is appending random query parameters. Your CDN might serve a cached URL at /products, but if an attacker appends the query with a random string like /products?a=abcd, your CDN cannot serve the cached page, and the origin server is forced to respond. Similarly, changing user agents, spoofing referrer pages, or launching requests from diverse geographic regions can help attackers evade WAF filtering rules.










