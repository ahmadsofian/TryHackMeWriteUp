## Task 2: Filenames and Paths
### Filepath Analysis
File paths and names are like crime scene clues, revealing attacker behaviour. Attackers may use different disk locations to hide their actions and reduce visibility. For example:

* C:\(System drive) can be a common target for persistence mechanisms.
* C:\Users\Public profile can enable cross-user access of detonated adversary tools.
* C:\Users\Public\Public Downloads provides a high-traffic directory that would often evade strict monitoring.

Additionally, adversaries may utilise other malware staging patterns such as:
* Utilising temporary directories such as C:\Windows\Temp\ for ephemeral payloads.
* Placing payloads in writable system paths, such as C:\ProgramData\ for stealth persistence.

### Filename Heuristic Indicators
Attackers are also known to modify filenames to escape detection through implementing various types of heuristic indicators, including:

* Double extensions - An example is invoice.pdf.exe, which leverages Windows' default settings that hide file extensions.
* System binary impersonation - A filename such as scvhost.exe abuses the user's familiarity with core system processes. Defenders should include legitimate locations for system processes in an allowlist, rather than standalone filenames.
* High-entropy Strings – A filename such as jh8F21.exe suggests automated packing or polymorphic generation, which is commonly used in a high-churn phishing operation.
* Masquerading - Filenames such as backup-2300.exe can blend with routine files, thus leveraging on reduced suspicion. Another example is a single character substitution, which can bypass detection while looking visually legitimate to an unsuspecting employee.

## Task 3: File Hash Lookup
We've identified suspicious heuristics from the sample files through path and name analysis. The analysis is inconclusive because attackers constantly rename files. We can tie the names back to the same binary through cryptographic fingerprinting through its file hash, especially SHA256 and MD5 hashes. File hashes uniquely identify files and malware, regardless of name changes. Attackers constantly rename their malware; hashes provide an immutable, actionable way to identify it.

It is advisable to work with potentially malicious binaries in a separate, isolated environment.

We can generate and verify our file hash using the following commands:
<img width="1235" height="240" alt="image" src="https://github.com/user-attachments/assets/5c6fbdb7-567f-446a-ae84-30a5b6e4c572" />

A few pointers as analysts when dealing with hashes:
* Store the hashes in lowercase to avoid needless differences.
* Hash what matters in your investigation. For example, if the malware resides in a ZIP file, hash both the archive and the extracted binary.
* Do not leave plain strings without the context of where and when you encountered them.
* Any byte change will change the resulting hash.

### Analysis With VirusTotal
There are several items from the search results that would be worth taking note of when you submit a hash:

* **Detection score:** This represents a crowdsourced security verdict from various vendors displayed as a ratio. The higher the number, the higher the confidence threat.
* **Threat labels and categories:** These are vendor-specific classifications of threats that help confirm their attribution across vendors.
* **Detection rules:** These are the technical signatures used by AV engines to identify threats. Typical classifications are YARA rules, Heuristic patterns, and behavioural triggers.
* **Properties:** This is where the core metadata for the file is stored, including the file type, size, and compilation timestamp.
* **Contained domains and IPs:** This information covers the malware's network infrastructure.
* **Contained files:** This section details any files embedded or dropped during the malware's execution.

We can break down the process above into granular steps that ensure VirusTotal becomes a tactical threat intelligence platform for the SOC to pivot towards actionable defence measures.
<img width="1231" height="838" alt="image" src="https://github.com/user-attachments/assets/daaf1a50-e17d-496a-80b3-1f48c7b0d75a" />

### Coress-Reference With MalwareBazaar
MalwareBazaar(opens in new tab) is an all-in-one database for malware collection and analysis. The project supports the following features:

* Malware Samples Upload: Security analysts can upload their malware samples for analysis and build the intelligence database. This can be done through the browser or an API.
* Malware Hunting: Hunting for malware samples is possible through various elements, such as :
* Malware Family tagging: You will find files classified by their malware families. An example of this is a file with only 5/70 detections on VirusTotal but tagged as #IcedID in MalwareBazaar; it should be treated as malicious.
* YARA rule integration: Many submissions will include rules that detect related samples. As an analyst, you should take note of these rules to be added to the EDR/SIEM for future hunting.
* Campaign attribution: Tags such as #TA551, which belong to a threat actor group, help link observed incidents to known adversaries. This can help identify coordinated attacks against an environment.
* Sample Availability: Malware samples are available for download and analysis. Reanalysing samples in a sandbox is best practice, which we shall cover in the next task.

### Lab
**Q1: What is the SHA256 hash of the file bl0gger?**
<img width="856" height="160" alt="image" src="https://github.com/user-attachments/assets/c75e17e9-456b-46a7-9102-96c4007150a0" />

**Q2: What is the threat classification label used to identify the malicious file?**
<img width="765" height="141" alt="image" src="https://github.com/user-attachments/assets/004a5ed7-18ce-4d21-bc11-4f5db6b27ce1" />


**Q3: When was the file first submitted for analysis? (Answer format: YYYY-MM-DD HH:MM:SS)**
<img width="429" height="149" alt="image" src="https://github.com/user-attachments/assets/355fa298-8553-4978-8ae2-87fd49af6973" />

**Q4: Which vendor classified the Morse-Code-Analyzer file as non-malicious?**
<img width="396" height="220" alt="image" src="https://github.com/user-attachments/assets/550f06a3-16e1-46ef-92da-95eac052e643" />

