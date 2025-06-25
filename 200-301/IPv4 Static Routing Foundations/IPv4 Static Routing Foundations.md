# CCNA Lab Guide: IPv4 Static Routing Foundations

## Introduction

Welcome to the Static Routing Foundations lab! In the real world, not every network runs a dynamic routing protocol. Small office/home office (SOHO) networks, secure isolated segments, and stub networks often rely on the simple, predictable, and secure nature of static routing. As a network administrator, you must be able to manually define the paths that data will take through your network.

**Your Mission:** You are the network administrator for a small company that has just expanded into three departments. The initial network setup has left these departments as isolated "islands" of connectivity. Your task is to implement a robust and resilient routing scheme using only static routes to enable full communication between all departments. This lab will challenge you to think like a router, manually building its understanding of the network topology.

---

## Network Diagram

```text
        +-----------------+                                 +-----------------+
        | PC1             |                                 | PC2             |
        | 192.168.1.10/24 |                                 | 192.168.2.10/24 |
        +-----------------+                                 +-----------------+
               |                                                     |
           +-------+                                             +-------+
           |  SW1  |                                             |  SW2  |
           +-------+                                             +-------+
               |                                                     |
         LAN 1 |                                               LAN 2 |
               |                                                     |
      .1 +----(R1)----------------------10.1.2.0/30----------------(R2)----+ .1
         |      \.1                                               /.2      |
         |       \                                               /       |
         |        \                                             /        |
   10.1.3.0/30     \                                           /     10.2.3.0/30
         |          \                                         /          |
         |           \                                       /           |
         |            \.2                                   /.1           |
         |             +---------------(R3)----------------+             |
         |                           .1 |                                |
         |                              | LAN 3: 192.168.3.0/24          |
         |                              |                                |
         |                          +-------+                            |
         |                          |  SW3  |                            |
         |                          +-------+                            |
         |                              |                                |
         |                          +-----------------+                  |
         |                          | PC3             |                  |
         |                          | 192.168.3.10/24 |                  |
         |                          +-----------------+                  |
         +----------------------------------------------------------------+
```

---

## IP Addressing Table

| Device | Interface | IP Address/Mask | Default Gateway |
| :--- | :--- | :--- | :--- |
| **R1** | Gi0/1 | `192.168.1.1/24` | N/A |
| | Gi0/2 | `10.1.2.1/30` | N/A |
| | Gi0/3 | `10.1.3.1/30` | N/A |
| **R2** | Gi0/1 | `192.168.2.1/24` | N/A |
| | Gi0/2 | `10.1.2.2/30` | N/A |
| | Gi0/3 | `10.2.3.2/30` | N/A |
| **R3** | Gi0/1 | `192.168.3.1/24` | N/A |
| | Gi0/2 | `10.1.3.2/30` | N/A |
| | Gi0/3 | `10.2.3.1/30` | N/A |
| **PC1** | eth0 | `192.168.1.10/24` | `192.168.1.1` |
| **PC2** | eth0 | `192.168.2.10/24` | `192.168.2.1` |
| **PC3** | eth0 | `192.168.3.10/24` | `192.168.3.1` |

---

## Module 1: Baseline Connectivity Analysis

In this first module, we will verify the initial state of the network. The lab has been pre-configured with IP addresses, but no routing is in place. We will confirm that hosts can communicate within their own LANs but cannot reach any remote networks.

### Step 1: Verify Local Connectivity on PC1

1. Open the console for **PC1**.
2. Ping the default gateway, **R1**, to confirm local network connectivity.

```bash
PC1# ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: seq=0 ttl=64 time=1.467 ms
64 bytes from 192.168.1.1: seq=1 ttl=64 time=0.992 ms
--- 192.168.1.1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
```

### Step 2: Test Remote Connectivity from PC1

1. From **PC1**, attempt to ping **PC2** at `192.168.2.10`.

```bash
PC1# ping 192.168.2.10
PING 192.168.2.10 (192.168.2.10): 56 data bytes
--- 192.168.2.10 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss
```

**Question:** Why did the ping fail? Where did the packet get dropped?

### Step 3: Analyze the Routing Table on R1

