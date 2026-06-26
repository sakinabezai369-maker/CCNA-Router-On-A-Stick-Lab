# CCNA Router-on-a-Stick (ROAS) Inter-VLAN Routing Lab

## 📌 Project Overview

This repository contains a Cisco Packet Tracer laboratory demonstrating **Router-on-a-Stick (ROAS)** Inter-VLAN Routing.

The lab shows how multiple VLANs can communicate through a **single physical router interface** using **IEEE 802.1Q trunking** and **router sub-interfaces**. It also demonstrates proper **Native VLAN (VLAN 199)** configuration and verification.

---

## 🎯 Learning Objectives

By completing this lab, you will learn how to:

* Configure VLANs on a Cisco switch.
* Configure IEEE 802.1Q trunking.
* Configure Router-on-a-Stick (ROAS).
* Configure router sub-interfaces.
* Configure a Native VLAN.
* Verify Inter-VLAN Routing.
* Troubleshoot common ROAS issues.

---

## 🏗️ Why Router-on-a-Stick?

Router-on-a-Stick (ROAS) is a scalable alternative to Traditional Inter-VLAN Routing.

Instead of requiring one physical router interface per VLAN, ROAS uses a **single trunk link** carrying traffic for multiple VLANs. The router separates this traffic using **802.1Q sub-interfaces**, significantly reducing hardware requirements.

---

## 🌐 Network Topology

![Network Topology](topology.png)

---

## 📋 Network Addressing

| Device  | Interface | IP Address     | VLAN              |
| ------- | --------- | -------------- | ----------------- |
| Router0 | G0/0.10   | 192.168.100.1  | VLAN 10 (Sales)   |
| Router0 | G0/0.20   | 192.168.150.1  | VLAN 20 (HR)      |
| Router0 | G0/0.30   | 192.168.200.1  | VLAN 199 (Native) |
| PC2     | Fa0       | 192.168.100.10 | VLAN 10           |
| PC3     | Fa0       | 192.168.100.11 | VLAN 10           |
| PC0     | Fa0       | 192.168.150.10 | VLAN 20           |
| PC1     | Fa0       | 192.168.150.11 | VLAN 20           |
| PC4     | Fa0       | 192.168.200.10 | VLAN 199          |
| PC5     | Fa0       | 192.168.200.11 | VLAN 199          |

---

# 🛠️ Switch Configuration

### VLAN Creation

```ios
vlan 10
 name Sales

vlan 20
 name HR

vlan 199
 name Native
```

### Access Ports

```ios
interface range fa0/4-5
 switchport mode access
 switchport access vlan 10

interface range fa0/1-2
 switchport mode access
 switchport access vlan 20

interface range fa0/6-7
 switchport mode access
 switchport access vlan 199
```

### Trunk Port

```ios
interface fa0/3
 switchport mode trunk
 switchport trunk native vlan 199
```

---

# 🛠️ Router Configuration

Enable the physical interface.

```ios
interface GigabitEthernet0/0
 no shutdown
```

### VLAN 10

```ios
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.100.1 255.255.255.0
```

### VLAN 20

```ios
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.150.1 255.255.255.0
```

### Native VLAN

```ios
interface GigabitEthernet0/0.30
 encapsulation dot1Q 199 native
 ip address 192.168.200.1 255.255.255.0
```

---

# 🔄 Traffic Flow Analysis

When **PC2 (VLAN 10)** sends traffic to **PC0 (VLAN 20)**:

1. PC2 determines that the destination belongs to another subnet.
2. PC2 forwards the packet to its Default Gateway.
3. The switch tags the frame with **802.1Q VLAN 10**.
4. The router receives the tagged frame on G0/0.
5. Sub-interface G0/0.10 processes the packet.
6. The router routes the packet to sub-interface G0/0.20.
7. The frame is re-tagged as VLAN 20.
8. The switch removes the tag before delivering the frame to PC0.

---

# 🔍 Verification

### Switch

```text
show vlan brief
show interfaces trunk
```

### Router

```text
show ip interface brief
show ip route
show running-config
```

---

# 🧪 Connectivity Test

```bash
ping 192.168.150.10
```

Expected Result:

* The first ping may timeout because of ARP.
* All subsequent replies should succeed.

---

# 📊 Screenshots

Include screenshots showing:

* Network Topology
* show vlan brief
* show interfaces trunk
* show ip route
* Successful Ping
* Packet Tracer Simulation Mode

---

# 🚀 Key Takeaways

* One physical router interface supports multiple VLANs.
* IEEE 802.1Q provides VLAN tagging.
* Native VLAN traffic is transmitted untagged.
* Router sub-interfaces enable Layer 3 routing between VLANs.
* ROAS is significantly more scalable than Traditional Inter-VLAN Routing.

---

# ⚠️ Design Considerations

Although Router-on-a-Stick greatly reduces hardware requirements, all VLAN traffic traverses a single trunk link. As network traffic grows, this trunk can become a bottleneck. Modern enterprise networks often replace ROAS with Layer 3 switches that perform Inter-VLAN Routing directly in hardware.

---

# 📂 Repository Structure

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

# 👩‍💻 About This Project

This project was developed as part of my CCNA learning journey and technical portfolio.

It demonstrates practical experience with IEEE 802.1Q trunking, Router-on-a-Stick (ROAS), Native VLAN configuration, router sub-interfaces, and Inter-VLAN Routing while reinforcing the networking concepts required for enterprise network design.
