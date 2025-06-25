# CCNA Lab Guide: Advanced Full-Mesh EtherChannel & PVST+

## Lab Objective

In this advanced scenario, you are a network engineer tasked with optimizing a highly redundant, full-mesh network that serves two distinct user groups (USERS on VLAN 10 and SERVERS on VLAN 20). The primary goal is to analyze the complex, load-balanced Per-VLAN Spanning Tree (PVST+) topology, troubleshoot a common EtherChannel misconfiguration, and then strategically implement a working EtherChannel to increase bandwidth, all while adhering to security best practices.

## Lab Diagram

This diagram represents our full-mesh core, with dual links between every switch and a dedicated host for each VLAN group.

### **Part 1: Initial Topology and STP Analysis**

A skilled engineer must first understand the existing environment. We need to verify our baseline configuration and see how PVST+ has created different logical topologies for our two VLANs. For this lab, **SW1 is the root for VLAN 10**, and **SW2 is the root for VLAN 20**.

#### **Step 1: Verify Interface Status and VLANs**

On `SW1`, let's get a high-level overview of our interfaces. This is a great first step to confirm physical connectivity and access port VLAN assignments.

```
SW1# show interfaces status

Port      Name               Status       Vlan       Duplex  Speed Type
...
Gi1/2                        connected    10           a-full a-1000 10/100/1000BaseTX
...
```

* **What This Does:** The `show interfaces status` command gives us a quick summary of which ports are connected, their VLAN assignment, duplex, and speed. We can immediately confirm that our link to PC1 is up and correctly assigned to VLAN 10.

#### **Step 2: Verify Trunk Configuration**

Now, let's verify the trunk links on `SW1`. Pay close attention to the **Native vlan** column.

```
SW1# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Gi0/0       on           802.1q         trunking      199
Gi0/1       on           802.1q         trunking      199
... (output truncated) ...

Port        Vlans allowed and active in management domain
Gi0/0       1,10,20,199
...
```

* **What This Does:** This command is essential for troubleshooting trunk links. We confirm three critical details:
  1. The ports are actively `trunking`.
  2. The `Native vlan` is correctly set to `199`.
  3. VLANs `10` and `20` are active and allowed across the trunks.

#### **Step 3: Analyze the STP Topology for VLAN 10 (USERS)**

Let's examine the logical topology for VLAN 10 from the perspective of `SW3`.

```
SW3# show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     aabb.cc00.0100  (This is SW1)
             Cost        19
             Port        1 (GigabitEthernet0/0)
  Bridge ID  Priority    32778
             Address     aabb.cc00.0300
...
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 19        128.1    P2p
Gi0/1               Altn BLK 19        128.2    P2p
Gi0/2               Altn BLK 19        128.3    P2p
... (all other inter-switch links are BLK) ...
```

* **Analysis:** For VLAN 10, `SW3`'s **Root Port** (its best path to the Root Bridge) is `Gi0/0`, the direct link to `SW1`. Every other path is in the `BLK` (blocking) state to prevent loops.

#### **Step 4: Analyze the STP Topology for VLAN 20 (SERVERS)**

Now, let's look at the *same switch* (`SW3`) but for VLAN 20.

```
SW3# show spanning-tree vlan 20

VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     aabb.cc00.0200  (This is SW2)
             Cost        19
             Port        3 (GigabitEthernet0/2)
  Bridge ID  Priority    32778
             Address     aabb.cc00.0300
...
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Altn BLK 19        128.1    P2p
Gi0/1               Altn BLK 19        128.2    P2p
Gi0/2               Root FWD 19        128.3    P2p
... (all other inter-switch links are BLK) ...
```

* **Analysis:** This is PVST+ in action! For VLAN 20, `SW3` has chosen a different Root Port (`Gi0/2`, the link to `SW2`) because `SW2` is the root for this VLAN. The network is actively load-balancing traffic across different physical links based on the VLAN.

### **Part 2: Troubleshooting Challenge - A Common Misconfiguration**

Now, we will attempt to create an EtherChannel between `SW2` and `SW3`. We are going to make a common configuration error to see what happens.

#### **Step 1: Configure a `passive-passive` LACP Channel**

We will attempt to bundle `Gi0/2` and `Gi0/3` on `SW2` and `SW3` into `channel-group 1` using the `passive` mode on both sides.

On `SW2`:
```
SW2# conf t
SW2(config)# interface range Gi0/2 - 3
SW2(config-if-range)# shutdown
SW2(config-if-range)# channel-group 1 mode passive
Creating a port-channel interface Port-channel 1
SW2(config-if-range)# no shutdown
SW2(config-if-range)# end
```