1. Open the console for **R1**.
2. Examine the routing table to understand what networks R1 knows about.

```text
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       ... (output truncated) ...

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.1.2.0/30 is directly connected, GigabitEthernet0/2
L        10.1.2.1/32 is directly connected, GigabitEthernet0/2
C        10.1.3.0/30 is directly connected, GigabitEthernet0/3
L        10.1.3.1/32 is directly connected, GigabitEthernet0/3
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
```

**Analysis:** R1 only knows about its directly connected networks. It has no information about the `192.168.2.0/24` network where PC2 resides. Therefore, when the ping packet from PC1 arrived at R1, R1 had nowhere to send it and dropped it.

---

## Module 2: Implementing Specific Static Routes

Our mission is to establish full connectivity. We will now manually add static routes to each router, teaching them how to reach all the remote networks in our topology.

### Step 1: Configure Static Routes on R1

We need to teach R1 how to reach the LAN of R2 (`192.168.2.0/24`) and the LAN of R3 (`192.168.3.0/24`).

1. On **R1**, enter global configuration mode.
2. Create a static route to `192.168.2.0/24` via R2's Gi0/2 interface (`10.1.2.2`).
3. Create a static route to `192.168.3.0/24` via R3's Gi0/2 interface (`10.1.3.2`).

```text
R1# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# ip route 192.168.2.0 255.255.255.0 10.1.2.2
R1(config)# ip route 192.168.3.0 255.255.255.0 10.1.3.2
```

### Step 2: Verify the Routing Table on R1

1. From configuration mode, use a `do` command to verify the new routes.

```text
R1(config)# do show ip route
... (output truncated) ...
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.1.2.0/30 is directly connected, GigabitEthernet0/2
L        10.1.2.1/32 is directly connected, GigabitEthernet0/2
C        10.1.3.0/30 is directly connected, GigabitEthernet0/3
L        10.1.3.1/32 is directly connected, GigabitEthernet0/3
S     192.168.2.0/24 [1/0] via 10.1.2.2
S     192.168.3.0/24 [1/0] via 10.1.3.2
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
```

**Analysis:** We can now see two new routes, marked with an `S` for Static. The `[1/0]` indicates the administrative distance (1 for static routes) and the metric (0, not used by static routes).

### Step 3: Configure Static Routes on R2 and R3

Now, you must complete the configuration. The logic is the same. Each router needs a route for every network it is not directly connected to.

1. **On R2:** Configure static routes for `192.168.1.0/24` (via R1) and `192.168.3.0/24` (via R3).
2. **On R3:** Configure static routes for `192.168.1.0/24` (via R1) and `192.168.2.0/24` (via R2).

*(Use the IP addressing table and the logic from Step 1 to determine the correct next-hop addresses.)*

### Step 4: Verify Full End-to-End Connectivity

1. Go back to **PC1**.
2. Ping **PC2** (`192.168.2.10`) and **PC3** (`192.168.3.10`). Both should now be successful!

```bash
PC1# ping 192.168.2.10
PING 192.168.2.10 (192.168.2.10): 56 data bytes
64 bytes from 192.168.2.10: seq=0 ttl=62 time=2.531 ms
...
PC1# ping 192.168.3.10
PING 192.168.3.10 (192.168.3.10): 56 data bytes
64 bytes from 192.168.3.10: seq=0 ttl=62 time=2.815 ms
...
```

### > Exam-Level Deep Dive: Symmetric vs. Asymmetric Routing

When you first added the route on R1, why did the ping still fail? This is a classic CCNA-level problem. A ping requires a two-way conversation: an ICMP Echo Request and an ICMP Echo Reply.
* **Request Path:** `PC1 -> R1 -> R2 -> PC2` (This worked after adding the route on R1).
* **Reply Path:** `PC2 -> R2 -> ???` (R2 did not know how to get back to `192.168.1.0/24`, so it dropped the reply).

For connectivity to work, routing must be **symmetric** (or at least functional in both directions). You must ensure there is a return path for all traffic.

---

## Module 3: Demonstrating Static Path Failure

Static routes are simple but not adaptive. If the path they point to goes down, they do not automatically find a new one. Let's prove this.

