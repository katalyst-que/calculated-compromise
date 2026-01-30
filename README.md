# Calculated Compromise: Characterizing Cloud-Based Cyber Conflicts
### Cloud-Based Deception Environment & Threat Intelligence Analysis

![Status: Completed](https://img.shields.io/badge/Status-Completed-success)
![Platform: Azure](https://img.shields.io/badge/Platform-Microsoft%20Azure-blue)
![Stack: T--Pot](https://img.shields.io/badge/Honeypot-T--Pot%2024.04-orange)
![Tools: Docker](https://img.shields.io/badge/Tools-Docker%20|%20Elastic%20Stack-blueviolet)
![Compliance: OpenSCAP](https://img.shields.io/badge/Compliance-OpenSCAP-green)

## Executive Summary
This project addresses a critical visibility gap in Small-to-Medium Enterprise (SME) cloud environments: the lack of "opportunistic" threat detection. By deploying a **T-Pot "Hive" multi-sensor honeypot** alongside a simulated production asset in Microsoft Azure, this project operationalized real-world threat intelligence to shift security posture from reactive to proactive.

Over a 7-day data collection period, the system captured **478,000+ unauthorized interaction attempts**. This intelligence was immediately analyzed and applied to harden a "Secured VM," demonstrating that environment-specific threat mitigation often extends beyond generalized compliance benchmarks (CIS).

---

## Technical Architecture

The infrastructure was deployed in **Microsoft Azure** using a dual-VM segmentation strategy to isolate the honeypot "sensor" from the simulated "production" service.

### Components
| Component | Spec | Role |
| :--- | :--- | :--- |
| **CapstoneHoney** | Debian 12, 32GB RAM, 128GB Disk | **The Sensor.** Hosts the T-Pot Hive (Dockerized honeypots + ELK Stack). |
| **Secured-VM** | Ubuntu 22.04, 8GB RAM | **The Target.** Simulates a production web server for hardening validation. |
| **Azure NSG** | Network Security Groups | **The Control.** Manages ingress/egress traffic rules based on intelligence. |

### Architecture Diagram
```mermaid
graph TD
    Internet((Internet)) -->|Attack Traffic| NSG_Honey[Azure NSG: Honeypot]
    Internet -->|Web Traffic| NSG_Sec[Azure NSG: Secured VM]
    
    subgraph Azure_Resource_Group [Azure Resource Group]
        NSG_Honey -->|Allow All Inbound| VM_Honey["CapstoneHoney VM (T-Pot Hive)"]
        NSG_Sec -->|Filtered Inbound| VM_Sec["Secured Service VM (Ubuntu 22.04)"]
    end
    
    VM_Honey -->|Threat Intel| Analysis[Kibana Analysis]
    Analysis -->|Block Rules| NSG_Sec

## Deployment & Analysis Workflow

### Phase 1: Infrastructure Provisioning & Deployment
The project executed a "Plan-Do-Check-Act" lifecycle, starting with the provisioning of the Azure environment.
* **Infrastructure:** Deployed a **Standard D4s v3** virtual machine. Initial stress testing with 8GB RAM caused system instability and boot loops; the instance was vertically scaled to **32GB RAM** to support the T-Pot "Hive" stack.
* **Secure Access:** Administrative access was restricted via SSH and the T-Pot Web Interface (Port 64297) was locked to a single authorized IP address.
* **Sensor Exposure:** To maximize the attack surface for data collection, a low-priority Azure Network Security Group (NSG) rule was configured to **Allow Any/Any** inbound traffic.

![T-Pot Web Dashboard Verification](docs/screenshots/Figure_A-13.png)
*Successful deployment of the T-Pot 24.04.1 "Hive" stack, verified via the web dashboard.*

---

### Phase 2: Threat Intelligence Analysis
The honeypot was left operational for a **7-day data collection period**, aggregating real-time attack telemetry into the Elastic Stack.
* **Volume:** The system recorded approximately **478,000 events**.
* **Top Targets:** The most frequently targeted ports were **22 (SSH), 5060 (SIP), 8728, 80 (HTTP), and 25 (SMTP)**.
* **Behavioral Signature:** Suricata identified over **8,000 inbound requests** matching the `User-Agent: python-requests` signature. This indicated automated scripts attempting post-exploitation activity, such as downloading second-stage payloads or establishing C2 callbacks.

![Global Attack Map](docs/screenshots/Figure_B-1.png)
*Real-time global attack activity detected within seconds of deployment.*

![Attack Signatures](docs/screenshots/Figure_B-4.png)
*Suricata analysis revealing the high volume of Python-based user-agent hits alongside VNC and ICMP traffic.*

---

### Phase 3: Hardening & Validation
The intelligence gathered was applied to a separate **Secured Ubuntu 22.04 VM** to test the efficacy of specific remediation strategies.

#### Stage A: Compliance-Based Hardening (OpenSCAP)
An initial OpenSCAP audit against the **CIS Level 1 Workstation Benchmark** established a baseline compliance score of **66.38%**, identifying 5 high-severity failures.
* **Action:** Using the Azure Serial Console, the `sshd_config` was modified to disable `PermitRootLogin` and `PermitEmptyPasswords`.
* **Result:** A follow-up audit confirmed the remediation, raising the compliance score to **68.23%**.

![OpenSCAP Remediation](docs/screenshots/Figure_C-6.png)
*Follow-up OpenSCAP scan showing reduced high-severity findings after SSH hardening.*

#### Stage B: Intelligence-Driven Hardening
To address the specific threats found in Phase 2, custom security controls were applied that went beyond standard compliance benchmarks.
* **Ingress Filtering:** Azure NSG rules were created to block the **Top 10 Aggressive IPs** identified by the honeypot (sourced from Kibana analysis).
* **Egress Filtering:** An "Anti-Callback" rule was implemented to **Deny Outbound Traffic on Ports 80/443**. This directly mitigated the risk of the `python-requests` malware downloading payloads or contacting Command & Control servers.

![NSG Block Logic](docs/screenshots/Figure_C-8.png)
*Implementation of outbound NSG rules to restrict suspicious Python-based callback traffic.*

---

### Outcome & Compliance Insight
A final OpenSCAP audit was conducted after applying the intelligence-driven controls. The compliance score remained unchanged at **68.23%**.

> **Critical Insight:** This result demonstrated that while standard compliance frameworks (CIS) ensure configuration hygiene, they do not account for dynamic network-level defenses. The "Anti-Callback" rules neutralized the primary active threat vector without altering the static compliance score, highlighting the necessity of pairing compliance with active threat intelligence.