On `SW3`:
```
SW3# conf t
SW3(config)# interface range Gi0/2 - 3
SW3(config-if-range)# shutdown
SW3(config-if-range)# channel-group 1 mode passive
Creating a port-channel interface Port-channel 1
SW3(config-if-range)# no shutdown
SW3(config-if-range)# end
```

#### **Step 2: Verify the (Failed) EtherChannel**

Now, let's check the status on `SW2`. Did the EtherChannel form?

```
SW2# show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
...
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)         LACP      Gi0/2(I)    Gi0/3(I)
```

**Challenge Question:** The EtherChannel did not form! The ports are marked as `(I)`, meaning they are `stand-alone`. The Port-channel itself is `(SD)`, meaning Layer 2 (`S`) but `down` (`D`). Based on the LACP `passive` mode we configured, can you explain why the negotiation failed?

### **Part 3: Implementing the Correct Solution**

You've diagnosed the problem, now let's implement the fix.

#### **The Solution and Explanation**

The EtherChannel failed because when both sides of an LACP negotiation are set to `passive`, neither side will actively initiate the conversation. They are both waiting for the other side to start. To fix this, at least one side must be set to `active`.

#### **Step 1: Correct the LACP Mode**

Let's fix the configuration on `SW2` by changing its mode to `active`. We will leave `SW3` as `passive`. This `active-passive` combination is a valid and common configuration.

On `SW2`:
```
SW2(config)# interface range Gi0/2 - 3
SW2(config-if-range)# channel-group 1 mode active
```

#### **Step 2: Re-Verify the EtherChannel**

Give the protocol a few seconds to converge, and then run the verification command on `SW2` again.

```
SW2# show etherchannel summary
...
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/2(P)    Gi0/3(P)
```

* **What This Does:** Success! The channel is now `(SU)` - up and in use - and the ports are `(P)` - bundled. Now we can apply the Layer 2 trunk configuration to the logical interface to make it fully operational.
  ```
  SW2(config)# interface port-channel 1
  SW2(config-if)# switchport mode trunk
  SW2(config-if)# switchport trunk native vlan 199
  ```
  *(Remember to do the same on SW3's Port-channel 1 interface)*

### **Part 4: Final Implementation and Impact Analysis**

Now that you've seen how to troubleshoot, let's create a production EtherChannel between `SW1` and `SW4` and analyze its impact.

#### **Step 1: Create the SW1-SW4 EtherChannel**

We will bundle `Gi1/0` and `Gi1/1` on both switches into `channel-group 4` using the `active-active` mode.

On `SW1`:
```
SW1(config)# interface range Gi1/0 - 1
SW1(config-if-range)# shutdown
SW1(config-if-range)# channel-group 4 mode active
SW1(config-if-range)# exit
SW1(config)# interface port-channel 4
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 199
SW1(config-if)# no shutdown
SW1(config-if)# interface range Gi1/0 - 1
SW1(config-if-range)# no shutdown
```

On `SW4`:
```
SW4(config)# interface range Gi1/0 - 1
SW4(config-if-range)# shutdown
SW4(config-if-range)# channel-group 4 mode active
SW4(config-if-range)# exit
SW4(config)# interface port-channel 4
SW4(config-if)# switchport mode trunk
SW4(config-if)# switchport trunk native vlan 199
SW4(config-if)# no shutdown
SW4(config-if)# interface range Gi1/0 - 1
SW4(config-if-range)# no shutdown
```

#### **Step 2: Analyze the Final STP Topology on SW4**

How did our new EtherChannel affect the two different STP topologies on `SW4`?

```
SW4# show spanning-tree vlan 10,20 summary
...
Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0010                     10         0        0          4         14
VLAN0020                     10         0        0          4         14
...
SW4# show spanning-tree vlan 10
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po4                 Root FWD 12        128.260  P2p

SW4# show spanning-tree vlan 20
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/2               Root FWD 19        128.3    P2p
Po4                 Altn BLK 12        128.260  P2p
```

* **Final Analysis:** This is the perfect outcome.
  * **For VLAN 10:** The new EtherChannel (`Po4`) has become the Root Port for `SW4`. Its STP cost of `12` is superior to any single link's cost of `19`, providing a high-speed path to the VLAN 10 root (`SW1`).
  * **For VLAN 20:** The Root Port remains the direct link to `SW2` (`Gi0/2`), as that is the most efficient path to the VLAN 20 root. Our new EtherChannel is correctly identified as a backup path and placed in the `Altn BLK` state for this VLAN.

You have successfully diagnosed a problem, implemented a robust solution, and engineered the network for both high availability and optimal performance.
