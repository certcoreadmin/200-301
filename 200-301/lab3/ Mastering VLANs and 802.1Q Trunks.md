# CCNA Lab: Mastering VLANs, Trunks, and Troubleshooting

[cite_start]**Lab ID:** vlan-trunk-001 
[cite_start]**Difficulty:** Level 2 (Foundational) 
[cite_start]**Estimated Time:** 40 Minutes 
[cite_start]**Core Topics:** Switching, VLANs, 802.1Q Trunking, Troubleshooting 

## Introduction: The Isolation Mandate

Welcome, network engineer. You've been assigned to a client with a small but growing network. [cite_start]Currently, their network is flat—a single Layer 2 domain where all devices, from Sales to Engineering, reside in the same broadcast domain (VLAN 1). [cite: 5, 12] [cite_start]This has served them well, but a new security mandate requires the complete network isolation of the Engineering department. [cite: 6] [cite_start]Your task is to implement this segmentation using VLANs and an 802.1Q trunk link. [cite: 8] [cite_start]Along the way, you'll see why simple IP address changes are insufficient [cite: 7][cite_start], and you'll intentionally create and resolve a common trunking misconfiguration, sharpening your troubleshooting skills. 

## Network Diagram

![Network Diagram](network_diagram.png)

## IP Addressing & VLAN Schema (Final State)

| Device | Interface Assignment | Final IP Address | Subnet Mask | Department | VLAN |
| :--- | :--- | :--- | :--- | :--- | :--- |
| PC1 | SW1 -> Gi0/1 | `10.10.1.11` | `255.255.255.0` | Sales | 1 |
| PC2 | SW1 -> Gi0/2 | `10.10.2.12` | `255.255.255.0` | Engineering | 2 |
| PC3 | SW2 -> Gi0/1 | `10.10.1.13` | `255.255.255.0` | Sales | 1 |
| PC4 | SW2 -> Gi0/2 | `10.10.2.14` | `255.255.255.0` | Engineering | 2 |
| SW1/SW2 | Gi0/0 | Trunk Port | N/A | Backbone | Native 256 |

---

### **Module 1: Baseline Network Testing**

In this module, you will verify the initial state of the network. [cite_start]All devices are in VLAN 1 and the `10.10.1.0/24` subnet, so full communication should be possible. 

1.  **Start all devices in your CML lab.** Open a console to PC1, PC2, PC3, and PC4.

2.  **Verify PC1's Connectivity:** From the console of PC1, ping every other PC to confirm full reachability.

    ```bash
    PC1# ping 10.10.1.12
    PING 10.10.1.12 (10.10.1.12): 56 data bytes
    64 bytes from 10.10.1.12: seq=0 ttl=64 time=1.192 ms
    ...
    PC1# ping 10.10.1.13
    PING 10.10.1.13 (10.10.1.13): 56 data bytes
    64 bytes from 10.10.1.13: seq=0 ttl=64 time=1.872 ms
    ...
    PC1# ping 10.10.1.14
    PING 10.10.1.14 (10.10.1.14): 56 data bytes
    64 bytes from 10.10.1.14: seq=0 ttl=64 time=2.134 ms
    ...
    ```

3.  **Examine the Switch MAC Address Table:** Open a console to SW1. View the MAC address table. You should see the MAC addresses of PC1 and PC2 on their local interfaces, and the MACs of PC3 and PC4 learned via the Gi0/0 link from SW2.

    ```
    SW1# show mac address-table
    Mac Address Table
    -------------------------------------------
    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
       1    [MAC_of_PC1]      DYNAMIC     Gi0/1
       1    [MAC_of_PC2]      DYNAMIC     Gi0/2
       1    [MAC_of_PC3]      DYNAMIC     Gi0/0
       1    [MAC_of_PC4]      DYNAMIC     Gi0/0
    Total Mac Addresses for this criterion: 4
    ```

> **Verification Question 1:** Why are the MAC addresses for PC3 and PC4 both learned on interface Gi0/0 of SW1?

---

### **Module 2: The Flawed Approach (IP-Only Segmentation)**

The first request is to isolate the Engineering PCs (PC2, PC4). [cite_start]A junior engineer attempts this by simply changing their IP addresses to a new subnet, `10.10.2.0/24`.  Let's execute this plan and observe the failure.

1.  **Re-IP the Engineering PCs:**
    * On **PC2**, run the following commands:
        ```bash
        PC2# sudo ip addr del 10.10.1.12/24 dev eth0
        PC2# sudo ip addr add 10.10.2.12/24 dev eth0
        ```
    * On **PC4**, run the following commands:
        ```bash
        PC4# sudo ip addr del 10.10.1.14/24 dev eth0
        PC4# sudo ip addr add 10.10.2.14/24 dev eth0
        ```