### Step 1: Start a Continuous Ping

1. On **PC1**, start a continuous ping to **PC3**.

```bash
PC1# ping 192.168.3.10 -c 100
```
This will send 100 pings. Let it run in the background.

### Step 2: Simulate a Link Failure

We will disable the direct link between R1 and R3.

1. On **R1**, enter configuration mode for the interface connecting to R3.
2. Shut down the interface.

```text
R1# configure terminal
R1(config)# interface GigabitEthernet0/3
R1(config-if)# shutdown
```

### Step 3: Observe the Failure

1. Switch back to the console of **PC1**.
2. You will see the pings begin to fail.

```text
...
64 bytes from 192.168.3.10: icmp_seq=5 ttl=62 time=3.14 ms
Request timeout for icmp_seq 6
Request timeout for icmp_seq 7
...
```

**Why?** R1's routing table still has only one route to `192.168.3.0/24`, and it points down a link that is now dead. The router has no mechanism to find the alternate path through R2.

### Step 4: Restore the Link

1. On **R1**, bring the interface back up.

```text
R1(config-if)# no shutdown
R1(config-if)# end
```
2. Observe that connectivity on PC1 is restored almost immediately.

---

## Module 4: Implementing a Floating Static Route

To build resilience, we can configure a backup route—a "floating" static route. It "floats" above the primary route in the routing table by having a higher (less preferred) administrative distance (AD). It only gets installed in the routing table if the primary route disappears.

### Step 1: Configure a Floating Static Route on R1

We will create a backup route to `192.168.3.0/24` that goes via R2. We will give it an AD of 10, making it less preferred than the default AD of 1 for a static route.

1. On **R1**, enter global configuration mode.
2. Add a new static route to `192.168.3.0/24` via R2, and append the administrative distance `10` at the end of the command.

```text
R1# configure terminal
R1(config)# ip route 192.168.3.0 255.255.255.0 10.1.2.2 10
```

### Step 2: Verify the Route is NOT Active

1. Examine the routing table on **R1**.

```text
R1(config)# do show ip route 192.168.3.0
Routing entry for 192.168.3.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.1.3.2
      Route metric is 0, traffic share count is 1
```
**Analysis:** The routing table only shows the primary route (via `10.1.3.2`, with AD 1). The router knows about the floating route but will not use it as long as the better primary route is available.

### Step 3: Test the Failover

1. Start a continuous ping from **PC1** to **PC3** again.
2. On **R1**, shut down the `GigabitEthernet0/3` interface again.

### Step 4: Observe the Resilient Failover

1. Watch the ping on **PC1**. You might see one or two packets drop, but connectivity will quickly resume!
2. Immediately check the routing table on **R1** while the link is down.

```text
R1# show ip route 192.168.3.0
Routing entry for 192.168.3.0/24
  Known via "static", distance 10, metric 0
  Routing Descriptor Blocks:
  * 10.1.2.2
      Route metric is 0, traffic share count is 1
```

**Success!** The primary route vanished when the interface went down, so the router immediately installed our floating static route with an AD of 10. Traffic is now flowing `R1 -> R2 -> R3`.

**Don't forget to bring the interface on R1 back up (`no shutdown`) before proceeding!**

---

## Conclusion & Review

Congratulations on completing the Static Routing Foundations lab! You have successfully transformed a disconnected set of networks into a fully connected and resilient infrastructure using only static routing.

**Skills Validated:**
* Configuration of next-hop static routes.
* Analysis of the IPv4 routing table.
* Troubleshooting connectivity by verifying symmetric routing.
* Implementation of floating static routes for path redundancy.
* Understanding of Administrative Distance.

This manual process highlights the control and predictability of static routing but also underscores its primary weakness: scalability. Imagine performing this process for hundreds of routes! This experience provides the perfect foundation for understanding why dynamic routing protocols like OSPF and EIGRP are essential for larger networks.

### Answer Key

* **Module 1, Step 2 Question:** The ping failed because the first-hop router, R1, did not have a route to the destination network `192.168.2.10`. It had no entry in its routing table for `192.168.2.0/24` and therefore dropped the packet.
