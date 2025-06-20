# CCNA Lab Guide: VLANs, Trunks, and Network Segmentation

**Candidate:** You, the CCNA Aspirant
**Mission:** To evolve a simple, flat network into a properly segmented architecture using VLANs and an 802.1Q trunk. This lab focuses on a meticulous, real-world implementation and troubleshooting process.

### Introduction: From Flat to Segmented

Welcome to the lab. Today, we're moving beyond basic connectivity. The scenario begins with a simple, flat Layer 2 network where every device can communicate freely. All devices exist in the `10.10.1.0/24` subnet and the default `VLAN 1`. However, a new business requirement has been introduced to create a separate, secure network for the new "Engineering" department and isolate their traffic.

This lab will guide you through the process of migrating the configuration to one that uses two VLANs. You will configure and verify VLANs, establish a secure 802.1Q trunk link, and deliberately create—then resolve—a common native VLAN mismatch error.

**Core Learning Objectives:**

* Configure and verify standard VLANs to segment a network into separate broadcast domains.
* Configure and verify an IEEE 802.1Q trunk link to pass traffic for multiple VLANs between switches.
* Diagnose and resolve common trunking issues, specifically native VLAN mismatches.
* Implement security best practices for trunk links, including assigning a dedicated, non-user VLAN as the native VLAN.

### Network Diagram (Target State)

This diagram shows our goal. We will create VLAN 20 for the Engineering department (PC2 and PC4) and leave the Sales department (PC1 and PC3) in a re-purposed VLAN 10. The topology requires two switches and four PCs, with two PCs connected to each switch. The switches are connected by a single link.

### IP Addressing Schema (Initial State)

| Device | Interface | IP Address/Subnet | Default Gateway | Final VLAN |
| :--- | :--- | :--- | :--- | :--- |
| PC1 | eth0 | 10.10.1.11/24 | 10.10.1.1 | 10 (Sales) |
| PC2 | eth0 | 10.10.1.12/24 | 10.10.1.1 | 20 (Engineering) |
| PC3 | eth0 | 10.10.1.13/24 | 10.10.1.1 | 10 (Sales) |
| PC4 | eth0 | 10.10.1.14/24 | 10.10.1.1 | 20 (Engineering) |
| SW1 | Gi0/0 | N/A (Trunk) | N/A | Trunk |
| SW2 | Gi0/0 | N/A (Trunk) | N/A | Trunk |

---

### **Part 1: Baseline Network Verification**

First, let's verify the initial "flat network" state. The lab starts with a fully converged Layer 2 network where only the default VLAN, VLAN 1, exists, and initial testing verifies full any-to-any connectivity.

1.  **Console into PC1.** Ping every other PC in the topology (PC2, PC3, and PC4).

    ```bash
    # On PC1
    ping 10.10.1.12 -c 4
    ping 10.10.1.13 -c 4
    ping 10.10.1.14 -c 4
    ```

2.  **Verification Question:** Did all pings succeed? Why?

3.  **Examine the Initial Switch State.** Console into **SW1** and use the following commands to see the default configuration.

    ```bash
    # On SW1
    show vlan brief
    show interfaces status
    ```

    Notice that all connected interfaces (Gi0/1, Gi0/2) are active and assigned to VLAN 1 by default.

---

### **Part 2: The "Wrong" Way - IP-Based Segmentation**

To understand *why* VLANs are necessary, we will first implement the separation incorrectly using only IP addressing. The catalyst for change is the need to segment the network for the "Engineering" department. We'll move the two Engineering PCs to a new subnet.

1.  **Re-IP the Engineering PCs.** On PC2 and PC4, change the IP address to the `10.10.2.0/24` subnet.

    ```bash
    # On PC2
    ip addr del 10.10.1.12/24 dev eth0
    ip addr add 10.10.2.12/24 dev eth0

    # On PC4
    ip addr del 10.10.1.14/24 dev eth0
    ip addr add 10.10.2.14/24 dev eth0
    ```

2.  **Test Connectivity.**
    * From **PC1**, try to ping **PC3** (`10.10.1.13`). It should **succeed**.
    * From **PC1**, try to ping **PC2** (`10.10.2.12`). It should **fail**.
    * From **PC2**, try to ping **PC4** (`10.10.2.14`). It should **fail**.

3.  **Verification & Analysis:** We have achieved some isolation, but it's flawed. The hosts are in different IP subnets, so they can't communicate without a router. More importantly, why did the ping from PC2 to PC4 fail? They are both in the "Engineering" subnet.

    The answer is that all ports are still in the same broadcast domain (VLAN 1), exposing them to unnecessary broadcast traffic from the other hosts. When PC2 sends an ARP request for PC4, that broadcast floods out all ports in VLAN 1 on SW1. SW2 receives it and also floods it to all its VLAN 1 ports. PC4 receives it, but its return unicast frame cannot get back to PC2. This is an inefficient and insecure setup.

