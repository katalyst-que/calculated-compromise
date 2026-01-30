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
    
    subgraph "Azure Resource Group"
        NSG_Honey -->|Allow All Inbound| VM_Honey[CapstoneHoney VM <br/> (T-Pot Hive)]
        NSG_Sec -->|Filtered Inbound| VM_Sec[Secured Service VM <br/> (Ubuntu 22.04)]
    end
    
    VM_Honey -->|Threat Intel| Analysis[Kibana Analysis]
    Analysis -->|Block Rules| NSG_Sec
