# CCNA Lab: Migrating to Router-on-a-Stick

## 1. Lab Introduction

Welcome to the lab. In today's scenario, you are a network engineer tasked with improving a small company's network. The company is growing, and the initial flat network design, where all devices are in the same broadcast domain, is no longer sufficient. New security and performance requirements dictate that we segment the network into departments using VLANs.

You will begin by implementing a basic inter-VLAN routing solution using a router with multiple physical interfaces. However, as the company expands further, you'll quickly discover the physical limitations of this design. This will be your catalyst to re-engineer the connection into a modern, scalable, and professional "Router-on-a-Stick" implementation, a cornerstone of CCNA-level networking.

**Your mission:** Successfully segment the network into three distinct VLANs (Management, Engineering, and Marketing) and configure a router to allow full, controlled communication between them using a single, efficient trunk link.

---

## 2. Network Diagram (Final State)

This diagram represents our target network configuration at the end of the lab.

![Network Diagram](https://i.imgur.com/8a6L2D6.png)

---

## 3. IP Addressing Schema

This table details the IP addressing plan for our network.

| Device | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | 10 | 10.10.10.1 | 255.255.255.0 | N/A |
| **R1** | 20 | 10.10.20.1 | 255.255.255.0 | N/A |
| **R1** | 30 | 10.10.30.1 | 255.255.255.0 | N/A |
| **PC1** | 10 | 10.10.10.10 | 255.255.255.0 | 10.10.10.1 |
| **PC2** | 20 | 10.10.20.10 | 255.255.255.0 | 10.10.20.1 |
| **PC3** | 20 | 10.10.20.11 | 255.255.255.0 | 10.10.20.1 |
| **PC4** | 30 | 10.10.30.10 | 255.255.255.0 | 10.10.30.1 |
| *Native VLAN*| 254| *(Unused)* | N/A | N/A |

---

## [Module 1] Initial Segmentation and Verification

In this module, you will create two VLANs and assign hosts to them. This will break communication between the hosts, proving that VLANs successfully isolate broadcast domains.

**Step 1: Verify Initial Connectivity**

Before making any changes, verify that all PCs can ping each other in the default flat network. *Note: The initial lab topology has all devices in VLAN 1.*

On **PC1**, ping **PC2**, **PC3**, and **PC4**.

```bash
PC1$ ping 10.10.1.11
PC1$ ping 10.10.1.12
PC1$ ping 10.10.1.13
```
*All pings should be successful.*

**Step 2: Create VLANs on Both Switches**

On **SW1** and **SW2**, create the Management and Engineering VLANs.

On **SW1**:
```
SW1# conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW1(config)# vlan 10
SW1(config-vlan)# name Management
SW1(config-vlan)# vlan 20
SW1(config-vlan)# name Engineering
SW1(config-vlan)# end
```

On **SW2**:
```
SW2# conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW2(config)# vlan 10
SW2(config-vlan)# name Management
SW2(config-vlan)# vlan 20
SW2(config-vlan)# name Engineering
SW2(config-vlan)# end
```

**Step 3: Verify VLAN Creation**

On both switches, use the `show vlan brief` command to verify the new VLANs exist.

On **SW1**:
```
SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                Gi1/0, Gi1/1, Gi1/2, Gi1/3
10   Management                       active    
20   Engineering                      active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```
*(Output on SW2 will be identical)*

> **Exam-Level Deep Dive: The VLAN Database**
> When you create a VLAN, the configuration is saved in a separate file called `vlan.dat` in the device's flash memory. This is different from the `startup-config` which holds the rest of the configuration. This is why you must delete both the `startup-config` and the `vlan.dat` file to fully reset a switch to its factory defaults.

**Step 4: Assign Switch Ports to VLANs**

Configure the switchports connected to the PCs to be in the correct access VLAN.

On **SW1**:
```
SW1# conf t
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description -> PC1 (Management)
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# interface GigabitEthernet0/2
SW1(config-if)# description -> PC2 (Engineering)
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# end
```

On **SW2**:
```
SW2# conf t
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description -> PC3 (Engineering)
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# interface GigabitEthernet0/2
SW2(config-if)# description -> PC4 (Initially Unused)
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 1
SW2(config-if)# end
```

**Step 5: Verify Port Assignments**

Use a variety of commands on **SW1** to see the configuration from different angles. This is a key troubleshooting skill.

```
SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/3, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3
10   Management                       active    Gi0/1
20   Engineering                      active    Gi0/2
...
```

Now, check the status of the interfaces.
```
SW1# show interface status

Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/1     -> PC1 (Management) connected    10         a-full a-1000 10/100/1000BaseTX
Gi0/2     -> PC2 (Engineering connected    20         a-full a-1000 10/100/1000BaseTX
...
```

Finally, get detailed information about a single interface.
```
SW1# show interface GigabitEthernet0/1 switchport
Name: Gi0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
...
Access Mode VLAN: 10 (Management)
...
```

**Step 6: Configure the Inter-Switch Link as a Trunk**

For VLAN traffic to cross between switches, the link connecting them must be a trunk.

On **SW1**:
```
SW1# conf t
SW1(config)# interface GigabitEthernet1/0
SW1(config-if)# description -> SW2 Trunk
SW1(config-if)# switchport mode trunk
SW1(config-if)# end
```
On **SW2**:
```
SW2# conf t
SW2(config)# interface GigabitEthernet1/0
SW2(config-if)# description -> SW1 Trunk
SW2(config-if)# switchport mode trunk
SW2(config-if)# end
```

**Step 7: Verify Trunking**
On **SW1**, verify the trunk is operational.
```
SW1# show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi1/0       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi1/0       1-4094

Port        Vlans allowed and active in management domain
Gi1/0       1,10,20

Port        Vlans in spanning tree forwarding state and not pruned
Gi1/0       1,10,20
```

**Step 8: Verify Loss of Connectivity**

Now that the hosts are in separate VLANs, they should **not** be able to ping each other. This is the expected behavior.

On **PC1**, attempt to ping **PC2**.
```bash
PC1$ ping 10.10.20.10
```
*This ping will fail. Why?*
> **Verification Question #1:** Why can't PC1 ping PC2, even though both switches know about both VLANs?

---

## [Module 2] Legacy Inter-VLAN Routing

Here, you will enable communication between VLAN 10 and 20 by connecting the router to the switch using two separate physical interfaces. Each interface will act as the gateway for one VLAN.

**Step 1: Configure Switch Ports for the Router**

On **SW1**, configure two access ports, one for each VLAN, to connect to the router.

```
SW1# conf t
SW1(config)# interface GigabitEthernet0/3
SW1(config-if)# description -> R1 for VLAN 10
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# interface GigabitEthernet1/1
SW1(config-if)# description -> R1 for VLAN 20
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# end
```

**Step 2: Configure Router Interfaces**

On **R1**, configure the two physical interfaces that connect to **SW1**. These will be the default gateways for their respective VLANs.

```
R1# conf t
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description -> Gateway for VLAN 10 (Management)
R1(config-if)# ip address 10.10.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# interface GigabitEthernet0/1
R1(config-if)# description -> Gateway for VLAN 20 (Engineering)
R1(config-if)# ip address 10.10.20.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# end
```
*You should see the interface line protocols change to 'up'.*

**Step 3: Verify Router Configuration and Connectivity**

First, check the router's interfaces.
```
R1# show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.10.10.1      YES manual up                    up
GigabitEthernet0/1         10.10.20.1      YES manual up                    up
...
```
Now, check the router's routing table. The 'C' and 'L' codes indicate directly connected networks and local interface IPs.
```
R1# show ip route
...
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.10.10.0/24 is directly connected, GigabitEthernet0/0
L        10.10.10.1/32 is directly connected, GigabitEthernet0/0
C        10.10.20.0/24 is directly connected, GigabitEthernet0/1
L        10.10.20.1/32 is directly connected, GigabitEthernet0/1
```

**Step 4: Update PC IP Configuration**

The PCs need to be configured with their new default gateways.

On **PC1**:
```bash
PC1$ sudo ip route add default via 10.10.10.1
```
On **PC2**:
```bash
PC2$ sudo ip route add default via 10.10.20.1
```
On **PC3**:
```bash
PC3$ sudo ip route add default via 10.10.20.1
```

**Step 5: Verify End-to-End Connectivity**

From **PC1**, you should now be able to ping its gateway (**R1**) and hosts in the other VLAN (**PC2** and **PC3**).

On **PC1**:
```bash
PC1$ ping 10.10.10.1   # Ping its own gateway
PC1$ ping 10.10.20.10  # Ping PC2
PC1$ ping 10.10.20.11  # Ping PC3
```
*All pings should now be successful.*

> **Verification Question #2:** You now need to add a third department, Marketing, in VLAN 30. Looking at your router, you see you have no more available physical interfaces. What is the primary limitation of this "legacy" inter-VLAN routing design?

---

## [Module 3] Migration to "Router-on-a-Stick"

The "Aha!" moment. You've hit the physical limits of the old design. Now, you will re-engineer the connection to use a single trunk link, which is far more scalable.

**Step 1: Create the Marketing VLAN**

First, create VLAN 30 on both switches.

On **SW1** and **SW2**:
```
SW1(config)# vlan 30
SW1(config-vlan)# name Marketing
SW1(config-vlan)# end
```

**Step 2: Move PC4 into the Marketing VLAN**

On **SW2**, move **PC4** into VLAN 30.
```
SW2# conf t
SW2(config)# interface GigabitEthernet0/2
SW2(config-if)# description -> PC4 (Marketing)
SW2(config-if)# switchport access vlan 30
SW2(config-if)# end
```

On **PC4**, update its IP address and default gateway to match the new subnet.
```bash
PC4$ sudo ip addr del 10.10.1.13/24 dev eth0
PC4$ sudo ip addr add 10.10.30.10/24 dev eth0
PC4$ sudo ip route add default via 10.10.30.1
```

**Step 3: Reconfigure the Switch-to-Router Link**

You will now convert the link between **R1** and **SW1** into an 802.1Q trunk. We will only use the link to `GigabitEthernet0/0` on the router.

On **SW1**:
```
SW1# conf t
SW1(config)# interface GigabitEthernet0/3
SW1(config-if)# description -> Trunk to R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 254
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface GigabitEthernet1/1
SW1(config-if)# description --- UNUSED ---
SW1(config-if)# shutdown
SW1(config-if)# end
```

> **Exam-Level Deep Dive: Security Best Practice**
> We set the native VLAN to `254`, an unused VLAN. Why? By default, the native VLAN is VLAN 1. If an attacker could gain access to VLAN 1, they could potentially craft double-tagged frames to "hop" into other VLANs. Moving the native VLAN to an unused, isolated ID is a critical security hardening step. We use `254` because it's a memorable number that is an invalid first octet for a Class A, B, or C address, making it easy to remember that it shouldn't have hosts in it.

**Step 4: Remove Legacy Configuration from Router**

On **R1**, you must remove the IP addresses from the physical interfaces before you can create logical subinterfaces.

```
R1# conf t
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no ip address
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# no ip address
R1(config-if)# description --- UNUSED ---
R1(config-if)# shutdown
R1(config-if)# end
```

> **Verification Question #3:** What would happen if you tried to create a subinterface with an IP address before removing the IP address from the parent physical interface?

**Step 5: Configure Router-on-a-Stick (ROAS)**

This is the core of the configuration. You will create logical subinterfaces, assign each one to a VLAN with the `encapsulation dot1q` command, and give it an IP address.

On **R1**:
```
R1# conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description >> Gateway for VLAN 10 (Management) <<
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 10.10.10.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description >> Gateway for VLAN 20 (Engineering) <<
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 10.10.20.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# description >> Gateway for VLAN 30 (Marketing) <<
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 10.10.30.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0
R1(config-if)# encapsulation dot1q 254 native
R1(config-if)# end
```

> **Exam-Level Deep Dive: ISL vs. 802.1Q**
> The `encapsulation dot1q` command tells the router to tag frames for that specific VLAN ID. `dot1q` is the IEEE standard for VLAN tagging. You might see an older, Cisco-proprietary protocol called `ISL` in legacy documentation. `dot1q` is the industry standard and the only one you need to know for the modern CCNA. `dot1q` is more efficient as it only adds a small 4-byte tag to the Ethernet frame, whereas ISL fully encapsulated the original frame, adding significant overhead. The `native` keyword is used to tell the router which VLAN's traffic should be sent *untagged*. This must match the native VLAN on the switch trunk port.

---

## [Module 4] Final Verification and Path Analysis

The migration is complete. Now, you must perform a full suite of tests to verify that the new configuration is working perfectly.

**Step 1: Verify Router Configuration**

Check the router's interfaces. You should now see the new subinterfaces.
```
R1# show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES manual up                    up
GigabitEthernet0/0.10      10.10.10.1      YES manual up                    up
GigabitEthernet0/0.20      10.10.20.1      YES manual up                    up
GigabitEthernet0/0.30      10.10.30.1      YES manual up                    up
GigabitEthernet0/1         unassigned      YES manual administratively down down
```

Examine the routing table. It should look very similar to before, but now all routes point to logical subinterfaces instead of physical ones.
```
R1# show ip route
...
      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
C        10.10.10.0/24 is directly connected, GigabitEthernet0/0.10
L        10.10.10.1/32 is directly connected, GigabitEthernet0/0.10
C        10.10.20.0/24 is directly connected, GigabitEthernet0/0.20
L        10.10.20.1/32 is directly connected, GigabitEthernet0/0.20
C        10.10.30.0/24 is directly connected, GigabitEthernet0/0.30
L        10.10.30.1/32 is directly connected, GigabitEthernet0/0.30
```

**Step 2: Verify Switch Trunk Port**

On **SW1**, verify the trunk port's status. Notice the native VLAN is now 254.
```
SW1# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/3       on               802.1q         trunking      254

Port        Vlans allowed on trunk
Gi0/3       1-4094

Port        Vlans allowed and active in management domain
Gi0/3       1,10,20,30,254

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/3       1,10,20,30,254
```

**Step 3: Verify Full End-to-End Connectivity**

From any PC, you should now be able to ping every other PC, regardless of its VLAN.

On **PC1** (VLAN 10):
```bash
PC1$ ping 10.10.20.10 # Ping PC2 (VLAN 20)
PC1$ ping 10.10.30.10 # Ping PC4 (VLAN 30)
```
*Both pings should succeed.*

On **PC4** (VLAN 30):
```bash
PC4$ ping 10.10.10.10 # Ping PC1 (VLAN 10)
PC4$ ping 10.10.20.11 # Ping PC3 (VLAN 20)
```
*Both pings should succeed.*

**Step 4: Analyze the Path with Traceroute**

Use `traceroute` to prove that **R1** is the Layer 3 hop between the VLANs.

On **PC1** (VLAN 10), trace the path to **PC4** (VLAN 30).
```bash
PC1$ traceroute 10.10.30.10
traceroute to 10.10.30.10 (10.10.30.10), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  0.450 ms  0.400 ms  0.350 ms
 2  10.10.30.10 (10.10.30.10)  0.850 ms  0.800 ms  0.750 ms
```
The output clearly shows that traffic from PC1 first goes to its gateway (`10.10.10.1` on R1), and the router then forwards it to the destination host.

> **Verification Question #4:** If you performed a `traceroute` from PC2 to PC3, would the output show the router's IP address? Why or why not?

---

## 5. Lab Conclusion

Congratulations on completing this advanced lab! You have successfully:
* Segmented a flat network using VLANs to create separate broadcast domains.
* Implemented and identified the limitations of legacy inter-VLAN routing.
* Migrated the network to a scalable and efficient "Router-on-a-Stick" configuration.
* Applied security best practices by changing the native VLAN.
* Performed a comprehensive verification and analysis of the final network state.

This "Router-on-a-Stick" configuration is a fundamental building block in network engineering. You are now well-equipped to design and implement robust, scalable LANs.

---

## 6. Answer Key

1.  **Why can't PC1 ping PC2?** Because they are in separate VLANs (and therefore separate broadcast domains and IP subnets). Without a Layer 3 device (a router) to route packets between them, communication is impossible.
2.  **What is the primary limitation of the legacy design?** It is not scalable. It requires one physical router interface for every VLAN. This is expensive, inefficient, and you will quickly run out of physical ports on the router.
3.  **What happens if you don't remove the physical IP?** The command to add a subinterface IP will be rejected. A physical interface and its subinterfaces cannot both have IP addresses assigned. The physical interface acts as a "container" for the logical ones.
4.  **Would a traceroute from PC2 to PC3 show the router?** No. PC2 and PC3 are in the same VLAN (VLAN 20) and the same IP subnet. Communication between them is switched at Layer 2 and never needs to be sent to the default gateway (the router).
