### Reconnaissance
* research and planning phase of an attack
* Gather information about the target to infor, for the next stage.
* OSINT
* Passive Recon -> Having no direct interaction with the target - WHOIS lookup, socmed scrapping, review breach data.
* Active Recon -> Direct contact with the target - social engineering, port scan, probing.

### Weaponization
* Threat actor write payload, malware to exploits the target system.
* E.g., infected documents, USB drivers, C2 infra, backdoor host, phishing templates

### Delivery
* Phishing email
* USB drops
* Watering hole attacks

### Exploitation
* Malicious macro execution, zero-day exploits, knwon CVEs
* Signed of exploitation could be --> unexpected process spawns, registry changes or new services created, suspicious commad-line.

### Installation
* Once attacker get access to the system, they might want to reconnect back if they lost connection or got detected.
* **Persistent backdoor** will let attacker access the system they compromised in the past. Can be achieve through:
    a) installing web shell on the webserver - web shell simplicity file formatting can be difficult to detect.
    b) install backdoor on victim's machine - use **Meterpreter** to install backdoor.
    c) create/modifying Windows services - technique known as **T1543.003** on MITRE ATT&CK.
    d) add entry to "run keys" for malicious payload in the Registry or the Startup Folder.
* This phase, attacker could use **Timestomping** to avoid detection by the forensics investigator.

### Command & Control
* Malicious connection between a C&C server and malware on the infected host. Infected host consistently communicate with the C2 server.
* After establish connection, the attacker has full control of the victim's machine.
* The most common adversaries:
    a) HTTP on port 80 and HTTPS on port 443, beaconing blends the malicious traffic with legitimate traffic, help evade firewalls
    b) DNS, infected machine makes constant DNS request to the DNS server that belongs to an attakcer (DNS Tunneling).

### Exfiltration (Actions on objective)
* Collect credentials
* Privilege escalation
* Internal recon
* Lateral movement in country environment
* Collect and exfiltrate sensitive data
* Delete backups and shadow copies
* Overwrite or corrupt data





