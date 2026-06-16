# TryHackMe: Risk Management

## 📝 Executive Summary
This write-up covers the fundamentals of enterprise risk management based on the NIST SP 800-30 framework. 
The lab focused on identifying, analyzing, responding to, and monitoring risks, with a heavy emphasis on distinguishing between qualitative and quantitative risk analysis, calculating financial metrics (SLE, ALE), and evaluating supply chain risks.

---

## 🏛️ The NIST SP 800-30 Risk Management Process
Risk management is a continuous, cyclical process designed to help an organization adapt to an evolving threat landscape. According to NIST SP 800-30, it consists of four core phases:

```text
  ┌──────────────┐       ┌──────────────┐
  │  Frame Risk  │ ────> │  Assess Risk │
  └──────────────┘       └──────────────┘
         ▲                      │
         │                      ▼
  ┌──────────────┐       ┌──────────────┐
  │ Monitor Risk │ <──── │ Respond Risk │
  └──────────────┘       └──────────────┘
```

### 1. Frame Risk
This phase establishes the context and boundaries for how an organization views risk.
* **Risk Assumptions:** Identifying the baseline threats and vulnerabilities the organization assumes it faces.
* **Risk Constraints:** Acknowledging limitations such as budget, legal requirements, or legacy technology.
* **Risk Tolerance:** Determining the level of risk executive leadership is willing to accept (risk appetite).
* **Priorities and Trade-offs:** Deciding which business functions are most critical when resources are limited.

### 2. Assess Risk
The process of identifying and analyzing risks to determine their potential operational impact. This involves identifying threat sources, uncovering vulnerabilities, estimating the likelihood of exploitation, and determining the overall impact on business operations.

### 3. Respond to Risk
Once risks are assessed, the organization must decide how to handle them. There are four primary risk treatment options:
* **Avoid:** Eliminating the risk entirely by stopping the associated activity (e.g., shutting down a highly vulnerable legacy application).
* **Transfer:** Shifting the financial burden of the risk to a third party (e.g., purchasing cyber insurance or outsourcing operations to a vendor).
* **Mitigate:** Implementing security controls to reduce the likelihood or impact of the risk (e.g., installing an EDR solution or enforcing MFA).
* **Accept:** Validating that the risk falls within acceptable tolerance levels and choosing to do nothing, often because the cost of fixing it outweighs the potential damage.

### 4. Monitor Risk
Continuously verifying that controls are operating effectively, checking that the risk environment hasn't changed, and ensuring that implemented responses match corporate risk tolerance over time.


## 📊 Risk Analysis: Qualitative vs. Quantitative
### Qualitative Risk Analysis
This approach uses descriptive scales (e.g., Low, Medium, High) to evaluate risks based on judgment and experience. 
During the lab, I utilized a standard Risk Matrix to map Likelihood against Impact to determine an overall risk score.

### Quantitative Risk Analysis
This approach assigns tangible, monetary values to risk components, enabling precise cost-benefit analyses for security investments.
* **Asset Value (AV):** The total financial worth of the asset being protected.
* **Exposure Factor (EF):**  The percentage of loss that an asset would suffer if a specific threat occurs (expressed as a percentage, e.g., 50% or 0.50).
* **Single Loss Expectancy (SLE):**  The monetary loss expected from a single risk event.

* **Annualized Rate of Occurrence (ARO):**  How many times a year the threat is expected to happen (e.g., once every two years = 0.5; three times a year = 3).
* **Annualized Loss Expectancy (ALE):**  The total estimated financial loss from a specific risk over the course of one year.
* **Value of a Safeguard**  To justify buying a security control (safeguard) to senior management, the financial value it brings must outweigh its cost. 

| Formula Name | Equation | Component Key |
| :--- | :--- | :--- |
| **Single Loss Expectancy** | `SLE = AV × EF` | **AV:** Asset Value <br>**EF:** Exposure Factor (%) |
| **Annualized Loss Expectancy** | `ALE = SLE × ARO` | **SLE:** Single Loss Expectancy <br>**ARO:** Annualized Rate of Occurrence |
| **Value of a Safeguard** | `Value = ALE(before) - ALE(after) - Cost` | **Cost:** Annual Cost of the Safeguard |

💡 GRC Decision Rule: If the resulting value is positive, the safeguard is financially justified. If the value is negative, the control costs more than the risk itself, meaning the organization should consider accepting or transferring the risk instead.

## 🌐 Supply Chain Risk Management (SCRM)
Modern organizations rely heavily on third parties. 
This lab highlighted that risk extends beyond our internal network and must be evaluated across three distinct supply chain vectors:
* **Hardware:** Mitigating the risk of counterfeit components, malicious firmware modifications, or hardware backdoors introduced during manufacturing or shipping.
* **Software:** Securing the software development lifecycle against open-source repository vulnerabilities, malicious code injections, or unpatched third-party dependencies.
* **Services:** Managing the risk of third-party vendors, consultants, or cloud providers who have access to internal systems but may maintain weaker security controls than our own.


💡 Key Takeaways
1. Risk Cannot Be Zero: Total security does not exist. The goal of GRC is to manage risk down to a level that aligns with corporate risk tolerance.
2. Money Talks to Executives: While engineers look at risks qualitatively, translating risks into Quantitative metrics (ALE/SLE) is the most effective way to secure a budget from executive stakeholders.
3. Third-Party Risk is Our Risk: A secure network can still be breached via a vulnerable vendor or software dependency. Robust Supply Chain Risk Management is vital for enterprise resilience.