2.  **Test Connectivity:**
    * From **PC2**, try to ping its Engineering counterpart, **PC4**, at `10.10.2.14`.
        ```bash
        PC2# ping 10.10.2.14
        PING 10.10.2.14 (10.10.2.14): 56 data bytes
        ^C
        --- 10.10.2.14 ping statistics ---
        5 packets transmitted, 0 packets received, 100% packet loss
        ```
    * From **PC2**, try to ping a Sales PC, **PC1**, at `10.10.1.11`. This will also fail.

> **Verification Question 2:** The ping from PC2 to PC4 failed. Both are on the same IP subnet (`10.10.2.0/24`) and the switches between them are functional. Why can't they communicate? (Hint: Think about Layer 2 vs. Layer 3 and broadcast domains).

---

### **Module 3: Proper Segmentation (VLAN Implementation)**

Now, let's implement the correct solution. We will create a new VLAN for the Engineering department and assign the correct switch ports to it.

1.  **Create the Engineering VLAN:** Create VLAN 2 on both switches.
    * On **SW1** and **SW2**:
        ```
        SW1# conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        SW1(config)# vlan 2
        SW1(config-vlan)# name Engineering
        SW1(config-vlan)# exit
        ```

2.  **Verify VLAN Creation:** Use a variety of `show` commands to see the new VLAN from different perspectives. This is a key skill.
    ```
    SW1(config)# do show vlan brief

    VLAN  Name                             Status    Ports
    ----  -------------------------------- --------- --------------------------------
    1     default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                     Gi1/0, Gi1/1, Gi1/2, Gi1/3
    2     Engineering                      active    
    ...

    SW1(config)# do show vlan name Engineering

    VLAN  Name                             Status    Ports
    ----  -------------------------------- --------- --------------------------------
    2     Engineering                      active
    ```

3.  **Assign Access Ports to VLAN 2:** On each switch, assign the port connected to the Engineering PC to VLAN 2.
    * On **SW1**:
        ```
        SW1(config)# interface gigabitEthernet 0/2
        SW1(config-if)# switchport mode access
        SW1(config-if)# switchport access vlan 2
        ```
    * On **SW2**:
        ```
        SW2(config)# interface gigabitEthernet 0/2
        SW2(config-if)# switchport mode access
        SW2(config-if)# switchport access vlan 2
        ```

4.  **Verify Port Assignment:** Confirm the ports are now in VLAN 2.
    * On **SW1**:
        ```
        SW1(config-if)# do show vlan brief

        VLAN  Name                             Status    Ports
        ----  -------------------------------- --------- --------------------------------
        1     default                          active    Gi0/0, Gi0/1, Gi0/3, Gi1/0
                                                         Gi1/1, Gi1/2, Gi1/3
        2     Engineering                      active    Gi0/2

        SW1(config-if)# do show interface gi0/2 switchport
        Name: Gi0/2
        Switchport: Enabled
        Administrative Mode: static access
        Operational Mode: static access
        ...
        Access Mode VLAN: 2 (Engineering)
        ...
        ```
    * Run similar verification commands on **SW2**.

> At this point, try pinging from PC2 to PC4 again. It will still fail. Why? Because the switches have no way to pass VLAN 2 traffic between them. The Gi0/0 link is still an access port in VLAN 1. We need a trunk.

---

### **Module 4: Trunk Configuration & The "Aha!" Moment**

This is the core of the lab. [cite_start]You will configure the trunk link, but with a deliberate native VLAN mismatch to witness and diagnose the resulting error. 

1.  **Configure Trunk on SW1:**
    ```
    SW1(config)# interface gigabitEthernet 0/0
    SW1(config-if)# switchport trunk encapsulation dot1q
    SW1(config-if)# switchport mode trunk
    SW1(config-if)# switchport trunk native vlan 256
    SW1(config-if)# switchport nonegotiate
    ```

> ### **Exam-Level Deep Dive: DTP and Trunk Security**
> * `switchport trunk encapsulation dot1q`: On older switches, you had to choose between the proprietary Cisco ISL and the industry-standard IEEE 802.1Q. Modern switches only use 802.1Q, but the command is still part of the syntax. [cite_start]We use it to be explicit. 
> * `switchport nonegotiate`: This command disables the Dynamic Trunking Protocol (DTP). DTP's job is to automatically negotiate a trunk link. While convenient, it's a security risk. An attacker could plug a device into a port, send DTP packets, and form a trunk, giving them access to all VLANs. Disabling DTP and manually configuring trunks (`switchport mode trunk`) is a critical security best practice. 
> * `switchport trunk native vlan 256`: The native VLAN is the one VLAN whose traffic crosses a trunk link *without* an 802.1Q tag. For security, this should never be VLAN 1 or a user data VLAN. [cite_start]We choose an unused, dedicated VLAN.  The number 256 is chosen because it's memorable—it's an invalid number in an IPv4 octet, making it easy to remember as "not for hosts."