**Q5: What MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?**
<img width="682" height="290" alt="image" src="https://github.com/user-attachments/assets/d17002f1-b24e-4b14-b83b-37e8ecc5f2a2" />


## Task 4: Sandbox Analysis
### Dynamic Analysi for SOC
Static properties such as hashes, strings, and sections tell you identity. To understand the impact of malicious files, you need to execute them safely in a controlled environment. Sandboxes provide a disposable VM instrumented to capture every process, registry write, and network packet associated with the malware being investigated.

Security analysts will use sandboxes to do three things:

* Confirm execution - if nothing happens, your alert might be a decoy.
* Extract runtime IOCs - domains, mutexes, dropped payloads, feed hunts and detections.
* Map to ATT&CK - most sandboxes auto‑label behaviour with technique IDs, fast‑tracking your report.

### Sandboxing Tools
The most widely used tools for detonating wild malware are Hybrid Analysis(opens in new tab) and Joe Sandbox(opens in new tab), each with its distinct application:
* Hybrid Analysis (HA) focuses on behaviour trees and a clean MITRE ATT&CK heatmap. It is suitable for a fast executive summary.
* Joe Sandbox (JS): goes deep, covering system calls, strings, and memory dumps. Great for reverse engineers and detection engineers.

### Analysing Hybrid Analysis Report
When we search using the file hash and analyse the generated reportfrom Hybrid Analysis, we can confirm that the file is malicious, with a threat score of 100/100. Additionally, TryDetectThis has all the details ingested for the remainder of the room.

We are also provided with information about the file's risk behaviour and the ATT&CK Techniques the threat actor observed using.

Additionally, we can gather all file details and information to inform our threat intelligence report. These details are vital for threat reporting, both internally and externally, including file compositions, imports, processes called, and network connections.

### Sandboxing Limitations
**1. Sandbox Evasion Techniques**
Threat actors would design their payloads to detect and evade sandboxes, leading to false negatives. Some of the common evasion tactics used include:
* Environment Awareness Checks: Malware checks for signs of virtualised/sandboxed environments
* Anti-Debugging & Anti-Sandboxing Tricks: Malware conducts debugger detection and checks for unique hardware IDs.

**2. Limited Execution Time & Coverage**
Most sandboxes terminate analysis after 2-5 minutes, which means multi-stage malware may not fully execute. Additionally, time-delayed attacks will evade detection. As previously covered, this would mean cross-referencing other threat intelligence resources.

**3. Encrypted & Obfuscated Traffic**
Many sandboxes cannot decrypt SSL/TLS traffic, leading to blind spots. This may result in HTTPS C2 Traffic appearing with no payload visibility or the malware utilising DNS tunnelling to hide data in DNS queries.

**4. Fileless & Living-off-the-Land (LotL) Malware**
Some threats never touch disk, bypassing traditional sandbox analysis by employing PowerShell Attacks and WMI Persistence.

### Lab
**Q1: What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis? (Answer: Tag1, Tag2, Tag3)**
<img width="366" height="198" alt="image" src="https://github.com/user-attachments/assets/002ffa61-317b-458c-8f4d-ab39f977a9d2" />

**Q2: What was the stealth command line executed from the file?**
<img width="724" height="43" alt="image" src="https://github.com/user-attachments/assets/5629e379-c454-4b79-be49-ae55eb5edccc" />

**Q3: Which other process was spawned according to the process tree?**
<img width="755" height="66" alt="image" src="https://github.com/user-attachments/assets/911b3313-6253-4562-80d6-b6aea8f32a76" />

**Q4: Analyze the payroll.pdf file located in the CTI Files folder and answer the questions below. The payroll.pdf application seems to be masquerading as which known Windows file?**

**Q5: What associated URL is linked to the file?**
<img width="529" height="143" alt="image" src="https://github.com/user-attachments/assets/82df630b-574c-43c4-86f3-18bc35e5fb59" />

**Q6: How many extracted strings were identified from the sandbox analysis of the file?**
<img width="698" height="240" alt="image" src="https://github.com/user-attachments/assets/50976cb0-46dd-46a6-8936-062f9c3fd920" />

## Task 5: Threat Intelligence Challenge
**Q1: What is the SHA256 hash of the file?**
<img width="763" height="135" alt="image" src="https://github.com/user-attachments/assets/4ffd4c19-1fb5-49e7-959b-da72d5c80007" />

**Q2: What family labels are assigned to the file on VirusTotal?**
<img width="376" height="52" alt="image" src="https://github.com/user-attachments/assets/a5ddfd59-a40f-4913-85c5-08ce6bbd54b4" />


**Q3: When was the first time the file was recorded in the wild? (Answer Format: YYYY-MM-DD HH:MM:SS UTC)**
<img width="619" height="172" alt="image" src="https://github.com/user-attachments/assets/b0250af3-3caf-4a84-b502-a6eec0ac586c" />

**Q4: Name the text file dropped during the execution of the malicious file.**
<img width="723" height="135" alt="image" src="https://github.com/user-attachments/assets/8a2686d8-f761-445c-8b3f-312dad4363ed" />


**Q5: What PowerShell command is observed to be executed?**
<img width="766" height="140" alt="image" src="https://github.com/user-attachments/assets/b4f2aae4-23ff-4271-9738-1c0b74340c6e" />

**Q6:What MITRE ATT&CK ID is associated with the actions of the command?**
<img width="766" height="140" alt="image" src="https://github.com/user-attachments/assets/32f1a934-9052-4004-865c-de9f269b77cd" />



