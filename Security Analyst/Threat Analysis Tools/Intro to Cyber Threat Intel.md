## Task 2: Cyber Threat Intelligence
In concrete terms, CTI seeks to answer three essential questions:
* Who, or what, is on the other end of this alert indicator?
* What was their behaviour in the past?
* How does my organisation respond, and what should I do about it right now?

### From Raw Data to Usable Intelligence
Information security literature distinguishes data, information, and intelligence, yet the three terms often blur in daily conversation. Making them explicit clarifies an analyst's objective.
<img width="1235" height="258" alt="image" src="https://github.com/user-attachments/assets/3f961700-d521-4c9c-83d2-7e3115ccc76e" />

A Level 1 analyst is responsible for making the artefacts usable and enriching them until they qualify as intelligence, or demonstrating that they never will. That push is enacted through enrichment: rapid, methodical lookups of public, commercial, and internal sources that shed light on origin, behaviour, and relevance.
During the ascent of data to intelligence, three more labels become paramount for analysts to know.
* Indicator of Compromise (IOC): Evidence of a breach, such as a C2 address in the logs.
* Indicator of Attack (IOA): A malicious action, such as PowerShell launching an unknown service, is underway.
* Tactics, Techniques, and Procedures (TTP): An adversary's detailed methodologies expressed in MITRE ATT&CK IDs and descriptions.

### Indicator Types Essential to First-Line Triage
Every artefact demands a tailored enrichment path. Memorising tools is less important than recognising what kind of indicator the alert supplies and knowing where to look. Below, we have a table showing the types of indicators we need to be aware of, with examples:
<img width="1232" height="552" alt="image" src="https://github.com/user-attachments/assets/29f0a6c0-f683-492e-a974-f46813cd300a" />

Actionable tip. Maintain a browser bookmark folder or a SIEM launcher panel that opens your preferred look-ups with the highlighted indicator pre-populated. The thirty seconds saved per alert compound into hours over a month.

### Feeds, Platforms, and Why the Distinction Matters
 
* **Feed:** A scheduled stream of indicators, usually delivered in various formats such as CSV, JSON, STIX, or through TAXII. Over-ingesting feeds without curation drowns analysts in false positives and erodes trust in the programme.
* **Platform:** A structured repository that stores indicators, tracks enrichment, maps relationships, and enforces sharing permissions. MISP and OpenCTI are leading open-source examples.
Sound CTI practice introduces feeds gradually, confirms they align with the organisation's threat model, and promotes them into the platform only after measuring actionability. The platform then becomes the single source of truth—an analyst queries it first, ensuring biographies of indicators evolve rather than fork.

### Sources of Cyber Threat Intelligence
Intelligence is only as trustworthy as its source, for it drives credibility and steers legal review if an IOC later triggers business disruption. As an analyst, you must know and note where each indicator originated.
There are four broad sources that you will come across in your practice:
* Internal telemetry: SIEM logs, EDR detections, phishing-mailbox submissions provide the highest immediate relevance.
* Commercial services: Vendor premium feeds, paid sandboxes**,** and closed-source analytics. These provide high fidelity, but may have export and sharing limits based on licensing.
* Open-source intelligence (OSINT): AbuseIPDB, URLhaus, public blogs with IOCs, and academic research. Before applying, information from these sources will need to be cross-confirmed.
* Communities & ISACs: Sector-specific lists marked with labels and rich context (e.g., FS-ISAC)

### Threat Intelligence Classifications
Threat intelligence is geared towards understanding the relationship between your operational environment and your adversary. With this in mind, we can break down threat intel into the following classifications:

* **Strategic intel:** High-level intelligence that looks into the organisation's threat landscape and maps out the risk areas based on trends, patterns and emerging threats that may impact business decisions. An example is an annual ransomware trends report predicting a shift to data-wiping extortion in healthcare.
* **Tactical intel:** Assessments of adversaries' behaviours through analysis of tactics, techniques, and procedures (TTPs). This can be in the form of Advisory notes, such as detailing new T1059.005 (Visual Basic) abuse in malspam.
* **Operational intel:** Campaign-specific details about the motives and intent to perform an attack. This is useful for understanding the critical assets available in the organisation (people, processes, and technologies) that may be targeted.
* **Technical intel:** Atomic indicators and artefacts such as IPs and hashes related to an attack.

## Task 3: CTI Lifecycle
Cyber Threat Intelligence follows a six-phase intelligence lifecycle that transforms raw data into contextualised and action-oriented insights geared towards triaging security incidents. We shall look at this lifecycle as a narrative that follows a single SOC Level 1 analyst, Alex, as they weave CTI into a routine defence objective: protecting TryHatMe's production database.
<img width="1140" height="800" alt="image" src="https://github.com/user-attachments/assets/fb85de5a-df30-447e-80f1-b901c0966788" />

**Traffic Light Protocol (TLP)—A Primer for Proper Sharing**
The Traffic Light Protocol is a four-colour labelling scheme defined by FIRST.org that governs how widely intel may be shared. These labels can be summarised in the following table:
<img width="1237" height="327" alt="image" src="https://github.com/user-attachments/assets/5b8204ed-584d-46b3-b37f-664e6aa9aca4" />

### Scenario Premises
#### Step 1: Direction - Defining the Mission
Begin brief planning meeting with the CTI lead and database admin. Together, to get the intelligence requirement:
1. Primary asset. The PostgreSQL production database.
2. Business risk. Data breach fines under GDPR and loss of customer trust.
3. Available controls. NGFW is capable of IP- and domain-based blocking; EDR can quarantine files by hash.
4. Initial CTI goal. Use threat-feed indicators to:
* Block or alert on suspicious IP addresses at the firewall, and
* Detect known malicious file hashes at the EDR layer

