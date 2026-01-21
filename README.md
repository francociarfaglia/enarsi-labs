# Network Architecture Repository

This repository is a specialized technical resource dedicated to the **implementation, optimization, and troubleshooting** of network architectures, strictly aligned with the Cisco ENARSI (300-410) certification objectives. 

It serves as a progressive portfolio documenting the resolution of complex routing scenarios and infrastructure services in simulated corporate environments.

---

## Project Focus: ENARSI Domains
Each lab is designed to validate technical knowledge in the following areas:

* **Layer 3 Technologies:** Advanced tuning and troubleshooting of **EIGRP, OSPF (v2/v3), and BGP**.
* **Route Manipulation:** Control of traffic using **Redistribution, Route-Maps, Prefix-Lists, and Policy-Based Routing (PBR)**.
* **VPN Services & Infrastructure:** Implementation and diagnostics of **MPLS, DMVPN, and IPv6** transitions.
* **Infrastructure Security & Services:** Hardening the control plane (**CoPP**), securing management, and troubleshooting services like SNMP, DHCP, and IP SLA.

---

## Repository Structure & Methodology
Every topology includes visual and technical evidence of the validation process:

| File/Folder | Purpose |
| :--- | :--- |
| **topology.md** | Technical documentation, objectives, and verification steps. |
| **/image** | Topology diagrams and CLI screenshots. |
| **/config** | YAML files used to bootstrap/import the lab into CML. |

---

##  How to Use This Repository

### 1. Clone the Project
```
git clone https://github.com/franco/enarsi_labs.git
```
### 2. Import Lab:

Log into your CML instance, click **Import**, and select the `.yaml` file found in the `/config` folder of the specific lab.
