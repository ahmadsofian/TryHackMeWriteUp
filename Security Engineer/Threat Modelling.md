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
