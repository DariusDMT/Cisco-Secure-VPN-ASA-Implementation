# Cisco-Secure-VPN-ASA-Implementation
This repository contains a full-scale network implementation designed in Cisco Packet Tracer. It simulates a secure connection between a Corporate Headquarters (HQ) and a Remote Branch office using a Site-to-Site IPsec VPN tunnel.

Network Architecture (Color-Coded Segments)

<p align="center">
  <img src="topology/image_e0f5d7.png" alt="Network Topology" width="800">
</p>
The topology is divided into three logical zones for clarity:

## 🟡 **Yellow Zone (LAN Infrastructure)**:

-HQ Site (Left): Features a hierarchical Core-Access design with Inter-VLAN routing for departments like Admin, HR, and IT Support.

-Branch Site (Right): A remote office setup connecting the Branch Manager and local staff to the central resources.



## 🔵 **Blue Zone (Security & NAT)**:

-The Cisco ASA Firewall acts as the security perimeter.

-Implements NAT (Network Address Translation) to 200.0.0.1, ensuring internal IP addresses are shielded before entering the public domain.

💡 Key Design Choice:
I offloaded the IPsec Encryption to the router to ensure the Firewall's performance remains optimal for traffic inspection.



## 🔴 **Red Zone (Public Transport & IPsec Tunnel)**:

-Represents the untrusted ISP environment.

-The IPsec VPN Tunnel is established here, providing end-to-end encryption (AES-256) and data integrity for all traffic moving between sites.



## **Key Technologies Implemented**
-Cisco ASA 5506-X Configuration: Security levels, ACLs, and NAT policies.

-Site-to-Site IPsec VPN: IKEv1 Phase 1 (ISAKMP) and Phase 2 (IPsec) negotiations.

-NAT Traversal: Handling VPN interesting traffic over translated IP addresses.

-VLAN/Inter-VLAN Routing: Departmental segmentation using Sub-interfaces and Core Switching.

## 📑 Master Device Inventory & Addressing

The following table centralizes all device configurations, including specific interfaces, IP assignments, and High Availability (HSRP) priority settings.

### 🏢 Headquarters (HQ) - Internal & Edge Infrastructure
| Device Name | Interface | IP Address | Subnet Mask | Default Gateway | Role / Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **PC-ADMIN-HQ** | Fa0 | `10.0.10.10` | `255.255.255.0` | `10.0.10.2` | Admin Host (VLAN 10) |
| **PC-IT-SUPPORT** | Fa0 | `10.0.10.11` | `255.255.255.0` | `10.0.10.2` | IT Support Host (VLAN 10) |
| **HR-1** | Fa0 | `10.0.20.10` | `255.255.255.0` | `10.0.20.2` | HR Dept Host (VLAN 20) |
| **HR-2** | Fa0 | `10.0.20.11` | `255.255.255.0` | `10.0.20.2` | HR Dept Host (VLAN 20) |
| **GUEST-PC** | Fa0 | `10.0.30.10` | `255.255.255.0` | `10.0.30.2` | Guest Network (VLAN 30) |
| **CORE-1** | Vlan 10 | `10.0.10.2` | `255.255.255.0` | `10.0.10.1`* | **HSRP Priority 110** (Primary) |
| **CORE-1** | Vlan 20 | `10.0.20.2` | `255.255.255.0` | `10.0.20.1`* | **HSRP Priority 110** (Primary) |
| **CORE-1** | Vlan 30 | `10.0.30.2` | `255.255.255.0` | `10.0.30.1`* | **HSRP Priority 110** (Primary) |
| **CORE-1** | Vlan 100 | `10.0.100.2`| `255.255.255.0` | `10.0.100.1`*| **HSRP Priority 110** (Primary) |
| **CORE-1** | G0/1 | `192.168.1.2`| `255.255.255.252`| - | Link to ASA (Outside) |
| **CORE-2** | Vlan 10 | `10.0.10.3` | `255.255.255.0` | `10.0.10.1`* | **HSRP Priority 100** (Standby) |
| **CORE-2** | Vlan 20 | `10.0.20.3` | `255.255.255.0` | `10.0.20.1`* | **HSRP Priority 100** (Standby) |
| **CORE-2** | Vlan 30 | `10.0.30.3` | `255.255.255.0` | `10.0.30.1`* | **HSRP Priority 100** (Standby) |
| **ASA FW** | G1/2 | `192.168.1.1`| `255.255.255.252`| - | Inside Interface (Sec: 100) |
| **ASA FW** | G1/1 | `200.0.0.1` | `255.255.255.252`| - | Outside Interface (Sec: 0) |
| **HQ-EDGE** | G0/0/1 | `200.0.0.2` | `255.255.255.252`| `200.0.0.1` | Internal Link towards ASA |
| **HQ-EDGE** | G0/0/0 | `82.210.10.2`| `255.255.255.252`| `82.210.10.1` | **VPN Head-end (Public IP)** |

