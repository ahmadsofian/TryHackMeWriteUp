## 📝 Executive Summary
This write-up documents my practical and conceptual understanding of threat modeling as taught in the TryHackMe Threat & Risks module. 
Threat modeling is a proactive engineering practice used to identify security requirements, pin down structural vulnerabilities, and quantify risks early in the development or system lifecycle.

## 🛠️ Key Concepts & Core Definitions
To properly model threats, we must understand the distinct relationship between a threat, a vulnerability, and the resulting risk:

* **Threat:** Any potential occurrence or actor that could harm an asset (e.g., a malicious hacker, a malware strain).
* **Vulnerability:** A weakness or flaw in a system’s design, implementation, or code that can be exploited (e.g., an unpatched software bug, an open port).
* **Risk:** The mathematical or qualitative intersection of the two. 

$$\text{Threat} \times \text{Vulnerability} = \text{Risk}$$

> 💡 **Takeaway:** A threat actor cannot cause harm if there is no vulnerability to exploit. Conversely, a severe vulnerability poses zero active risk if there is no viable threat actor capable of reaching it.

---

## 🔄 The Threat Modelling Process
Threat modeling is not a one-time technical checkbox; it is a continuous, collaborative lifecycle that requires input from cross-functional teams across the business:

```text
  1. Define Scope ──> 2. Identify Assets ──> 3. Identify Threats
                                                       │
  6. Monitor & Evaluate <── 5. Countermeasures <── 4. Analyze & Prioritize
```

## PASTA Framework (Risk-Centric Hacking)
Process for Attack Simulation and Threat Analysis (PASTA) is a 7-step, risk-centric threat modeling methodology. It aims to align technical threat identification directly with business objectives, using simulation techniques to see if attackers can achieve their goals.

## DREAD Framework (Risk Prioritization)
Once threats are identified, DREAD is used to qualitatively score and prioritize remediation efforts (typically on a scale of 1-3 or 1-10 per category):
* **Damage:** How severe would the impact be if the exploit succeeds?
* **Reproducibility:** How easy is it for an attacker to replicate the attack?
* **Exploitability:** How much effort/skill is required to execute the exploit?
* **Affected Users:** What percentage of our infrastructure or user base would be impacted?
* **Discoverability:** How easy is it for an external threat actor to find this vulnerability?

---

## 🔬 Practical Lab Application: MITRE ATT&CK Framework
The practical portion of this room involved utilizing the MITRE ATT&CK Matrix to ground real-world threat modeling scenarios in actual, observed adversary behaviors.
1. Identifying Potential Attacks: Moving away from theoretical assumptions by using MITRE's vast database of real-world Threat Actor Groups (APTs) to see exactly how they target similar industries.
2. Developing Threat Scenarios: Building specific attack paths based on known adversary Tactics, Techniques, and Procedures (TTPs).
3. Prioritizing Remediation: Mapping our existing defensive controls against the MITRE matrix to identify "blind spots"—areas where we have no detection coverage for active real-world techniques.

### Lab Hands-on
During the lab, I used the MITRE ATT&CK Navigator to filter the isolate threat data:
* **Platform** Filtering: Narrowed down techniques specifically targeting relevant environments (e.g., Linux, AWS Cloud, or Windows).
* **Actor Profiling**: Filtered for specific threat groups known to target manufacturing and infrastructure setups.
* **Defense Mapping**: Highlighted and color-coded techniques that our simulated environment could successfully detect versus those where we lacked logging or security controls.

---

## 💡Key Takeways
* Threat modelling should be happened during the design phase. Fix architectural flaw before the code is written is exponentiallt cheaper and safer than fixing a live breach.
* Vulnerability is only as dangerous as the threat actor who can reach it and the value of the asset it protects.
* A mature security engineering team uses STRIDE to find the threats, MITRE ATT&CK to simulate real-world attack behaviors, and DREAD to decide what to patch first.



