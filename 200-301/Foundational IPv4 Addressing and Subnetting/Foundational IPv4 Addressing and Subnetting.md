# CCNA Lab Guide: Dual-Core IPv4 Design (Professional Edition)

## Introduction

You are a junior network engineer, and today is your first day on the network refresh project. The new equipment has been racked and cabled according to the company's new standard: a highly redundant, dual-core hierarchical design. The network is physically ready, but it is silent. There is no connectivity.

A senior engineer has handed you an IP addressing plan with a challenge. To build an efficient network, you must calculate the correct subnet masks for all network links and user segments. Your mission is to perform the initial Layer 3 bring-up, configuring every device from the core routers down to the end-user PCs. Success will be measured by establishing local connectivity throughout the entire network.

## Network Diagram
<img src="https://i.imgur.com/39J0T4D.png" alt="A dual-core hierarchical network diagram with 2 core routers, 2 distribution switches, 4 access switches, and 4 PCs." width="100%">

## IP Addressing Table

**Your Task:** Before you begin, analyze the diagram and complete the `Subnet Mask` and `Default Gateway` columns. Use the most efficient subnetting possible for all point-to-point infrastructure links.

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Core Routers** | | | | |
| R1 | Gi0/0 | 10.0.0.1 | ? | N/A |
| R1 | Gi0/1 | 10.1.1.1 | ? | N/A |
| R1 | Gi0/2 | 10.2.1.1 | ? | N/A |
| R1 | Lo0 | 1.1.1.1 | ? | N/A |
| R2 | Gi0/0 | 10.0.0.2 | ? | N/A |
| R2 | Gi0/1 | 10.1.2.1 | ? | N/A |
| R2 | Gi0/2 | 10.2.2.1 | ? | N/A |
| R2 | Lo0 | 2.2.2.2 | ? | N/A |
| **Distribution Switches** | | | | |
| DSW1 | Gi0/0 | 10.1.1.2 | ? | N/A |
| DSW1 | Gi0/1 | 10.1.2.2 | ? | N/A |
| DSW1 | Vlan10 | 192.168.10.1 | ? | N/A |
| DSW1 | Vlan20 | 192.168.20.1 | ? | N/A |
| DSW2 | Gi0/0 | 10.2.1.2 | ? | N/A |
| DSW2 | Gi0/1 | 10.2.2.2 | ? | N/A |
| DSW2 | Vlan30 | 192.168.30.1 | ? | N/A |
| DSW2 | Vlan40 | 192.168.40.1 | ? | N/A |
| **End Hosts** | | | | |
| PC1 | eth0 | 192.168.10.10 | ? | ? |
| PC2 | eth0 | 192.168.20.10 | ? | ? |
| PC3 | eth0 | 192.168.30.10 | ? | ? |
| PC4 | eth0 | 192.168.40.10 | ? | ? |

## Lab Modules

---

### **Module 1: Core Router Configuration**

First, you'll bring the network backbone online by configuring the two core routers.

#### **Router 1 Configuration**
```
R1> enable
R1# configure terminal
R1(config)# hostname R1
R1(config)# interface Loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description Link to R2
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description Link to DSW1
R1(config-if)# ip address 10.1.1.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/2
R1(config-if)# description Link to DSW2
R1(config-if)# ip address 10.2.1.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# end
```
**Verification for R1:**
```
R1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.0.0.1        YES manual up                    up
GigabitEthernet0/1     10.1.1.1        YES manual up                    up
GigabitEthernet0/2     10.2.1.1        YES manual up                    up
Loopback0              1.1.1.1         YES manual up                    up
```

#### **Router 2 Configuration**
```
R2> enable
R2# configure terminal
R2(config)# hostname R2
R2(config)# interface Loopback0
R2(config-if)# ip address 2.2.2.2 255.255.255.255
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/0
R2(config-if)# description Link to R1
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/1
R2(config-if)# description Link to DSW1
R2(config-if)# ip address 10.1.2.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/2
R2(config-if)# description Link to DSW2
R2(config-if)# ip address 10.2.2.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
```
**Verification for R2:** A successful configuration adds `Connected` and `Local` routes to the routing table. Let's verify this.
```
R2# show ip route
Codes: L - local, C - connected, S - static...
Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0
L        10.0.0.2/32 is directly connected, GigabitEthernet0/0
C        10.1.2.0/30 is directly connected, GigabitEthernet0/1
L        10.1.2.1/32 is directly connected, GigabitEthernet0/1
C        10.2.2.0/30 is directly connected, GigabitEthernet0/2
L        10.2.2.1/32 is directly connected, GigabitEthernet0/2
```

---

### **Module 2: Distribution Layer Configuration**

Now, configure the distribution switches. Each will have two redundant uplinks—one to each core router.

