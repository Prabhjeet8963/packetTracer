# Master Scenario D: Integrated Enterprise Network Configuration

This document provides the architectural design and complete Cisco IOS configuration commands to build a comprehensive, multi-service enterprise topology in Cisco Packet Tracer. 

This scenario integrates **Subnetting (VLSM)**, **OSPF Routing**, **NAT/PAT**, **DHCP/ARP services**, and transport-layer socket address binding (**TCP/UDP**).

---

## 1. Network Topology

```mermaid
graph TD
    subgraph LAN Subnets (Private Space)
        PC1(PC1 - DHCP Client) --- Switch1[Internal Switch]
        PC2(PC2 - DHCP Client) --- Switch1
        LocalWeb(Local Web Server<br>192.168.1.30/27<br>TCP Port 80) --- Switch1
    end

    Switch1 ===|G0/0| HQ[HQ-Router]
    
    subgraph Enterprise Core (OSPF Area 0)
        HQ ===|S0/1/0: 10.0.0.4/30| Edge[Edge-Router]
    end

    subgraph Simulated Internet (Public Space)
        Edge ===|G0/1: 203.0.113.2/30<br>NAT Outside| ISP[ISP-Router]
        ISP ===|G0/0: 8.8.8.1/24| InternetServer(Public DNS/Web Server<br>8.8.8.8<br>TCP 80, UDP 53)
    end

    style HQ fill:#d1e7dd,stroke:#0f5132,stroke-width:2px;
    style Edge fill:#f8d7da,stroke:#842029,stroke-width:2px;
    style ISP fill:#fff3cd,stroke:#664d03,stroke-width:2px;
```

---

## 2. IP Addressing Plan (VLSM)

We use the base subnet `192.168.1.0/24` for internal subnets and a public pool `203.0.113.0/30` for the simulated WAN link.

| Network/Link | Size | Subnet Mask | Network Address | Gateway/Interface | Device / Host |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **Enterprise LAN** | 30 Hosts | `255.255.255.224` (/27) | `192.168.1.0/27` | `192.168.1.1` | PCs (DHCP), Local Web (`.30`) |
| **HQ $\leftrightarrow$ Edge Link** | 2 Hosts | `255.255.255.252` (/30) | `10.0.0.4/30` | `.5` (HQ), `.6` (Edge) | Serial 0/1/0 |
| **Edge $\leftrightarrow$ ISP Link** | 2 Hosts | `255.255.255.252` (/30) | `203.0.113.0/30` | `.1` (ISP), `.2` (Edge) | GigabitEthernet 0/1 |
| **Google/Public LAN** | 254 Hosts | `255.255.255.0` (/24) | `8.8.8.0/24` | `8.8.8.1` | Public Web/DNS Server (`8.8.8.8`) |

---

## 3. Layer 4 Transport: Ports & Socket Addresses

Sockets uniquely identify network connections using:
$$\text{Socket Address} = \text{IP Address} + \text{Protocol} + \text{Port Number}$$

Below are the key sockets active in this network during communication:

| Service / Client | Protocol | Port Number | Socket Address |
| :--- | :---: | :---: | :--- |
| **DNS Server (Public)** | UDP | 53 | `8.8.8.8:53` (UDP) |
| **Web Server (Public)** | TCP | 80 | `8.8.8.8:80` (TCP) |
| **Local Web Server** | TCP | 80 | `192.168.1.30:80` (TCP) |
| **DHCP Client Request**| UDP | 68 (Client) / 67 (Server)| `0.0.0.0:68` $\to$ `255.255.255.255:67` |
| **PC1 Web Client (Ephemeral)**| TCP | Dynamic (e.g., 1025) | `192.168.1.2:1025` (Inside Socket) |
| **PC1 Translated NAT Socket**| TCP | Dynamic (e.g., 2049) | `203.0.113.2:2049` (Outside Socket) |

---

## 4. Protocols Cheat Sheet

