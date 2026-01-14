# Network Architecture Repository

This repository is a specialized technical resource featuring network architectures aligned with **CCNP Enterprise** certification objectives. It serves as a centralized hub for the design, validation, and documentation of complex corporate infrastructures.

---

## Project Scope
Each simulation environment is built to address key domains of the ENCOR and ENARSI exams:

* **Advanced Routing:** Implementation and tuning of OSPF, EIGRP, and MP-BGP.
* **WAN Connectivity:** Enterprise edge solutions including DMVPN and site-to-site tunnels.
* **Security:** Infrastructure hardening, CoPP, and secure management planes.
* **Automation:** Utilizing CML features for lab orchestration and node configuration.

---

## Repository Structure
To maintain consistency and ease of use, every topology is organized into a dedicated folder with the following standardized sub-folders:

| File/Folder | Purpose |
| :--- | :--- |
| **topology.md** | Detailed technical documentation, objectives, and verification steps. |
| **/image** | High-resolution topology diagrams and CLI screenshots. |
| **/config** | YAML files used to bootstrap/import the lab into CML. |

---

##  How to Use This Repository

### 1. Clone the Project
```
git clone [https://github.com/francociarfaglia/topologies.git](https://github.com/francociarfaglia/topologies.git)
```
### 2. Import Lab:

Log into your CML instance, click **Import**, and select the `.yaml` file found in the `/config` folder of the specific lab.

---

### Project Goals

* **Design:** Professional-grade corporate infrastructure layouts.
* **Validation:** Step-by-step verification of routing and switching logic.
* **Documentation:** High-quality diagrams and technical summaries for study and reference.