#### **DSW1 Configuration**
```
DSW1(config)# hostname DSW1
DSW1(config)# vlan 10
DSW1(config-vlan)# name USERS_A
DSW1(config-vlan)# exit
DSW1(config)# vlan 20
DSW1(config-vlan)# name USERS_B
DSW1(config-vlan)# exit
DSW1(config)# interface GigabitEthernet0/0
DSW1(config-if)# description Link to R1
DSW1(config-if)# no switchport
DSW1(config-if)# ip address 10.1.1.2 255.255.255.252
DSW1(config-if)# no shutdown
DSW1(config-if)# exit
DSW1(config)# interface GigabitEthernet0/1
DSW1(config-if)# description Link to R2
DSW1(config-if)# no switchport
DSW1(config-if)# ip address 10.1.2.2 255.255.255.252
DSW1(config-if)# no shutdown
DSW1(config-if)# exit
DSW1(config)# interface Vlan10
DSW1(config-if)# ip address 192.168.10.1 255.255.255.0
DSW1(config-if)# no shutdown
DSW1(config-if)# exit
DSW1(config)# interface Vlan20
DSW1(config-if)# ip address 192.168.20.1 255.255.255.0
DSW1(config-if)# no shutdown
DSW1(config-if)# end
```
**Verification for DSW1:**
1. **Verify L3 Port Conversion:** The `show interfaces switchport` command will prove a port is now Layer 3.
```
DSW1# show interfaces gigabitEthernet 0/0 switchport
Name: Gi0/0
Switchport: Disabled
```
2. **Verify Physical Neighbors:** Use `show cdp neighbors` to confirm your redundant physical connections to the core.
```
DSW1# show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R1               Gig 0/0           178        R S I       CSR1000V  Gig 0/1
R2               Gig 0/1           155        R S I       CSR1000V  Gig 0/1
```

#### **DSW2 Configuration**
```
DSW2(config)# hostname DSW2
DSW2(config)# vlan 30
DSW2(config-vlan)# name USERS_C
DSW2(config-vlan)# exit
DSW2(config)# vlan 40
DSW2(config-vlan)# name USERS_D
DSW2(config-vlan)# exit
DSW2(config)# interface GigabitEthernet0/0
DSW2(config-if)# description Link to R1
DSW2(config-if)# no switchport
DSW2(config-if)# ip address 10.2.1.2 255.255.255.252
DSW2(config-if)# no shutdown
DSW2(config-if)# exit
DSW2(config)# interface GigabitEthernet0/1
DSW2(config-if)# description Link to R2
DSW2(config-if)# no switchport
DSW2(config-if)# ip address 10.2.2.2 255.255.255.252
DSW2(config-if)# no shutdown
DSW2(config-if)# exit
DSW2(config)# interface Vlan30
DSW2(config-if)# ip address 192.168.30.1 255.255.255.0
DSW2(config-if)# no shutdown
DSW2(config-if)# exit
DSW2(config)# interface Vlan40
DSW2(config-if)# ip address 192.168.40.1 255.255.255.0
DSW2(config-if)# no shutdown
DSW2(config-if)# end
```
**Verification for DSW2:**
```
DSW2# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     10.2.1.2        YES manual up                    up
GigabitEthernet0/1     10.2.2.2        YES manual up                    up
Vlan1                  unassigned      YES unset  administratively down down
Vlan30                 192.168.30.1    YES manual up                    up
Vlan40                 192.168.40.1    YES manual up                    up
```

---

### **Module 3: Access Layer Configuration**

This crucial step connects the end hosts to the network. You will configure each access switch port connected to a PC as a static `access` port and assign it to the correct VLAN.

#### **ASW1 Configuration**
```
ASW1(config)# hostname ASW1
ASW1(config)# vlan 10
ASW1(config-vlan)# name USERS_A
ASW1(config-vlan)# exit
ASW1(config)# interface GigabitEthernet0/1
ASW1(config-if)# description Link to PC1
ASW1(config-if)# switchport mode access
ASW1(config-if)# switchport access vlan 10
ASW1(config-if)# no shutdown
ASW1(config-if)# end
```
**Verification for ASW1:**
```
ASW1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/2, Gi0/3, Gi1/0...
10   USERS_A                          active    Gi0/1
```
#### **ASW2 Configuration**
```
ASW2(config)# hostname ASW2
ASW2(config)# vlan 20
ASW2(config-vlan)# name USERS_B
ASW2(config-vlan)# exit
ASW2(config)# interface GigabitEthernet0/1
ASW2(config-if)# description Link to PC2
ASW2(config-if)# switchport mode access
ASW2(config-if)# switchport access vlan 20
ASW2(config-if)# no shutdown
ASW2(config-if)# end
```
#### **ASW3 Configuration**
```
ASW3(config)# hostname ASW3
ASW3(config)# vlan 30
ASW3(config-vlan)# name USERS_C
ASW3(config-vlan)# exit
ASW3(config)# interface GigabitEthernet0/1
ASW3(config-if)# description Link to PC3
ASW3(config-if)# switchport mode access
ASW3(config-if)# switchport access vlan 30
ASW3(config-if)# no shutdown
ASW3(config-if)# end
```
#### **ASW4 Configuration**
```
ASW4(config)# hostname ASW4
ASW4(config)# vlan 40
ASW4(config-vlan)# name USERS_D
ASW4(config-vlan)# exit
ASW4(config)# interface GigabitEthernet0/1
ASW4(config-if)# description Link to PC4
ASW4(config-if)# switchport mode access
ASW4(config-if)# switchport access vlan 40
ASW4(config-if)# no shutdown
ASW4(config-if)# end
```