* **ARP (Address Resolution Protocol)**: Maps a known IPv4 address (Layer 3) to an unknown physical MAC address (Layer 2) on the local LAN interface. Run `arp -a` in the PC command prompt to inspect.
* **DHCP (Dynamic Host Configuration Protocol)**: The modern successor to **RARP** (Reverse ARP). Instead of mapping MACs to IPs statically, DHCP dynamically leases IP addresses, subnet masks, DNS servers, and gateway details.
* **NAT (Network Address Translation)**: Translates private RFC 1918 IPs (`192.168.X.X`) to public internet-routable IPs (`203.0.113.2`) to conserve global IP addresses.
* **OSPFv2 (Open Shortest Path First)**: A classless Link-State Routing Protocol used internally to dynamically exchange subnet paths between `HQ-Router` and `Edge-Router`.

---

## 5. Complete Router Configurations

Copy and paste these commands into the **CLI** tab of each router in Cisco Packet Tracer.

### A. HQ-Router Configuration
Performs internal routing, serves as the DHCP Server for local hosts, and advertises the LAN via OSPF.

```ios
enable
configure terminal
hostname HQ-Router

# 1. Interface Configurations
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.224
 no shutdown
 exit

interface Serial0/1/0
 ip address 10.0.0.5 255.255.255.252
 clock rate 64000
 no shutdown
 exit

# 2. DHCP Server Configuration (Replaces RARP functionality)
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.1.30
ip dhcp pool LAN_POOL
 network 192.168.1.0 255.255.255.224
 default-router 192.168.1.1
 dns-server 8.8.8.8
 exit

# 3. Dynamic Routing (OSPFv2)
router ospf 1
 network 192.168.1.0 0.0.0.31 area 0
 network 10.0.0.4 0.0.0.3 area 0
 exit
```

---

### B. Edge-Router Configuration
Runs OSPF internally, routes statically to the ISP, and translates internal addresses to the public IP using NAT Overload (PAT).

```ios
enable
configure terminal
hostname Edge-Router

# 1. Interface Configurations
interface Serial0/1/0
 ip address 10.0.0.6 255.255.255.252
 ip nat inside
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
 exit

# 2. Routing Configuration (OSPF & Static Default Route to Internet)
router ospf 1
 network 10.0.0.4 0.0.0.3 area 0
 # Redistribute default route to internal networks via OSPF
 default-information originate
 exit

# Default route pointing to the ISP's gateway interface IP
ip route 0.0.0.0 0.0.0.0 203.0.113.1

# 3. NAT Configuration (PAT / NAT Overload)
# Define access list of private subnets permitted to be translated
access-list 1 permit 192.168.1.0 0.0.0.255

# Apply NAT rule to bind access-list 1 to the external GigabitEthernet0/1 interface
ip nat inside source list 1 interface GigabitEthernet0/1 overload
exit
```

---

### C. ISP-Router Configuration (Simulated ISP)
Provides public routes. Does not run OSPF (simulating a separate autonomous system).

```ios
enable
configure terminal
hostname ISP-Router

# 1. Interface Configurations
interface GigabitEthernet0/1
 ip address 203.0.113.1 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/0
 ip address 8.8.8.1 255.255.255.0
 no shutdown
 exit

# 2. Return Static Route for NAT Pool to reach internal networks
# Must point to the Edge-Router's public IP
ip route 203.0.113.2 255.255.255.255 GigabitEthernet0/1
exit
```

---

## 6. How to Verify All Services in Packet Tracer

1. **ARP Verification**:
   - Open a command prompt on **PC1** and run:
     ```cmd
     arp -a
     ```
   - Send a ping to the Local Web Server: `ping 192.168.1.30`.
   - Run `arp -a` again. You will see the physical MAC address resolved for `192.168.1.30`.

2. **DHCP Verification**:
   - Go to **PC1** -> **IP Configuration** and select **DHCP**. 
   - Ensure it receives an IP address in the `192.168.1.X` range along with Gateway (`192.168.1.1`) and DNS (`8.8.8.8`).

3. **NAT & Socket Verification**:
   - Open the web browser on **PC1** and navigate to `http://8.8.8.8`.
   - In Packet Tracer's **Simulation Mode**, inspect the PDU (Packet Data Unit).
   - Under Layer 4 details, observe the source port (e.g., `1025`) and destination port (`80`).
   - On the `Edge-Router` CLI, run:
     ```ios
     Edge-Router# show ip nat translations
     ```
   - Observe the active mapping translating the inside socket address `192.168.1.X:1025` to the outside socket address `203.0.113.2:1025`.
