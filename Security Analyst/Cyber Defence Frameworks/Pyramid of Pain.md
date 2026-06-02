### Hash Values - Trivial
* Common hashing algorithm is the **MD5**, **SHA-1** , **SHA-2**
* has not considered cryptographically secure if two files have the same hash values or digest
* Online tools for hash lookups - VirusTotal and Metadefender Cloud - OPSWAT.
* In this topic I learn to perform has lookups by using the VirusTotal, found the name of the file samples from given hashes.

### IP Address - Easy
* Common defense tactic is to block, drop or deny inbound request from IP addresses.
* **Fast Flux** -  a DNS technique used by botnets to hide phishing, web proxing, malware delivery and malware communication activities behind compromised host acting as proxies. Communication between malware and its C&C server is challenging to be discovered by security professionals.
* In this topic i learn to read the report from **Any.run** to identify the IP Address based on the malicious process given and the domain name of the malicious IP Address attempt to communicate with.

### Domain Names - Simple
* Hands-on lab exercise using the **app.any.run**. Assessed the HTTP Request, Connections and DNS Request.
* Analyse the domain name, identified the unicode attack and understand the analysis process

### Host Artifacts - Annoying
* Host artifacts are the traces or observables that attackers leave on the system, such as registry values, suspicious process execution, attack patterns or IOCs (Indicators of Compromise), files dropped by malicious applications, or anything exclusive to the current threat.
* Reviewed the Malware Analysis Sandbox report from Any run. To find the malware from given IP Address connections.
* Reviewed VirusTotal report to finde th vendors that determine the host to be malicious.

### Network Artifacts - Annoying
* Can be a user-agent string, C2 information, or URI patterns followed by the HTTP POST requests.
* Attacker might use a User-Agent string that hasn’t been observed in the environment before or seems out of the ordinary.
* Network artifacts can be detected in PCAPs (file that contains the network packet dumps) by using a tool such as Wireshark or TShark, or exploring IDS (Intrusion Detection System) alerts from a tool such as Snort(opens in new tab).
* Analyzed a PCAP file to identify the Emotet trojan.

### Tools - Challenging
* Antivirus signature, detection rules and YARA rules can be great weapons to use against attackers at this stage.
* MalwareBaazar and Malshare to provide access to sample, malicious feed and YARA results. Helpful for TH and IR.
* SOC Prime Threat Detection Marketplace is a place where pro shares their detection rules for different threats including latest CVE's.
* Fuzy hashing match two files with minor differences based on the fuzzy hash values. SSDeep.

### TTPs - Tough
* Tactics, Techniques & Procedures. Include the wholde MITRE ATT&CK Matrix.
* Used ATT&CK to analyze techniques fall unders Exfiltration category.