> \* *Note: `10.0.x.1` represents the Virtual IP (VIP) managed by HSRP across the Core switches.*

### 🏪 Branch Office - Remote Infrastructure
| Device Name | Interface | IP Address | Subnet Mask | Default Gateway | Role / Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Branch-Mgr** | Fa0 | `10.1.100.10`| `255.255.255.0` | `10.1.100.1` | Manager Workstation |
| **PC1** | Fa0 | `10.1.100.11`| `255.255.255.0` | `10.1.100.1` | Staff Host (VLAN 100) |
| **PC2** | Fa0 | `10.1.100.12`| `255.255.255.0` | `10.1.100.1` | Staff Host (VLAN 100) |
| **BRANCH R** | G0/0/1.100| `10.1.100.1` | `255.255.255.0` | - | Sub-int (VLAN 100 Gateway) |
| **BRANCH R** | G0/0/1.101| `10.1.101.1` | `255.255.255.0` | - | Sub-int (VLAN 101 Gateway) |
| **BRANCH R** | G0/0/0 | `188.5.5.2` | `255.255.255.252`| `188.5.5.1` | **VPN Endpoint (Public IP)** |

---

## 🔒 Security & VPN Implementation

### 1. VPN Site-to-Site (IPsec)
The tunnel ensures data confidentiality between HQ and Branch.
* **Phase 1 (ISAKMP):** AES-256 encryption, SHA hashing, DH Group 2.
* **Phase 2 (IPsec):** ESP with AES encryption and SHA authentication.

### 2. Cisco ASA & NAT Traversal
The **Cisco ASA Firewall** is positioned behind the Edge Router to filter internal traffic. A major technical challenge was **NAT Traversal**:
* The ASA translates internal IPs to `200.0.0.1`.
* The **HQ-EDGE** router was configured with specific Access Control Lists (ACLs) to recognize and encrypt this translated traffic.

### 3. High Availability (HSRP)
To prevent network downtime, **HSRP** was implemented on the Core Switches.
* **Active Gateway:** Core-1 (Priority 110).
* **Standby Gateway:** Core-2 (Priority 100).
* This ensures that if one switch fails, the Virtual IP (`10.0.x.1`) remains reachable, keeping users online.

---

## 🛠️ Troubleshooting Log
* **ISP Routing Issue:** Resolved by implementing static routes on the ISP router to allow reachability between the two public WAN subnets (`82.0.0.0` and `188.0.0.0`).
* **SPI Mismatch:** Fixed IPsec synchronization issues in Packet Tracer by clearing crypto sessions and forcing a re-negotiation through ICMP keep-alives.
* **ACL Refinement:** Adjusted the VPN "interesting traffic" list to include the NAT-ed pool from the ASA Firewall.

## 📂 Project Files
* `/configs`: Contains `.txt` files with `show run` outputs for all major devices.
* `/topology`: Contains high-resolution diagrams and color-coded segmentation maps.
* `/lab`: The original `.pkt` (Cisco Packet Tracer) file for simulation.