---

### **Part 3: The "Right" Way - VLAN Implementation**

Now, let's correctly segment the network by migrating the configuration to one that uses VLANs. First, revert the IP changes on PC2 and PC4.

```bash
# On PC2
ip addr del 10.10.2.12/24 dev eth0
ip addr add 10.10.1.12/24 dev eth0

# On PC4
ip addr del 10.10.2.14/24 dev eth0
ip addr add 10.10.1.14/24 dev eth0
```

1.  **Create VLANs on Both Switches.** A VLAN must be created on each switch that needs to service it. We will create VLAN 10 for Sales and VLAN 20 for Engineering.

    ```bash
    # On SW1 and SW2
    configure terminal
    vlan 10
    name Sales
    vlan 20
    name Engineering
    exit
    ```

2.  **Verify VLAN Creation.** On both switches, use a robust verification methodology. Don't rely on just one command.

    ```bash
    # On SW1 and SW2
    show vlan brief
    show vlan name Sales
    show vlan id 20
    ```

3.  **Assign Ports to VLANs.** On each switch, assign the correct access ports to the new VLANs.
    * **SW1**: Gi0/1 -> VLAN 10 (Sales), Gi0/2 -> VLAN 20 (Engineering)
    * **SW2**: Gi0/1 -> VLAN 10 (Sales), Gi0/2 -> VLAN 20 (Engineering)

    ```bash
    # On SW1
    configure terminal
    interface GigabitEthernet0/1
    switchport mode access
    switchport access vlan 10
    !
    interface GigabitEthernet0/2
    switchport mode access
    switchport access vlan 20
    exit

    # On SW2
    configure terminal
    interface GigabitEthernet0/1
    switchport mode access
    switchport access vlan 10
    !
    interface GigabitEthernet0/2
    switchport mode access
    switchport access vlan 20
    exit
    ```

4.  **Meticulous Port-Level Verification.** Confirm your changes on both switches.

    ```bash
    # On SW1 and SW2
    show vlan brief
    show interface status
    show interface g0/1 switchport
    show interface g0/2 switchport
    ```

    * **Verification Question:** In the `show interface <X> switchport` output, what is the "Administrative Mode" and the "Operational Mode"?

5.  **Test Connectivity.**
    * From PC1 (`10.10.1.11`), ping PC3 (`10.10.1.13`). **This should fail.**
    * From PC2 (`10.10.1.12`), ping PC4 (`10.10.1.14`). **This should fail.**
    * Why? The hosts are now in separate broadcast domains, but the link between SW1 and SW2 is still a simple access port in VLAN 1. It doesn't know how to pass traffic for VLANs 10 and 20. We need a trunk.

---

### **Part 4: Implementing the 802.1Q Trunk**

A trunk is a point-to-point link that can carry traffic for multiple VLANs. We will manually configure the link between SW1 and SW2 (Gi0/0) as a manually configured 802.1Q trunk.

1.  **Configure Trunk on SW1 and SW2.**

    ```bash
    # On SW1 and SW2
    configure terminal
    interface GigabitEthernet0/0
    switchport trunk encapsulation dot1q
    switchport mode trunk
    exit
    ```

> #### **Exam-Level Deep Dive: Trunking Protocols & Modes**
>
> * **`switchport trunk encapsulation dot1q`**: This command explicitly sets the trunking protocol to IEEE 802.1Q. The lab requires you to manually set the trunking protocol to dot1q, explaining that it is the modern standard over older, proprietary protocols like ISL.
> * **`switchport mode trunk`**: We are explicitly setting the interface to be a trunk. The student is guided to manually configure the switchport mode to trunk rather than relying on the default dynamic auto-negotiation (DTP). Relying on DTP is not a security best practice, as it can lead to unintended trunks forming if a misconfigured device is connected. Manually setting the mode to `trunk` disables DTP and hard-codes the interface's function.

2.  **Verify the Trunk.** Use a comprehensive set of commands on both switches.

    ```bash
    # On SW1 and SW2
    show interfaces trunk
    show interface g0/0 switchport
    ```

    * **Verification Question:** In the `show interfaces trunk` output, what is the status of the trunk? What VLANs are allowed to cross it?

