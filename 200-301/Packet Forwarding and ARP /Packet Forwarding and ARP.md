# CCNA Lab: The Packet's Journey - Uncovering ARP and Multi-Switch Forwarding

## Introduction: The Investigation

Welcome, network detective. You've been assigned to a network that is, by all accounts, perfectly functional. A client can communicate with a server across multiple network segments. But *how*? Your mission is not to fix a problem, but to uncover the hidden mechanics of this communication. You will dissect the end-to-end flow of a single packet, revealing the elegant collaboration between Layer 2 switching across an intermediate switch and Layer 3 routing that makes modern networking possible.

## Network Topology (v2)

Our network has been updated with an intermediate switch, `SW_Core`, to better reflect real-world designs.

![Network Diagram v2](https://storage.googleapis.com/gen-ai-assisting-tool-images/network_diagram_v2.png)

## IP Addressing Schema

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| PC1 | eth0 | 10.1.1.10 | 255.255.255.0 | 10.1.1.1 |
| R1 | Gi0/0 | 10.1.1.1 | 255.255.255.0 | N/A |
| | Gi0/1 | 10.12.12.1 | 255.255.255.0 | N/A |
| R2 | Gi0/0 | 10.12.12.2 | 255.255.255.0 | N/A |
| | Gi0/1 | 10.3.3.1 | 255.255.255.0 | N/A |
| SERVER | eth0 | 10.3.3.10 | 255.255.255.0 | 10.3.3.1 |

---

### **Module 1: Initial Path & Data Verification**

In this first module, we'll confirm the baseline functionality and gather the key data points we need for our investigation.

**Step 1: Verify Physical Connectivity with CDP**

A true professional verifies the physical and data-link layers first. Use the Cisco Discovery Protocol (CDP) to map out the connections.

On **SW1**, verify its neighbors.

```
SW1# show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
PC1              Gig 0/1           149        H           Alpine    eth0
SW_Core          Gig 0/0           175        S I         IOSvL2    Gig 0/0
```

SW1 correctly sees PC1 and our new switch, SW_Core. Now, let's hop to **SW_Core**.

```
SW_Core# show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R1               Gig 0/1           133        R S I       IOSv      Gig 0/0
SW1              Gig 0/0           162        S I         IOSvL2    Gig 0/0
```

Perfect. SW_Core sees its neighbors, SW1 and R1. This confirms our physical cabling is correct.

**Step 2: Verify End-to-End Connectivity and Path**

From PC1, run a `ping` and `traceroute` to the SERVER.

```
PC1# ping 10.3.3.10
PING 10.3.3.10 (10.3.3.10): 56 data bytes
64 bytes from 10.3.3.10: seq=0 ttl=62 time=1.876 ms

PC1# traceroute 10.3.3.10
traceroute to 10.3.3.10 (10.3.3.10), 30 hops max, 46 byte packets
 1  10.1.1.1 (10.1.1.1)  0.451 ms  0.328 ms  0.315 ms
 2  10.12.12.2 (10.12.12.2)  0.748 ms  0.737 ms  0.725 ms
 3  10.3.3.10 (10.3.3.10)  1.154 ms  1.143 ms  1.131 ms
```

The Layer 3 path is unchanged, which is expected. The intermediate switch is transparent to the IP layer.

> **Verification Question 1:** Based on the `traceroute` output, what are the IP addresses of the two routers between PC1 and the SERVER?

---

### **Module 2: Deconstructing the ARP Process**

The ARP process itself remains the same, but now the ARP request and reply frames must traverse our new switch, SW_Core. Let's verify.

**Step 1: Watch the Live ARP Exchange on the Router**

Enable ARP debugging on R1.

```
R1# debug arp
ARP packet debugging is on
```

Now, go to PC1, clear its ARP cache, and ping R1's IP address.

```
PC1# sudo ip neigh flush all
PC1# ping 10.1.1.1 -c 1
```

Switch back to the R1 console. You will see the exact same ARP exchange as before, proving that the Layer 2 path doesn't change the Layer 3 ARP mechanism.

```
R1#
*Jun 20 22:45:10.123: IP ARP: rcvd req src 10.1.1.10 5254.0000.0100, dst 10.1.1.1 0000.0000.0000 GigabitEthernet0/0
*Jun 20 22:45:10.123: IP ARP: sent rep src 10.1.1.1 5254.0000.0200, dst 10.1.1.10 5254.0000.0100 GigabitEthernet0/0
```

Remember to **turn off the debug** once you've seen the output.

```
R1# no debug arp
ARP packet debugging is off
```

---

### **Module 3: Analyzing Multi-Switch Layer 2 Forwarding**

This is where our new switch makes things interesting. How does a frame from PC1 get to R1? Both SW1 and SW_Core must cooperate.

**Step 1: Inspect the MAC Address Tables**

First, check the MAC table on **SW1**.

```
SW1# show mac address-table | include DYNAMIC
   1    5254.0000.0100    DYNAMIC     Gi0/1
   1    5254.0000.0200    DYNAMIC     Gi0/0
```

Notice that SW1 learns PC1's MAC on Gi0/1 (correct) and R1's MAC on Gi0/0, the port leading to SW_Core. SW1 doesn't know about R1 directly; it only knows the next-hop switch.

Now, check the MAC table on **SW_Core**.

```
SW_Core# show mac address-table | include DYNAMIC
   1    5254.0000.0100    DYNAMIC     Gi0/0
   1    5254.0000.0200    DYNAMIC     Gi0/1
```

This is the crucial insight. SW_Core has learned PC1's MAC address on its port Gi0/0 (from SW1) and R1's MAC address on its port Gi0/1 (from R1). It bridges the gap.

**Step 2: Clear the MAC Tables and Observe Repopulation**

Let's see the switches rebuild their tables. Clear the tables on **both switches**.

```
SW1# clear mac address-table dynamic
SW_Core# clear mac address-table dynamic
```

Wait about 15-30 seconds and check both tables again *without generating any user traffic*. You will see the MAC address for R1 (`5254.0000.0200`) reappear on both switches, learned from interface keepalives propagating through the network.

> **Verification Question 2:** When a frame is sent from PC1 to R1, SW1 forwards it out port Gi0/0. How does SW_Core know to then forward that same frame out its port Gi0/1 to reach R1?

---

### **Module 4: Synthesizing the End-to-End Flow**

The core concepts are the same, but the Layer 2 path is now more complex.

**Final Challenge:**

Based on your observations, describe the step-by-step process of a ping packet traveling from PC1 to the SERVER. Be specific about what happens to the **Source/Destination IP** and **Source/Destination MAC** addresses at each hop, and which ports are used on SW1 and SW_Core.

## Conclusion

By adding a single switch, you have dramatically enhanced your understanding of Layer 2. You have now seen firsthand how multiple switches work together transparently to forward frames, each making independent decisions based on their own MAC address tables. This multi-switch forwarding, combined with the ARP resolution at the edge of each IP network, forms the bedrock of packet delivery.

## Answer Key

* **Verification Question 1:** The two routers are at IP addresses `10.1.1.1` (R1) and `10.12.12.2` (R2).

* **Verification Question 2:** SW_Core knows to forward the frame to Gi0/1 because it has an entry in its own MAC address table that maps R1's destination MAC address to that specific port. It learned this mapping when R1 sent a frame (like a keepalive) from that port previously.

* **Final Challenge Answer:** The core IP and MAC address logic remains the same as in the previous version, but the Layer 2 path now includes SW_Core. A frame from PC1 to R1 is sent from PC1, forwarded by SW1 out its uplink port (e.g. Gi0/0), received by SW_Core on its downlink port (e.g. Gi0/0), and finally forwarded by SW_Core out its uplink port (e.g., Gi0/1) to R1.
