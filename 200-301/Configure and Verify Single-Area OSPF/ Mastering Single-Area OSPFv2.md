# CCNA Lab: Mastering Single-Area OSPFv2

Welcome to this advanced CCNA lab, where you'll step into the role of a network engineer tasked with integrating a new router into an existing OSPF network. This isn't just about commands; it's about understanding the *why* behind the configuration. We will explore OSPF neighbor adjacencies, advertising different subnet types, path selection, and advanced security best practices.

**Lab Duration:** 35-55 minutes

### Network Diagram

The physical topology consists of three routers in a triangle. Each router also has logical loopback interfaces (not shown in the physical diagram) to simulate end-user or service LAN segments.

![Network Diagram](https://i.imgur.com/3sZ2c3n.png)

### IP Addressing Table

| Device | Interface | IP Address/Mask | Notes |
| :--- | :--- | :--- | :--- |
| **R1** | Loopback0 | `1.1.1.1/32` | Mgmt Network (Host Route) |
| | Loopback1 | `100.1.1.1/24` | LAN Segment A |
| | Loopback2 | `100.1.2.1/24` | LAN Segment B |
| | GigabitEthernet0/0 | `10.1.2.1/24` | Link to R2 |
| | GigabitEthernet0/1 | `10.1.3.1/24` | Link to R3 |
| **R2** | Loopback0 | `2.2.2.2/32` | Mgmt Network (Host Route) |
| | Loopback1 | `100.2.1.1/24` | LAN Segment C |
| | Loopback2 | `100.2.2.1/24` | LAN Segment D |
| | GigabitEthernet0/0 | `10.1.2.2/24` | Link to R1 |
| | GigabitEthernet0/1 | `10.2.3.2/24` | Link to R3 |
| **R3** | Loopback0 | `3.3.3.3/32` | Mgmt Network (Host Route) |
| | Loopback1 | `100.3.1.1/24` | LAN Segment E |
| | Loopback2 | `100.3.2.1/24` | LAN Segment F |
| | GigabitEthernet0/0 | `10.2.3.3/24` | Link to R2 |
| | GigabitEthernet0/1 | `10.1.3.3/24` | Link to R1 |

## Lab Guide

### Module 1: Initial State Verification

R1 and R2 are pre-configured for OSPF Area 0. Verify their status.

**Step 1: Verify OSPF on R1**
1.  Access the console of **R1**.
2.  Inspect the routing table. You should see OSPF-learned routes for R2's networks. Notice how the `/32` loopback is advertised differently from the `/24` loopbacks.

    ```
    R1# show ip route ospf
    ```

    **Sample Output:**
    ```
    O        2.2.2.2/32 [110/2] via 10.1.2.2, 00:01:30, GigabitEthernet0/0
    O     100.2.1.0/24 [110/2] via 10.1.2.2, 00:01:30, GigabitEthernet0/0
    O     100.2.2.0/24 [110/2] via 10.1.2.2, 00:01:30, GigabitEthernet0/0
    ```

### Module 2: Core OSPF Configuration & Verification

Now, you will configure R3 to join the OSPF domain and advertise all its networks.

**Step 1: Configure OSPF on R3**
1.  Access the console of **R3**.
2.  Enable OSPF with Process ID 1 and manually set the Router ID.

    ```
    R3# configure terminal
    R3(config)# router ospf 1
    R3(config-router)# router-id 3.3.3.3
    ```

3.  Enable OSPF on all of R3's interfaces using a single, comprehensive network command.

    ```
    R3(config-router)# network 0.0.0.0 255.255.255.255 area 0
    R3(config-router)# end
    ```

**Step 2: Verify Full Network Convergence**
1.  On R3, check for new OSPF neighbor relationships with R1 and R2.

    ```
    R3# show ip ospf neighbor
    ```
    **Sample Output:**
    ```
    Neighbor ID     Pri   State           Dead Time   Address         Interface
    1.1.1.1           0   FULL/  -        00:00:31    10.1.3.1        GigabitEthernet0/1
    2.2.2.2           0   FULL/  -        00:00:35    10.2.3.2        GigabitEthernet0/0
    ```

2.  Examine R3's routing table. It should now be populated with routes to all networks from R1 and R2.

    ```
    R3# show ip route ospf
    ```

    **Sample Output:**
    ```
    O        1.1.1.1/32 [110/2] via 10.1.3.1, 00:03:10, GigabitEthernet0/1
    O        2.2.2.2/32 [110/2] via 10.2.3.2, 00:03:05, GigabitEthernet0/0
    O     10.1.2.0/24 [110/2] via 10.2.3.2, 00:03:05, GigabitEthernet0/0
                         [110/2] via 10.1.3.1, 00:03:05, GigabitEthernet0/1
    O     100.1.1.0/24 [110/2] via 10.1.3.1, 00:03:10, GigabitEthernet0/1
    O     100.1.2.0/24 [110/2] via 10.1.3.1, 00:03:10, GigabitEthernet0/1
    O     100.2.1.0/24 [110/2] via 10.2.3.2, 00:03:05, GigabitEthernet0/0
    O     100.2.2.0/24 [110/2] via 10.2.3.2, 00:03:05, GigabitEthernet0/0
    ```

### Module 3: OSPF Path Control & Metric Manipulation

The task is the same: force traffic from R3 destined to the `10.1.2.0/24` network (the link between R1 and R2) to go via R1.

**Step 1: Modify the OSPF Cost on R3**
1.  Increase the OSPF cost on the interface connected to R2 (`GigabitEthernet0/0`).
    ```
    R3# configure terminal
    R3(config)# interface GigabitEthernet0/0
    R3(config-if)# ip ospf cost 100
    R3(config-if)# end
    ```

**Step 2: Verify the Path Change**
1.  Re-examine R3's routing table and confirm the path to `10.1.2.0/24` is now solely via R1.
    ```
    R3# show ip route ospf
    ```
    **Sample Output:**
    ```
    ...
    O     10.1.2.0/24 [110/2] via 10.1.3.1, 00:00:15, GigabitEthernet0/1
    ...
    ```

### Module 4: OSPF Security & Optimization (Advanced)

The superior method for securing OSPF is to make it passive *by default* and then selectively activate it only on the necessary transit interfaces.

**Step 1: Reconfigure for `passive-interface default`**
1.  On **R1, R2, and R3**, enter OSPF configuration. First, make all interfaces passive. Second, use the `no` command to specifically enable OSPF Hello packets on the router-to-router links.

    On **all three routers (R1, R2, R3)**, apply this logic:
    ```
    Router# configure terminal
    Router(config)# router ospf 1
    Router(config-router)# passive-interface default
    Router(config-router)# no passive-interface GigabitEthernet0/0
    Router(config-router)# no passive-interface GigabitEthernet0/1
    Router(config-router)# end
    ```
    *Applying this configuration to all three routers ensures they can all form adjacencies on their appropriate physical links.*

**Step 2: Verify the Security Posture**
1.  On any router, check the OSPF-enabled interfaces. Note that all loopbacks are now correctly in a `PASSIVE` state.
    ```
    R3# show ip ospf interface brief
    ```
    **Sample Output:**
    ```
    Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
    Lo1          1     0               100.3.1.1/24       1     PASSIVE 0/0
    Lo2          1     0               100.3.2.1/24       1     PASSIVE 0/0
    Lo0          1     0               3.3.3.3/32         1     PASSIVE 0/0
    Gi0/1        1     0               10.1.3.3/24        1     P2P   1/1
    Gi0/0        1     0               10.2.3.3/24        100   P2P   1/1
    ```

---
## Further Exploration (Optional)

### Advanced Topic 1: Peeking into the LSDB
You've seen the *result* of OSPF in the routing table, but how does it get there? You can view the Link-State Database (LSDB) to see the raw topological data OSPF uses.

**Step 1: Examine a Router LSA**
1.  Examine R3's LSA from the perspective of R1.
    ```
    R1# show ip ospf database router 3.3.3.3
    ```
    **Analysis:** This output shows you every link R3 is advertising, including its neighbors and its local "stub" networks (the loopbacks). For stub networks, the `Link ID` is the network address and the `Link Data` is the subnet mask, which is how OSPF communicates the full subnet information.

### Advanced Topic 2: The Mystery of the Missing DR and BDR
You may have heard of a Designated Router (DR) and Backup DR (BDR) in OSPF. Why haven't we seen one in this lab?

**What is the purpose of a DR and BDR?**
Imagine a network segment with 10 routers connected to a single Ethernet switch (a multi-access network). Without a DR, every router would need to form a full neighbor adjacency with every other router to exchange routing updates. This would create a massive number of adjacencies (`n*(n-1)/2`, so 45 adjacencies for 10 routers!) and flood the network with redundant LSAs.

To solve this, OSPF elects a **Designated Router (DR)** on multi-access networks. All other routers (called DROthers) form a full adjacency *only* with the DR and BDR. They send their LSAs to the DR, which then relays the LSA to all other routers on the segment. This drastically reduces the number of adjacencies and redundant updates, making OSPF much more efficient. The **Backup DR (BDR)** listens in and takes over immediately if the DR fails.

**Why is there no DR in our lab?**
A DR/BDR election only happens on **multi-access** network types, like Ethernet. The links in our lab are serial connections, which OSPF treats by default as **Point-to-Point**. On a point-to-point link, there can only ever be two routers, so the complexity of a DR is unnecessary.

**Step 1: Verify the OSPF Network Type**
1.  On R1, let's examine one of its OSPF interfaces in detail.
    ```
    R1# show ip ospf interface GigabitEthernet0/0
    ```
    **Sample Output:**
    ```
    GigabitEthernet0/0 is up, line protocol is up
      Internet Address 10.1.2.1/24, Area 0, Process ID 1, Router ID 1.1.1.1
      Network Type POINT_TO_POINT, Cost: 1
      Topology-MTID    Cost    Disabled    Shutdown      Topology Name
            0           1         no          no            Base
      Transmit Delay is 1 sec, State POINT_TO_POINT
    ...
    ```
**Analysis:** The output clearly states `Network Type POINT_TO_POINT`. Because of this, no DR election occurs. You will not see any lines in the output mentioning a Designated Router, and the `State` is simply `POINT_TO_POINT`. If this were an Ethernet segment, the network type would be `BROADCAST` and you would see additional lines identifying the DR, BDR, and the router's own state in the election (e.g., DROTHER). This distinction is a fundamental concept in OSPF operations.

