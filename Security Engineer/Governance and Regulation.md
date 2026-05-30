## 📝 Executive Summary
This write ups cover the cores frameworks, compliance regulations and cyber audit methodology
---
## 🛠️ Frameworks & Regulations Covered
*   **Regulatory Compliance:** GDPR (General Data Protection Regulation), PCI DSS (Payment Card Industry Data Security Standard)
*   **Security Frameworks:** NIST SP 800-53, ISO/IEC 27001
*   **Auditing Frameworks:** SOC 2 (System and Organization Controls)
---
# 🔍 Key Learning Domains
### 1. Regulatory Frameworks: GDPR & PCI DSS
Understand the legal and industrial baselines for protecting sensitive data:
*   **GDPR:** Focused on protecting the personal data and privacy of EU citizens. It mandates strict data handling processing principles, data minimization, and grants individuals the "Right to be Forgotten."
*   **PCI DSS:** A technical and operational framework required for any entity that processes, stores, or transmits credit card holder data. It enforces strict network segmentation, encryption of data in transit/at rest, and regular vulnerability scanning.

### 2. Strategy Controls: NIST SP 800-53
NIST 800-53 cataloged security and privacy controls into structured functional categories. I analyzed how these controls are designated to protect organizational operations:

| Control Type | Description | Example Implementation |
| :--- | :--- | :--- |
| **Administrative / Management** | Policies, procedures, and personnel security measures. | Security awareness training, acceptable use policies. |
| **Technical / Logical** | Controls executed by computer systems via hardware, software, or firmware. | Multi-Factor Authentication (MFA), Firewalls, EDR/SIEM rules. |
| **Physical** | Tangible barriers and countermeasures protecting assets from physical access. | CCTV cameras, biometric door locks, data center badges. |
| **Strategic** | Long-term high-level planning aligning security with business risk appetite. | Enterprise risk management planning, setting compliance baselines. |


### 3. Program Management Lifecycle
To manage an effective security program, organizations must move away from reactive fixes and adopt a continuous lifecycle approach:

```text
 ┌────────────┐      ┌────────────┐      ┌────────────┐      ┌────────────┐
 │  Discover  │ ──>  │    Map     │ ──>  │   Manage   │ ──>  │  Monitor   │
 └────────────┘      └────────────┘      └────────────┘      └────────────┘