3.  **Test Cross-Switch Connectivity.** Now that the trunk is up, let's re-run our tests.
    * From **PC1** (VLAN 10), ping **PC3** (VLAN 10). It should **SUCCEED**.
    * From **PC2** (VLAN 20), ping **PC4** (VLAN 20). It should **SUCCEED**.
    * From **PC1** (VLAN 10), ping **PC2** (VLAN 20). It should **FAIL**.
    * From **PC1** (VLAN 10), ping **PC4** (VLAN 20). It should **FAIL**.

---

### **Part 5: The "Aha!" Moment - Native VLAN Mismatch**

Now for a critical troubleshooting exercise. The *native VLAN* on an 802.1Q trunk is the one VLAN whose traffic crosses the trunk *without* an 802.1Q tag. The lab deliberately engineers a native VLAN mismatch.

1.  **Create a Mismatch.** On **SW1 only**, change the native VLAN to 99.

    ```bash
    # On SW1 ONLY
    configure terminal
    interface GigabitEthernet0/0
    switchport trunk native vlan 99
    exit
    ```

2.  **Observe the Error!** Wait about 30-60 seconds. You should see log messages appearing on the console of **both** switches, complaining about a native VLAN mismatch. This is Cisco Discovery Protocol (CDP) at work!

    ```
    %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/0 (99), with SW2 GigabitEthernet0/0 (1).
    ```

3.  **Verify the Symptom.** Even with the error, the trunk is technically "up". But it's in a dangerous and unpredictable state. The mismatch is caused by an inconsistent configuration across the link. Use this command on both switches to confirm the mismatch.

    ```bash
    # On SW1 and SW2
    show interfaces trunk
    ```

    Un-tagged traffic sent from SW1 (like CDP or DTP) is expected to be in VLAN 99, but SW2 receives it and believes it belongs to its own native VLAN, VLAN 1. This is a security risk and causes protocols to misbehave.

4.  **Resolve the Mismatch.** On **SW2**, apply the correct configuration to resolve the error.

    ```bash
    # On SW2
    configure terminal
    interface GigabitEthernet0/0
    switchport trunk native vlan 99
    exit
    ```

    The console errors should cease.

---

### **Part 6: Security Best Practice - The Blackhole VLAN**

The lab stresses that the native VLAN should not be the default VLAN 1. The best practice is to create a dedicated, unused VLAN for the native VLAN on all trunks. This acts as a "blackhole"—any traffic mistakenly sent into it goes nowhere.

1.  **Create the Blackhole VLAN.** On **both switches**, create a new, unused VLAN.

    ```bash
    # On SW1 and SW2
    configure terminal
    vlan 999
    name Blackhole
    exit
    ```

2.  **Assign the Blackhole VLAN as Native.** Apply this configuration to the trunk interface on **both switches**.

    ```bash
    # On SW1 and SW2
    configure terminal
    interface GigabitEthernet0/0
    switchport trunk native vlan 999
    exit
    ```

> #### **Exam-Level Deep Dive: Native VLAN Security**
>
> Why did we choose VLAN 999? We need a VLAN that will *never* have user devices assigned to it and will never have a Switched Virtual Interface (SVI) for routing. By using a dedicated, documented, and unused VLAN as the native VLAN, we enhance security. If an attacker somehow managed to send untagged traffic into our trunk (a technique called VLAN hopping), that traffic would be dumped into the dead-end VLAN 999, effectively neutralizing the attack.

3.  **Final Verification.** On both switches, verify the final state of your trunk.

    ```bash
    # On SW1 and SW2
    show interfaces trunk
    ```

    The native VLAN should now be 999 on both ends, and all console errors should be gone. Connectivity within VLANs 10 and 20 should still be perfect, and isolation between them should be absolute.

### Conclusion & Next Steps

Congratulations. You have successfully segmented a flat network into two distinct, functional VLANs operating across both switches. You configured a secure 802.1Q trunk link, diagnosed and resolved a native VLAN mismatch, and implemented security best practices.

You have now demonstrated complete isolation between the Sales and Engineering departments. While this is secure, it's not yet functional for inter-departmental work. This state of complete separation creates the perfect justification for a subsequent lab on Inter-VLAN Routing.

### **Answer Key**

* **Part 1, Question 2:** Yes, all pings succeeded. In the initial state, all PCs and switch ports were in the same subnet (`10.10.1.0/24`) and the same broadcast domain (VLAN 1), allowing for free communication across the entire Layer 2 fabric.
* **Part 3, Question 4:** The `Administrative Mode` should be `access`, and the `Operational Mode` should also be `access`. This confirms the `switchport mode access` command took effect.
* **Part 4, Question 2:** The trunk status should be `trunking`. The allowed VLANs list should show all active VLANs by default (1, 10, 20). The `show interface G0/0 switchport` command will also show the `Administrative Mode` as `trunk`.
