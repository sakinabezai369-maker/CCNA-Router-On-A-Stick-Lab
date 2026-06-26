# Router-on-a-Stick (ROAS) Inter-VLAN Routing Lab

## 📌 Project Overview
This repository contains a Cisco Packet Tracer laboratory demonstrating **Router-on-a-Stick (ROAS)** Inter-VLAN Routing. The project showcases how to route traffic between multiple logically isolated networks (VLAN 10, VLAN 20, and Native VLAN 199) using a single physical router interface split into virtual sub-interfaces via 802.1Q encapsulation.

---

## 🎯 Learning Objectives
By completing this lab, you will learn how to:
* Configure VLANs on a Cisco Catalyst Switch.
* Implement IEEE 802.1Q trunking parameters.
* Set up Router-on-a-Stick (ROAS) using virtual sub-interfaces.
* Handle strict Native VLAN coordination and remediation.
* Troubleshoot and verify Layer 2/Layer 3 boundaries using Cisco IOS verification tools.

---

## 🏗️ Why Router-on-a-Stick?
Router-on-a-Stick (ROAS) is a highly scalable alternative to Traditional Inter-VLAN Routing. 

Instead of exhausting one physical router interface per VLAN (which leads to hardware wastage and high costs), ROAS multiplexes traffic from multiple broadcast domains over a **single physical trunk link**. The router separates and routes this traffic logically using 802.1Q virtual sub-interfaces.

---

## 🌐 Network Topology
The actual network diagram implemented in this laboratory is shown below:

![Network Topology](topology.png)

### 📋 Network Addressing & Port Mapping Table:
| Device Name | Interface / Sub-interface | IP Address | Subnet Mask | Default Gateway | VLAN Assignment | Connected Switch Port |
| --- | --- | --- | --- | --- | --- | --- |
| **Router0** | Gig0/0 | Unassigned | N/A | N/A | Trunk Main Interface | Connected to Fa0/3 |
| **Router0** | Gig0/0.10 | 192.168.100.1 | 255.255.255.0 | N/A | VLAN 10 (Sales) | Virtual Mapping |
| **Router0** | Gig0/0.20 | 192.168.150.1 | 255.255.255.0 | N/A | VLAN 20 (HR) | Virtual Mapping |
| **Router0** | Gig0/0.30 | 192.168.200.1 | 255.255.255.0 | N/A | VLAN 199 (Native) | Virtual Mapping |
| **PC2** | FastEthernet0 | 192.168.100.10 | 255.255.255.0 | 192.168.100.1 | VLAN 10 (Sales) | **Fa0/4** |
| **PC3** | FastEthernet0 | 192.168.100.11 | 255.255.255.0 | 192.168.100.1 | VLAN 10 (Sales) | **Fa0/5** |
| **PC0** | FastEthernet0 | 192.168.150.10 | 255.255.255.0 | 192.168.150.1 | VLAN 20 (HR) | **Fa0/1** |
| **PC1** | FastEthernet0 | 192.168.150.11 | 255.255.255.0 | 192.168.150.1 | VLAN 20 (HR) | **Fa0/2** |
| **PC4** | FastEthernet0 | 192.168.200.10 | 255.255.255.0 | 192.168.200.1 | VLAN 199 (Native) | **Fa0/6** |
| **PC5** | FastEthernet0 | 192.168.200.11 | 255.255.255.0 | 192.168.200.1 | VLAN 199 (Native) | **Fa0/7** |

---

## 🛠️ CLI Configurations

### 1. Switch Configurations (Switch0)
* Created and explicitly named departmental VLANs.
* Configured local access ports for the data networks.
* Secured administrative hosts on ports `Fa0/6-7` by mapping them as **Access Ports** directly into Native VLAN 199 to mitigate L2 vulnerabilities.
* Configured uplink interface `Fa0/3` as an **802.1Q Trunk Port** explicitly referencing Native VLAN 199.

```ios
! VLAN Initialization
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Sales
Switch(config)# vlan 20
Switch(config-vlan)# name HR
Switch(config)# vlan 199
Switch(config-vlan)# name Native

! Access Mode Port Allocations
Switch(config)# interface range fa0/4-5
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10

Switch(config)# interface range fa0/1-2
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20

Switch(config)# interface range fa0/6-7
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 199

! Router Link Trunk Infrastructure
Switch(config)# interface fa0/3
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 199
Switch(config-if)# end
Switch# write

```

### 2. Router Configurations (Router0)

* Brought up the primary physical interface context.
* Spawned targeted sub-interfaces mapping back to Layer 2 802.1Q tags.
* Configured sub-interface `.30` with the `native` statement to instruct the IP engine to handle inbound untagged streams as Native VLAN 199 data.

```ios
Router> enable
Router# configure terminal
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown

! Sub-interface mapping for VLAN 10
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.100.1 255.255.255.0

! Sub-interface mapping for VLAN 20
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.150.1 255.255.255.0

! Sub-interface mapping for Native VLAN 199
Router(config)# interface GigabitEthernet0/0.30
Router(config-subif)# encapsulation dot1Q 199 native
Router(config-subif)# ip address 192.168.200.1 255.255.255.0
Router(config-subif)# end
Router# write

```