Construct measurable question that will steer the subsequent collection and analysis

#### Step 2: Collection - Assembling the Raw Material
Guided by the question to gather intelligence from four sources:
<img width="1231" height="359" alt="image" src="https://github.com/user-attachments/assets/f34e05d4-c95a-4f5c-8897-4109375967cd" />
Alex exports each feed in STIX or CSV, then stores a dated copy in the SOC's "raw-intel" S3 bucket, ensuring reproducibility.

#### Step 3: Processing - Normalising and Correlating the Data
Raw feeds never align perfectly, and artefacts may arrive in mixed formats. To correct this, threat intel will need to be normalised and correlated. Normalising ensures threat data from various sources is standardised into a standard format for consistent analysis. Correlation involves identifying connections and links between data from different sources and threats. It can also help identify contradictions, such as having a benign tag in one's history versus a malicious label in new data today.

Processing threat intel will involve executing scheduled Python scripts that:
1. Normalises indicator syntax such as having IPv6 addresses compressed, domains in lowercase or stripping off IP subnet masks.
2. Correlates and deduplicates against the platform's existing indicator table.
3. Tag each entry with source, date, and TLP label.
4. Converts the final set of intel into two action files, such as:
* firewall_blocklist.csv, a firewall-ready CSV that covers the indicator, action, and comments
* edr_hash_rules.yar, an EDR YARA-rule file for hash blocking.


#### Step 4: Analysis - Turning Information into Judgement
This results in providing answers to the questions in Phase 1:

* Firewall indicators (Q1). Cross-check: A Splunk query over the previous 30 days shows one of the NGFW IPs attempting but failing to open TCP 5432 against the production subnet. That event validates the indicator's applicability.
* Hash indicators (Q2). Reverse-search: OpenCTI links the new hash to the PgSteal malware family; Any.Run sandbox analysis demonstrates credential-dump behaviour. Because the organisation uses exactly the named ODBC driver, the hash is marked as high priority, and relevance is confirmed.
Then grades each indicator:
<img width="1237" height="257" alt="image" src="https://github.com/user-attachments/assets/df70d030-0d3a-4dfd-a42b-c425817c1909" />

#### Step 5: Dissemination – Getting Intelligence to the Right Consumers
Unused threat intelligence becomes shelfware. We delivers tailored output:
<img width="1233" height="322" alt="image" src="https://github.com/user-attachments/assets/3e1fa7d4-851c-427b-80bf-702630e017fc" />
By tailoring depth to the audience, We ensures that each consumer receives only the detail required to act, nothing more and nothing less.

#### Step 6: Feedback – Measuring and Refining the Cycle
Two weeks later, metrics and reports from the firewall and EDR reports show the progress of establishing an effective threat intelligence workflow:
<img width="1236" height="201" alt="image" src="https://github.com/user-attachments/assets/a88e9fc9-ef80-43ca-b09f-8473750341dd" />

Because the outcome meets the initial direction objectives, management approves expansion: the next sprint will add other threat parameters, such as DNS tunnelling IOCs for the same asset group. 

## Task4: CTI Standard & Framework
### MITRE ATT&CK
As an L1 analyst, you can use the matrix during an investigation in the following way:
1. Match the behaviour in the alert to a tactic/technique pair.
2. Write the ID in your triage note: "Observed T1071.001 (web-based C2) against FINANCE-TRYHATME-00".
3. Hand the note to Level 2 or Incident Response; they instantly know which mitigations and threat-actor profiles apply.

### MITRE D3FEND
If ATT&CK catalogues how adversaries attack, D3FEND catalogues how defenders respond. Each entry maps to defensive tactics such as Credential Hardening or Data Obfuscation_._

A case for this would look similar to this:
* Your proxy raises a T1048.003 DNS tunnel alert.
* Search D3FEND for the matching defensive technique: D3—NTDN DNS—request analysis. The page lists practical controls: block extensive TXT records and alert on uncommon query entropy.
* Add the most feasible control to your "next actions" field; you have just supplied a mitigation, not only a diagnosis.

### Cyber Kill Chain
<img width="1169" height="623" alt="image" src="https://github.com/user-attachments/assets/2cfd25b0-63d4-41d4-b241-cf9d300a6854" />

### CVEs, CVSS and the NVD
A SOC queue contains almost as many vulnerability notifications as malware alerts. As an SOC L1 analyst, you must understand how to identify and organise vulnerability notifications.
* **CVE** (Common Vulnerabilities and Exposures) — provides a catalogue number for discovered vulnerabilities, e.g., CVE-2023-4863.
* **CVSS** (Common Vulnerability Scoring System) — a 0–10 severity scale with temporal and environmental modifiers for vulnerabilities.
* **NVD** (National Vulnerability Database) — the canonical repository that links CVE numbers to CVSS scores, exploits, and affected products.

### Sharing and Processing Intel
We previously discussed platforms and feeds from which threat intelligence can be retrieved. When organisations publish fresh indicators, every peer that consumes and validates them strengthens the collective defence and feeds back improvements. This information looks to hinge on two standards: STIX and TAXII.

* **STIX:** We mentioned STIX previously as the structured JSON schema for describing threat information.
* **TAXII:** The Trusted Automated eXchange of Indicator Information is a set of secure APIs used to exchange threat intelligence in near real-time for detection, prevention, and mitigation of threats. It supports two sharing models: Collection, which ensures threat intel is collected and hosted by a producer, and Channel, which publishes threat intel to users from a central server.