2.  **Configure Trunk on SW2 (Intentionally Incorrect):**
    ```
    SW2(config)# interface gigabitEthernet 0/0
    SW2(config-if)# switchport trunk encapsulation dot1q
    SW2(config-if)# switchport mode trunk
    SW2(config-if)# no switchport trunk native vlan
    SW2(config-if)# switchport nonegotiate
    ```
    *By not setting the native VLAN, we are leaving it at the default of VLAN 1.*

3.  **Detect the Mismatch:** Wait about 30-60 seconds. Cisco Discovery Protocol (CDP) runs between the switches and will detect this problem. Check the logs on **both** switches.
    ```
    SW1#
    *Jun 19 20:15:00.123: %CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/0 (256), with SW2 GigabitEthernet0/0 (1).
    ```

4.  **Verify with `show` commands:** The `show interfaces trunk` command also reveals the issue, although less explicitly than the log message.
    ```
    SW1# show interfaces trunk
    Port        Mode         Encapsulation  Status        Native vlan
    Gi0/0       on           802.1q         trunking      256

    Port        Vlans allowed on trunk
    Gi0/0       1-4094
    ...
    ```

> **Verification Question 3:** Besides the log message, what other command could you use with a `neighbor` keyword to get detailed information about SW2, including its native VLAN setting?

---

### **Module 5: Resolution and Final Verification**

Let's fix the native VLAN mismatch and complete the lab by verifying full, but segmented, connectivity.

1.  **Correct the Native VLAN on SW2:**
    ```
    SW2# conf t
    SW2(config)# interface gigabitEthernet 0/0
    SW2(config-if)# switchport trunk native vlan 256
    ```

2.  **Verify Trunk Status:** The trunk should now be consistent across both switches.
    ```
    SW2(config-if)# do show interfaces trunk
    Port        Mode         Encapsulation  Status        Native vlan
    Gi0/0       on           802.1q         trunking      256
    ...
    ```

3.  **Final Connectivity Test:**
    * From **PC2**, ping **PC4**. The ping should now succeed!
        ```bash
        PC2# ping 10.10.2.14
        PING 10.10.2.14 (10.10.2.14): 56 data bytes
        64 bytes from 10.10.2.14: seq=0 ttl=64 time=2.543 ms
        ```
    * From **PC1**, ping **PC3**. This should also succeed as they are both in VLAN 1.
        ```bash
        PC1# ping 10.10.1.13
        PING 10.10.1.13 (10.10.1.13): 56 data bytes
        64 bytes from 10.10.1.13: seq=0 ttl=64 time=1.998 ms
        ```
    * From **PC2** (VLAN 2), attempt to ping **PC1** (VLAN 1). This ping **MUST FAIL**.
        ```bash
        PC2# ping 10.10.1.11
        PING 10.10.1.11 (10.10.1.11): 56 data bytes
        ^C
        --- 10.10.1.11 ping statistics ---
        5 packets transmitted, 0 packets received, 100% packet loss
        ```

Congratulations! You have successfully segmented the network. [cite_start]Hosts within the same VLAN can communicate across switches, but inter-VLAN communication is blocked, as intended.  [cite_start]This sets the stage perfectly for a future lab on Inter-VLAN routing. 

## Summary of Skills Learned

* [cite_start]**VLAN Configuration:** Created VLANs and assigned names. 
* [cite_start]**Access Port Assignment:** Assigned switch ports to specific VLANs. 
* [cite_start]**802.1Q Trunking:** Manually configured a trunk link with best practices. 
* [cite_start]**Troubleshooting:** Diagnosed and resolved a native VLAN mismatch using CDP logs and verification commands. 
* [cite_start]**Security Best Practices:** Disabled DTP and utilized a dedicated native VLAN for enhanced security. 

## Answer Key

1.  **Question 1:** Because SW1 and SW2 are connected by a simple Layer 2 link (in VLAN 1 by default), SW1 learns the MAC addresses of any devices connected to SW2 through that single connecting port, Gi0/0.
2.  **Question 2:** Even though the PCs are on the same IP subnet, they are still in the same Layer 2 broadcast domain (VLAN 1). When PC2 sends an ARP request for PC4's MAC address, that broadcast is heard by PC1 and PC3, who are on a different IP subnet and will ignore it. The switches happily forward the broadcast across the domain, but no device on the `10.10.2.0/24` network can reply to the ARP from the `10.10.1.0/24` devices, and vice-versa. [cite_start]Proper VLANs are needed to create separate L2 broadcast domains. 
3.  **Question 3:** The command `show cdp neighbors detail` would provide extensive information about SW2 from the perspective of SW1, including its IP address, platform, and the native VLAN configured on its end of the link.