---

## 🏷️ Understanding Tagged vs. Untagged Traffic in this Lab

To accurately demonstrate frame processing behavior on Cisco hardware, we must audit how the platform encapsulates data across access and trunk links based on our port matrices:

### 1. Untagged Data Frames (VLAN 10 & VLAN 20 Access Links)

* End devices (like PC2/PC3 in Sales, and PC0/PC1 in HR) do not natively process Layer 2 802.1Q tags. They emit clean, standard **Untagged Ethernet Frames**.
* When **PC2** transmits a payload, it exits its network card untagged. **Switch0** intercepts it on ingress interface **`Fa0/4`**. Because this port is restricted to Access VLAN 10, the switch internally processes it as Sales data. Before forwarding this traffic up to the router, the switch must insert a 4-byte 802.1Q Tag containing the tag header ID 10 so Router0 can read it.

### 2. The Native VLAN Paradigm Exception (PC4 & PC5 on Ports `Fa0/6-7`)

* **Core Rule:** The Native VLAN is the single broadcast domain on an 802.1Q trunk port that transmits and expects data **without an explicit 802.1Q tag header (Untagged)** across the physical wire.
* In our deployment, **PC4** and **PC5** are administrative hosts attached to access ports **`Fa0/6`** and **`Fa0/7`**, explicitly mapped to **VLAN 199 (Native)**.
* When **PC4** transmits an ICMP echo request, it enters Switch0 untagged. When Switch0 forwards this payload across the uplink trunk link `Fa0/3` toward the router, **it skips adding an 802.1Q tag header**. This happens because the frame's internal VLAN ID (199) matches the designated `switchport trunk native vlan 199` statement.
* **Router0** receives this raw, untagged packet directly onto physical port `Gig0/0`. Because it has an explicit `encapsulation dot1Q 199 native` statement bound to sub-interface `Gig0/0.30`, it parses the untagged frame natively into that sub-interface context without looking for an explicit tag field.

---

## 🔄 Traffic Flow Analysis (Data Path Engineering)

When PC2 (VLAN 10) initiates an ICMP ping message destined for PC0 (VLAN 20):

1. **PC2** calculates that the target IP resides on a foreign subnet and encapsulates the packet using the MAC address of its Default Gateway.
2. **Switch0** receives the raw frame on access interface `Fa0/4` and applies an internal **802.1Q Tag (VLAN ID: 10)**.
3. The switch drops the tagged frame down trunk link `Fa0/3` directly to **Router0**.
4. The router strips the L2 header on physical port `G0/0`, redirects it to logical sub-interface `G0/0.10`, reads the L3 destination address (`192.168.150.10`), and performs a route table lookup.
5. Finding a directly connected match via sub-interface `G0/0.20`, the router encapsulates the packet into a brand new Layer 2 frame, tags it explicitly as **VLAN 20**, and passes it right back down the same wire.
6. The switch picks up the frame on trunk port `Fa0/3`, notices the VLAN 20 tag, strips the tag entirely, and transfers a clean untagged frame to **PC0** over access link `Fa0/1`.

---

## 🔍 Verification & Connectivity Matrices

Verification was completed using Cisco IOS validation suites and ICMP routines.

### 📊 Practical Evidence

#### 🟢 Verification Command Checks

Executing `show vlan brief` on the switch confirms accurate data-plane allocation:


Executing `show ip route` on the router displays active connected routing protocols across subnets:


#### 🟢 Successful Inter-VLAN Ping Tests

Initial packet drops represent standard ARP discovery routines, immediately followed by **100% data transmission (0% loss)**:

```text
C:\>ping 192.168.150.10
Pinging 192.168.150.10 with 32 bytes of data:

Reply from 192.168.150.10: bytes=32 time<1ms TTL=127
Reply from 192.168.150.10: bytes=32 time<1ms TTL=127
Reply from 192.168.150.10: bytes=32 time<1ms TTL=127
Reply from 192.168.150.10: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.150.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

```

---

## ⚠️ Design Considerations & Key Takeaways

While Router-on-a-Stick minimizes upfront infrastructural hardware costs, it channels all Inter-VLAN workloads through a single shared physical bottleneck link. As a business grows, this link can face heavy traffic congestion. Modern enterprise networks typically handle high-volume inter-departmental traffic directly inside core hardware fabrics using Layer 3 Multilayer Switches.

---

## 📂 Repository Structure

```text
Router-on-a-Stick-Inter-VLAN-Routing/
│
├── README.md
├── Router-on-a-Stick.pkt
├── topology.png
└── screenshots/
    ├── show-vlan-brief.png
    ├── show-interfaces-trunk.png
    ├── show-ip-route.png
    ├── ping-success.png
    └── simulation-mode.png

```

---

## 👩‍💻 About This Project

This project was engineered during my **CCNA** certification studies to master Layer 2 broadcast containment parameters, dot1q trunk multiplexing syntax, and Layer 3 sub-interface virtual routing boundaries.
