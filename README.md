
# 🛡️ Network Security Lab 6.3  
### Firewall Policy Design, Implementation & Verification

<div align="center">

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-049FD9?logo=cisco&logoColor=white)](https://www.netacad.com/courses/packet-tracer)
[![Network Security](https://img.shields.io/badge/Network%20Security-ACL%20%7C%20Zone%20Segmentation-red)]()
[![Status](https://img.shields.io/badge/Status-Completed-success)]()
[![Author](https://img.shields.io/badge/Author-Sereyodam%20Chek-blue)]()

</div>

---

## 📑 Table of Contents
- [🌟 Overview](#-overview)
- [🎯 Objectives](#-objectives)
- [🗺️ Network Topology](#️-network-topology)
- [🖥️ Device Inventory](#️-device-inventory)
- [🔌 Cabling & Connectivity](#-cabling--connectivity)
- [🌐 IP Addressing Plan](#-ip-addressing-plan)
- [🔒 Security Policy Summary](#-security-policy-summary)
- [📂 Repository Structure](#-repository-structure)
- [🚀 Getting Started](#-getting-started)
- [✅ Verification Checklist](#-verification-checklist)
- [⚠️ Known Limitations](#️-known-limitations)
- [📜 Disclaimer](#-disclaimer)
- [👤 Author](#-author)

---

## 🌟 Overview

> **Mission:** Simulate a multi-zone university network (*Meanchey Digital University*) and implement a complete, enterprise-grade firewall policy using Cisco IOS named extended ACLs.

This hands-on network security laboratory covers zone segmentation, least-privilege access control, default-deny design, stateless return-traffic handling, SSH-restricted device management, and evidence-based security verification. 

The lab follows a rigorous, industry-aligned methodology: **Design → Baseline → Apply → Test → Correlate → Analyze → Rollback**.

---

## 🎯 Objectives

- 📐 **Policy Translation:** Translate business requirements into an ordered, justified firewall policy.
- 🧱 **Segmentation:** Implement zone-based network segmentation using VLANs and routed subinterfaces.
- 🛡️ **ACL Implementation:** Apply named extended ACLs with correct placement, direction, and wildcard masks.
- 🔒 **Least Privilege:** Enforce least privilege and default-deny at every trust boundary.
- 💻 **Secure Management:** Restrict device administration to a single authorized management host via SSH.
- 🧪 **Evidence-Based Testing:** Test and document both permitted and prohibited traffic flows.
- 🔄 **Change Control:** Practice safe configuration rollback and change management procedures.

---

## 🗺️ Network Topology

<div align="center">
  <img src="https://raw.githubusercontent.com/SereyodamChek/Network-Security-Firewall-Lab-6.3/main/lab.png" alt="Lab 6.3 Topology" width="100%" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"/>
</div>

### 📍 Zone Legend
| Zone | Description | Trust Level |
| :--- | :--- | :---: |
| 🌐 **Internet** | Untrusted external test source | 🔴 Untrusted |
| 🌉 **Transit** | ISP-to-firewall routed link | 🟡 Neutral |
| 🏛️ **DMZ** | Publicly exposed web server | 🟠 Semi-Trusted |
| 👔 **Administration (VLAN 10)** | Administrative workstation | 🟢 Trusted |
| 👥 **Staff (VLAN 20)** | Ordinary staff workstation | 🟢 Trusted |
| ⚙️ **Management (VLAN 99)** | Authorized network management station | 🟣 Highly Trusted |

---

## 🖥️ Device Inventory

| Device | Packet Tracer Model | Role & Responsibility |
| :--- | :--- | :--- |
| **ISP** | Router 2911 | Simulated Internet edge router |
| **EDGE-FW** | Router 2911 | Primary firewall/router enforcing all zone ACLs |
| **SW-DMZ** | Switch 2960 | Layer-2 DMZ segment switch |
| **SW-INSIDE** | Switch 2960 | Internal VLANs (10/20/99) + Management SVI |
| **WEB-DMZ** | Server-PT | Public HTTP/HTTPS web server |
| **Internet-PC** | PC-PT | Untrusted external test client |
| **ADMIN-PC** | PC-PT | Administration zone test host |
| **STAFF-PC** | PC-PT | Staff zone test host |
| **NMS-PC** | PC-PT | Authorized network management host |

---

## 🔌 Cabling & Connectivity

| # | Device A / Port | Device B / Port | Cable Type |
| :-: | :--- | :--- | :--- |
| 1 | `Internet-PC` / FastEthernet0 | `ISP` / G0/0 | Copper Straight-Through |
| 2 | `ISP` / G0/1 | `EDGE-FW` / G0/0 | Copper Straight-Through |
| 3 | `EDGE-FW` / G0/1 | `SW-DMZ` / G0/1 | Copper Straight-Through |
| 4 | `SW-DMZ` / F0/10 | `WEB-DMZ` / FastEthernet0 | Copper Straight-Through |
| 5 | `EDGE-FW` / G0/2 | `SW-INSIDE` / G0/1 | Copper Straight-Through *(802.1Q Trunk)* |
| 6 | `SW-INSIDE` / F0/10 | `ADMIN-PC` / FastEthernet0 | Copper Straight-Through |
| 7 | `SW-INSIDE` / F0/20 | `STAFF-PC` / FastEthernet0 | Copper Straight-Through |
| 8 | `SW-INSIDE` / F0/24 | `NMS-PC` / FastEthernet0 | Copper Straight-Through |

---

## 🌐 IP Addressing Plan

| Device / Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- |
| **Internet-PC** | `198.51.100.10` | `/24` | `198.51.100.1` |
| **ISP** `G0/0` | `198.51.100.1` | `/24` | — |
| **ISP** `G0/1` | `203.0.113.1` | `/30` | — |
| **EDGE-FW** `G0/0` *(outside)* | `203.0.113.2` | `/30` | — |
| **EDGE-FW** `G0/1` *(DMZ)* | `172.16.10.1` | `/24` | — |
| **WEB-DMZ** | `172.16.10.10` | `/24` | `172.16.10.1` |
| **EDGE-FW** `G0/2.10` *(Admin)* | `192.168.10.1` | `/24` | — |
| **ADMIN-PC** | `192.168.10.10` | `/24` | `192.168.10.1` |
| **EDGE-FW** `G0/2.20` *(Staff)* | `192.168.20.1` | `/24` | — |
| **STAFF-PC** | `192.168.20.10` | `/24` | `192.168.20.1` |
| **EDGE-FW** `G0/2.99` *(Mgmt)* | `192.168.99.1` | `/24` | — |
| **SW-INSIDE** `VLAN 99 SVI` | `192.168.99.2` | `/24` | `192.168.99.1` |
| **NMS-PC** | `192.168.99.10` | `/24` | `192.168.99.1` |

> **📌 Routing Notes:**  
> - **ISP:** Static routes to `172.16.10.0/24`, `192.168.10.0/24`, `192.168.20.0/24`, and `192.168.99.0/24` via `203.0.113.2` *(direct-routing simplification, no NAT)*.  
> - **EDGE-FW:** Default route (`0.0.0.0/0`) via `203.0.113.1`.

---

## 🔒 Security Policy Summary

### Boundary Rules
| Boundary | Default Action | Approved Exceptions |
| :--- | :---: | :--- |
| **Internet → DMZ** | 🚫 Deny | ✅ `TCP 80/443` to `WEB-DMZ` only |
| **Internet → Internal** | 🚫 Deny | ✅ Established/echo-reply returns only |
| **Admin/Staff → DMZ** | 🚫 Deny | ✅ `TCP 80/443` web access |
| **Admin/Staff → Internet**| 🚫 Deny | ✅ `TCP 80/443` + ICMP echo to test host |
| **DMZ → Internal** | 🚫 Deny | ✅ Narrow web-return traffic only; no initiated pivots |
| **Staff → Management** | 🚫 Deny | 🚫 None — explicit hard block |
| **Management → Devices**| 🚫 Deny | ✅ SSH from `NMS-PC` only, source-restricted via VTY ACL |

### 🏛️ Core Design Principles
- **Default-Deny First:** Written at every boundary, with justified exceptions layered on top.
- **First-Match Logic:** Narrow permits placed above broad denies.
- **Visibility:** Explicit `deny ip any any log` at the end of every ACL.
- **Anti-Spoofing:** RFC 1918 sources denied when arriving from the WAN interface.
- **Stateless Handling:** TCP `established` and exact ICMP `echo-reply` used for return traffic *(no `permit ip any any`)*.
- **Out-of-Band Mgmt:** SSH-only device management, restricted to a single authorized host.

### 🛡️ Named ACL Deployment Matrix
| ACL Name | Applied On | Direction |
| :--- | :--- | :---: |
| `OUTSIDE-IN` | `EDGE-FW G0/0` | Inbound |
| `DMZ-IN` | `EDGE-FW G0/1` | Inbound |
| `ADMIN-IN` | `EDGE-FW G0/2.10` | Inbound |
| `STAFF-IN` | `EDGE-FW G0/2.20` | Inbound |
| `MGMT-IN` | `EDGE-FW G0/2.99` | Inbound |
| `VTY-NMS-ONLY`| `line vty 0 4` *(EDGE-FW, SW-INSIDE)* | `access-class in` |

---

## 📂 Repository Structure

```text
📦 network-security-lab-6.3
 ┣ 📄 firewall-lab-6.3.pkt         # Cisco Packet Tracer project file
 ┣ 📄 README.md                    # Project documentation (this file)
 ┣ 📂 configs/                     # Full CLI configuration exports
 ┃ ┣ 📄 ISP.txt
 ┃ ┣ 📄 EDGE-FW.txt
 ┃ ┣ 📄 SW-INSIDE.txt
 ┃ ┗ 📄 SW-DMZ.txt
 ┣ 📂 screenshots/                 # Evidence captures
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ baseline-verification.png
 ┃ ┣ 🖼️ acl-verification.png
 ┃ ┣ 🖼️ allowed-flow-tests.png
 ┃ ┣ 🖼️ denied-flow-tests.png
 ┃ ┗ 🖼️ blocked-flow-simulation.png
 ┗ 📂 docs/
   ┗ 📄 policy-register.md         # Full ordered rule tables (optional)
```

---

## 🚀 Getting Started

1. **Install Cisco Packet Tracer**  
   Download via [Cisco Networking Academy](https://www.netacad.com/courses/packet-tracer) *(free registration required)*.
2. **Clone the Repository**  
   ```bash
   git clone https://github.com/SereyodamChek/Network-Security-Firewall-Lab-6.3.git
   cd Network-Security-Firewall-Lab-6.3
   ```
3. **Open the Project**  
   Launch `firewall-lab-6.3.pkt` in Cisco Packet Tracer.
4. **Review Configurations**  
   Navigate to the `configs/` directory to review the exact CLI commands applied to each device.

---

## ✅ Verification Checklist

- [x] **Baseline:** Routing and connectivity confirmed *before* any ACL application.
- [x] **Deployment:** All five named ACLs created and applied to the correct interface/direction.
- [x] **Attachment:** `show ip interface` used to confirm ACL attachment.
- [x] **Allowed Flows:** Minimum 4 tests passed *(Internet→DMZ web, Admin/Staff→DMZ, ICMP diagnostics, SSH mgmt)*.
- [x] **Denied Flows:** Minimum 4 tests passed *(Internet→internal, Staff→Management, DMZ pivot attempts)*.
- [x] **Correlation:** Blocked flow correlated with ACL hit counter / Simulation Mode packet trace.
- [x] **Gap Analysis:** Performed and corrective changes retested.
- [x] **Rollback:** Safe rollback procedure documented *(detach ACLs before deletion, restore baseline)*.

---

## ⚠️ Known Limitations

> **ℹ️ Technical Context for Evaluators:**
> - **Stateless Nature:** Cisco IOS extended ACLs are **stateless** packet filters, not full stateful firewalls. The `established` keyword only checks TCP ACK/RST flags, not true connection state.
> - **Return Traffic:** ICMP and UDP require explicit protocol-specific return rules; there is no automatic return-path creation.
> - **No NAT:** This lab routes RFC 1918 addresses directly through the simulated ISP. This is a deliberate simplification for testing purposes and **not** a production design.
> - **Emulation Limits:** Packet Tracer may not fully emulate ACL logging or all counter behaviors. Where unsupported, this is explicitly noted rather than fabricated.

---

## 📜 Disclaimer

This project uses fictional institution names, RFC 1918 private address ranges, and RFC 5737 documentation ranges. It was built and tested **exclusively** in an isolated Packet Tracer lab environment. It does not represent any real organization, live network, or production system.

---

## 👤 Author

**Sereyodam Chek**  
🎓 *Course:* Advanced Data Communication  
🔬 *Lab:* 6.3 — Network Security, Management, and Optimisation  

<div align="center">
  <sub>Built with 🛡️ and ☕ for educational excellence.</sub>
</div>


