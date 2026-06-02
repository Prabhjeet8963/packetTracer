# Impact of Subnetting Design on Network Performance

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco_Packet_Tracer-v8.0+-blue.svg?style=flat-square&logo=cisco)](https://www.netacad.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)
[![Routing](https://img.shields.io/badge/Routing-OSPF_Area_0-red.svg?style=flat-square)](https://en.wikipedia.org/wiki/Open_Shortest_Path_First)
[![Topology](https://img.shields.io/badge/Topology-FLSM%20%7C%20VLSM%20%7C%20RoaS-orange.svg?style=flat-square)](#address-plan)

Supplementary Cisco Packet Tracer simulation files for the research paper:

> **Impact of Subnetting Design on Network Performance**  
> **Author:** Mrigank Singh (SAP ID: 590012123)  
> **Affiliation:** B.Tech Computer Science, UPES Dehradun — 2024–2028  

---

## Table of Contents
- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Address Plan](#address-plan)
  - [Scenario A — FLSM](#scenario-a--flsm)
  - [Scenario B — VLSM](#scenario-b--vlsm)
  - [Scenario C — Over-segmented /28](#scenario-c--over-segmented-28)
- [Network Topologies](#network-topologies)
- [Routing Configuration](#routing-configuration)
- [Installation & Usage](#installation--usage)
- [Verification & Commands](#verification--commands)
- [Key Experimental Results](#key-experimental-results)
- [Requirements](#requirements)
- [Master Scenario D: Integrated Network (Extra)](#master-scenario-d-comprehensive-enterprise-network)
- [Citation](#citation)
- [License](#license)

---

## Overview

This repository hosts three pre-configured Cisco Packet Tracer (`.pkt`) files designed to analyze how different subnetting strategies (FLSM, VLSM, and Over-segmented Router-on-a-Stick) impact overall network performance. The simulations measure metric variables including routing table overhead, OSPF adjacency, propagation latency (RTT), and packet loss under realistic topology conditions.

---

## Repository Structure

The repository contains four main simulation files:

| File | Scenario | Strategy | Subnets | Routers | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [`scenario-a-flsm.pkt`](file:///c:/Users/LENOVO/Desktop/Subnetting/scenario-a-flsm.pkt) | **Scenario A** | Fixed-Length Subnet Mask (FLSM) | $4 \times \text{/26}$ | 1 | Uniform subnetting using a static mask. |
| [`scenario-b-vlsm.pkt`](file:///c:/Users/LENOVO/Desktop/Subnetting/scenario-b-vlsm.pkt) | **Scenario B** | Variable-Length Subnet Mask (VLSM) | Mixed (`/27`, `/26`, `/25`) | 2 | Variable sizing based on host demand with OSPF routing. |
| [`scenario-c-oversegmented.pkt`](file:///c:/Users/LENOVO/Desktop/Subnetting/scenario-c-oversegmented.pkt) | **Scenario C** | Over-segmented (RoaS) | $8 \times \text{/28}$ | 1 | Router-on-a-stick topology over 8 VLANs via an 802.1Q trunk. |
| [`scenario-d-comprehensive.pkt`](file:///c:/Users/LENOVO/Desktop/Subnetting/scenario-d-comprehensive.pkt) | **Scenario D** | Comprehensive (VLSM/NAT/DHCP/OSPF) | Mixed | 3 | Full-stack topology including NAT, DHCP, OSPF, and Servers. |

---

## Address Plan

All scenarios allocate addresses from the standard private Class C network space `192.168.1.0/24`. Scenario B also utilizes a `10.0.0.0/30` subnet for its point-to-point link.

### Scenario A — FLSM
All subnets use a uniform `/26` mask (supporting up to 62 hosts per subnet).

| Subnet ID | Network Address | Gateway IP | Connected Hosts |
| :---: | :--- | :--- | :--- |
| **1** | `192.168.1.0/26` | `192.168.1.1` | PC0, PC1 |
| **2** | `192.168.1.64/26` | `192.168.1.65` | PC2, PC3 |
| **3** | `192.168.1.128/26` | `192.168.1.129` | PC4, PC5 |
| **4** | `192.168.1.192/26` | `192.168.1.193` | PC6, PC7 |

### Scenario B — VLSM
Addresses allocated dynamically based on specific host count demands to minimize waste.

| Subnet/LAN | Network Address | Gateway IP | Configured Router | Assigned Hosts |
| :--- | :--- | :--- | :--- | :--- |
| **LAN-A** | `192.168.1.0/27` | `192.168.1.1` | Router0 | PC0, PC1 |
| **LAN-B** | `192.168.1.32/27` | `192.168.1.33` | Router0 | PC2, PC3 |
| **LAN-C** | `192.168.1.64/26` | `192.168.1.65` | Router1 | PC4, PC5, PC6, PC7 |
| **LAN-D** | `192.168.1.128/25` | `192.168.1.129` | Router1 | PC8, PC9 |
| **P2P Link**| `10.0.0.0/30` | — | R0 $\leftrightarrow$ R1 | Serial WAN Interface Link |

### Scenario C — Over-segmented /28 (Router-on-a-Stick)
Uses a single router interface subdivided into subinterfaces using 802.1Q encapsulation to route between 8 VLANs.

| VLAN ID | Subnet Network | Gateway IP | Active Host |
| :---: | :--- | :--- | :--- |
| **VLAN 1** | `192.168.1.0/28` | `192.168.1.1` | PC0 |
| **VLAN 2** | `192.168.1.16/28` | `192.168.1.17` | PC1 |
| **VLAN 3** | `192.168.1.32/28` | `192.168.1.33` | PC2 |
| **VLAN 4** | `192.168.1.48/28` | `192.168.1.49` | PC3 |
| **VLAN 5** | `192.168.1.64/28` | `192.168.1.65` | PC4 |
| **VLAN 6** | `192.168.1.80/28` | `192.168.1.81` | PC5 |
| **VLAN 7** | `192.168.1.96/28` | `192.168.1.97` | PC6 |
| **VLAN 8** | `192.168.1.112/28` | `192.168.1.113` | PC7 |

---

## Network Topologies

Here are structural diagrams of the simulated environments:

### Scenario A: FLSM
```mermaid
graph TD
    subgraph Scenario A — FLSM /26
        R1[Router 0]
        R1 --- Sub1["Subnet 1 (.0/26)"]
        R1 --- Sub2["Subnet 2 (.64/26)"]
        R1 --- Sub3["Subnet 3 (.128/26)"]
        R1 --- Sub4["Subnet 4 (.192/26)"]
        
        Sub1 --- PC0(PC0) & PC1(PC1)
        Sub2 --- PC2(PC2) & PC3(PC3)
        Sub3 --- PC4(PC4) & PC5(PC5)
        Sub4 --- PC6(PC6) & PC7(PC7)
    end
    style R1 fill:#d1e7dd,stroke:#0f5132,stroke-width:2px;
```

### Scenario B: VLSM (Multi-Router Link)
```mermaid
graph LR
    subgraph Scenario B — VLSM
        subgraph Router0 LANs
            PC0(PC0) & PC1(PC1) --- LA["LAN-A (.0/27)"]
            PC2(PC2) & PC3(PC3) --- LB["LAN-B (.32/27)"]
            LA --- R0[Router0]
            LB --- R0
        end

        R0 <-->|Serial Link: 10.0.0.0/30| R1[Router1]

        subgraph Router1 LANs
            R1 --- LC["LAN-C (.64/26)"]
            R1 --- LD["LAN-D (.128/25)"]
            LC --- PC4(PC4) & PC5(PC5) & PC6(PC6) & PC7(PC7)
            LD --- PC8(PC8) & PC9(PC9)
        end
    end
    style R0 fill:#d1e7dd,stroke:#0f5132,stroke-width:2px;
    style R1 fill:#d1e7dd,stroke:#0f5132,stroke-width:2px;
```

### Scenario C: Over-segmented (Router-on-a-Stick)
```mermaid
graph TD
    subgraph Scenario C — RoaS
        R[Router 0] <==>|802.1Q Trunk Trunk| S[Switch 0]
        S --- V1[VLAN 1] --- PC0(PC0)
        S --- V2[VLAN 2] --- PC1(PC1)
        S --- V3[VLAN 3] --- PC2(PC2)
        S --- V4[VLAN 4] --- PC3(PC3)
        S --- V5[VLAN 5] --- PC4(PC4)
        S --- V6[VLAN 6] --- PC5(PC5)
        S --- V7[VLAN 7] --- PC6(PC6)
        S --- V8[VLAN 8] --- PC7(PC7)
    end
    style R fill:#d1e7dd,stroke:#0f5132,stroke-width:2px;
    style S fill:#cff4fc,stroke:#087990,stroke-width:2px;
```

---

## Routing Configuration

All networks are configured to run **OSPFv2 (Process ID 1, Area 0)** to distribute routing information.

- **Scenario A**: Configured with a single router; all subnets are directly connected. Hence, no active dynamic OSPF neighbors are established.
- **Scenario B**: Adjacency is negotiated and fully established over the serial interface link between Router0 and Router1 (transitioned to the `FULL` state).
- **Scenario C**: Subnets exist on virtual interfaces (subinterfaces) of the single router. No dynamic OSPF neighbors are present.

---

## Installation & Usage

To view and interact with these scenarios:

1. **Install Cisco Packet Tracer**: Download version `8.0` or higher from the [Cisco Networking Academy](https://www.netacad.com/).
2. **Clone the Repository**:
   ```bash
   git clone https://github.com/Mrigank005/subnetting-performance-study.git
   cd subnetting-performance-study
   ```
3. **Open Simulation Files**: Launch Packet Tracer and navigate to `File -> Open` to select any of the `.pkt` files.

---

## Verification & Commands

You can verify connectivity and view network states using the Packet Tracer command lines.

### 1. Test Connectivity (End-to-End Ping)
On any end host (e.g., PC0), open **Desktop -> Command Prompt** and run:
```cmd
ping 192.168.1.70
```

### 2. Verify Routing Tables
Click on any Router, navigate to the **CLI** tab, and enter:
```ios
Router# show ip route
```

### 3. Check OSPF Neighbor States (Scenario B)
Verify dynamic link state routing adjacent neighbors by running:
```ios
Router# show ip ospf neighbor
```

---

## Key Experimental Results

*Data extracted directly from research paper measurements:*

| Evaluated Metric | Scenario A (FLSM) | Scenario B (VLSM) | Scenario C (RoaS) |
| :--- | :---: | :---: | :---: |
| **Routing Table Entries** | 8 | 8 (Router0) | 16 |
| **Distinct Subnet Masks** | 2 | 4 | 2 |
| **OSPF Neighbors** | 0 | 1 (`FULL` state) | 0 |
| **Ping TTL** | 127 | 126 | 127 |
| **Average Round Trip Time (RTT)** | <1ms | 3ms | <1ms |
| **Packet Loss Rate** | 0% | 0% | 0% |

---

## Requirements

- **Software**: Cisco Packet Tracer **v8.0** or later.
- **Compatibility Note**: Older versions (e.g., v7.x or below) are not backwards-compatible and will fail to parse the XML structure of the `.pkt` files.

---

## Master Scenario D: Comprehensive Enterprise Network

In addition to Scenarios A, B, and C, we have designed a **Master Scenario D** configuration that builds an integrated enterprise topology. This scenario is highly recommended for advanced study as it incorporates:
- **IP Addressing & Subnetting (VLSM)**
- **OSPF Dynamic Routing & Default Route Propagation**
- **Dynamic DHCP & local Address Resolution Protocol (ARP)**
- **Network Address Translation (NAT/PAT)** on the edge boundary
- **TCP/UDP Socket Ports** for DNS (UDP 53) and Web Server (TCP 80) access

For full details, topology diagrams, and step-by-step Cisco IOS CLI configuration scripts, see the [Master Scenario Configuration Guide](file:///c:/Users/LENOVO/Desktop/Subnetting/master_scenario_config.md).

---

## Citation

If you use or build upon these topologies for academic studies, please cite the paper:

```bibtex
@article{singh2026impact,
  author    = {Mrigank Singh},
  title     = {Impact of Subnetting Design on Network Performance},
  institution = {University of Petroleum and Energy Studies (UPES), Dehradun},
  year      = {2026},
  url       = {https://github.com/Mrigank005/subnetting-performance-study}
}
```

Standard Text Citation:
> Singh, M. (2026). *Impact of Subnetting Design on Network Performance*. UPES Dehradun. Available at: https://github.com/Mrigank005/subnetting-performance-study

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
