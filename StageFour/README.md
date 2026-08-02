> [!NOTE]
> **Stage Four is currently under active development.**

## [📌View the PREVIEW here!](https://cerealicious.github.io/CSP450/Preview.html)

### 🚀 Upcoming Features
* **Automated Generation:** A dedicated web interface is in progress to streamline and automate all Stage Four setup processes.
* **Documentation & Updates:** Detailed instructions and release notes will be published here soon.

Feel free to **⭐Star** or **Watch** this repository for upcoming releases and updates.
---
---
<img width="813" height="572" alt="StageFour-Diagram" src="https://github.com/user-attachments/assets/474bb795-a2e3-4fd4-86d4-c4f1ba84efbe" />

## 📋 Overview of Stage 4 Architecture Changes
In Stage 3, each tenant relied on a single core switch and single router. In Stage 4, all 4 group members (a full Pod) combine physical resources:

- **2x Aruba 6300 Core Switches** running **VRRP** (Virtual Router Redundancy Protocol) to share a virtual gateway per VLAN.
- **2x Aruba 2540/2530 Access Switches** trunking all 4 member VLANs across the rack.
- **4x Ubuntu FRR Routers** providing redundant default route origination back to SenecaNet/Internet.
---

## 📒 Execution Phases

### 🔹 Phase 1: Subnet & VRRP Gateway Calculation

- **Goal:** Map out unique IP addresses, Virtual Gateways (VRRP), and `/30` point-to-point router links for all 4 pod members.
- **Key Tasks:**
  - Keep each member's `/26` host/server network subnet derived from their Unique IDs.
  - Define Master Priority (`priority 10`) on Switch 1 vs Backup Priority (`priority 5`) on Switch 2 for all VLANs.
  - Establish the Virtual Primary IP  as the active gateway for all client/server devices across the pod.
  - Assign dedicated core IPs for Switch 1 and for Switch 2 per VLAN.

### 🔹 Phase 2: Aruba 6300 Core Switch Redundancy Configuration

- **Goal:** Configure both 6300 Core switches for Inter-VLAN routing, VRRP, dynamic DHCP pools, and OSPF Area 0 uplink routing.
- **Key Tasks:**
  - Provision all 4 member VLANs on both switches.
  - Configure VRRP groups (`vrrp <VLAN_ID> address-family ipv4`) on all SVIs.
  - Set up DHCP Server Pools referencing the Virtual VRRP Gateway as the `default-router`.
  - Configure interconnect trunk links (`1/1/1` and `1/1/2`) allowing all pod VLANs.
  - Enable global processes: `router ospf 1`, `router-id`, and `router vrrp enable`.

### 🔹 Phase 3: Aruba 2540 / 2530 Access Layer Trunking

- **Goal:** Extend all 4 member VLANs to physical access ports so any client/server VM can plug into any switch in the rack.
- **Key Tasks:**
  - Create all 4 pod VLANs across both access switches.
  - Configure uplink ports (`1/1/1`, `1/1/2`, `1/1/5`) as VLAN trunks allowing all member VLANs.
  - Map dedicated access ports (`1/1/3`, `1/1/4`, etc.) to specific member VLANs (`vlan access <VLAN_ID>`).
---
### 🔹 Phase 4: Upstream Linux Router (FRR) & OSPF Configuration

- **Goal:** Reconfigure all 4 Ubuntu Routers to advertise their point-to-point links and originate default routes into the pod.
- **Key Tasks:**
  - Update `vtysh` OSPF parameters to advertise `/30` point-to-point interfaces.
  - Configure `default-information originate` to inject the internet route (`0.0.0.0/0`) dynamically into OSPF Area 0.
  - Ensure `nftables` NAT rules remain active for outward internet translation.

### 🔹 Phase 5: Verification, Services Testing & Failover Validation

- **Goal:** Verify that web database queries succeed and validate zero-downtime VRRP failover during a physical hardware outage.
- **Key Tasks:**
  - **Baseline Test:** Run web database queries (Stage 3 app) across all pod member IPs.
  - **Master Check:** Execute `show vrrp` on both Core Switches to confirm Switch 1 is `MASTER` and Switch 2 is `BACKUP`.
  - **Failover Simulation:** Physically pull the power cable on Core Switch 1.
  - **Backup Verification:** Execute `show vrrp` on Core Switch 2 to confirm it transitioned to `MASTER`.
  - **Service Re-Test:** Perform database search queries again to prove continuity without network disruption.
  - **Failback Verification:** Reconnect power to Switch 1 and verify VRRP preempts back to original states.
----
### 🔹 Submission & Documentation

- **Goal:** Collect required proof screenshots and script files for GitHub submission.
- **Key Tasks:**
  - Capture `show vrrp` output on Switch 1 & 2 **BEFORE** power cut (2 Screenshots).
  - Capture `show vrrp` output on Switch 1 & 2 **AFTER** power cut (2 Screenshots).
  - Collate final customized configuration scripts for all 4 Aruba switches and FRR routers.