---

### **Module 4: Host Verification**
The PCs are pre-configured in the CML file. Log into each PC and run these commands to verify their settings.
* **On PC1:**
  `PC1:~# ip addr show eth0`
  `PC1:~# ip route`
* **On PC2:**
  `PC2:~# ip addr show eth0`
  `PC2:~# ip route`
* **On PC3:**
  `PC3:~# ip addr show eth0`
  `PC3:~# ip route`
* **On PC4:**
  `PC4:~# ip addr show eth0`
  `PC4:~# ip route`

---

### **Module 5: End-to-End Connectivity Testing**
Verify connectivity across the network to confirm your implementation was a success.

1.  **Test Access to Distribution:** From each PC, ping its default gateway.
    `PC1:~# ping 192.168.10.1`
    `PC2:~# ping 192.168.20.1`
    `PC3:~# ping 192.168.30.1`
    `PC4:~# ping 192.168.40.1`

2.  **Test Distribution to Core:** From each Distribution Switch, ping BOTH of its core router neighbors.
    `DSW1# ping 10.1.1.1`
    `DSW1# ping 10.1.2.1`
    `DSW2# ping 10.2.1.1`
    `DSW2# ping 10.2.2.1`

3.  **Test Core to Core:** From R1, ping R2 on its direct link and its loopback.
    `R1# ping 10.0.0.2`
    `R1# ping 2.2.2.2`

4.  **Trace the Path:** From a PC, trace the path to a core router's loopback. This proves the entire path is working.
    `PC1:~# traceroute 2.2.2.2`
    The output should show three hops:
    1. `192.168.10.1` (DSW1)
    2. `10.1.2.1` (R2's link from DSW1)
    3. `2.2.2.2` (R2's loopback)

## Conclusion

Congratulations, engineer! You have successfully brought the new network online. By methodically calculating subnets and configuring each layer of the hierarchy, you have established a robust and redundant foundation. All devices can now communicate with their immediate neighbors. The network is ready for the next stage: implementing routing protocols to achieve full end-to-end connectivity.

## Answer Key

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Core Routers** | | | | |
| R1 | Gi0/0 | 10.0.0.1 | `255.255.255.252` | N/A |
| R1 | Gi0/1 | 10.1.1.1 | `255.255.255.252` | N/A |
| R1 | Gi0/2 | 10.2.1.1 | `255.255.255.252` | N/A |
| R1 | Lo0 | 1.1.1.1 | `255.255.255.255` | N/A |
| R2 | Gi0/0 | 10.0.0.2 | `255.255.255.252` | N/A |
| R2 | Gi0/1 | 10.1.2.1 | `255.255.255.252` | N/A |
| R2 | Gi0/2 | 10.2.2.1 | `255.255.255.252` | N/A |
| R2 | Lo0 | 2.2.2.2 | `255.255.255.255` | N/A |
| **Distribution Switches** | | | | |
| DSW1 | Gi0/0 | 10.1.1.2 | `255.255.255.252` | N/A |
| DSW1 | Gi0/1 | 10.1.2.2 | `255.255.255.252` | N/A |
| DSW1 | Vlan10 | 192.168.10.1 | `255.255.255.0` | N/A |
| DSW1 | Vlan20 | 192.168.20.1 | `255.255.255.0` | N/A |
| DSW2 | Gi0/0 | 10.2.1.2 | `255.255.255.252` | N/A |
| DSW2 | Gi0/1 | 10.2.2.2 | `255.255.255.252` | N/A |
| DSW2 | Vlan30 | 192.168.30.1 | `255.255.255.0` | N/A |
| DSW2 | Vlan40 | 192.168.40.1 | `255.255.255.0` | N/A |
| **End Hosts** | | | | |
| PC1 | eth0 | 192.168.10.10 | `255.255.255.0` | `192.168.10.1` |
| PC2 | eth0 | 192.168.20.10 | `255.255.255.0` | `192.168.20.1` |
| PC3 | eth0 | 192.168.30.10 | `255.255.255.0` | `192.168.30.1` |
| PC4 | eth0 | 192.168.40.10 | `255.255.255.0` | `192.168.40.1` |
