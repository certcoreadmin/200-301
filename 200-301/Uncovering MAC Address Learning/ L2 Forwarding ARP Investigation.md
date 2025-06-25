# CCNA Lab Guide: L2 Forwarding & ARP Investigation

## 1. Introduction & Lab Scenario

Welcome to a more advanced Layer 2 investigation. As a network administrator, you've just connected workstations for three different departments—Administration (`PC1-Admin`), Sales (`PC2-Sales`), and Engineering (`PC3-Eng`)—to a new switch.

A critical principle in networking is understanding the relationship between Layer 2 (MAC addresses) and Layer 3 (IP addresses). A switch uses MAC addresses to forward frames, but hosts use IP addresses to communicate. The **Address Resolution Protocol (ARP)** is the glue that binds these two layers together.

**Your Mission:** Your goal is to not only observe how the switch learns MAC addresses but to correlate this behavior directly with the ARP tables on the end-hosts. You will prove connectivity from Admin to Engineering and analyze the "before and after" state on hosts and the switch to build a complete picture of the forwarding process.

---

## 2. Network Diagram & IP Addressing

Review the updated topology with our three department workstations.

```
      +----------------------+
      |         SW1          |
      |   (IOSvL2 Switch)    |
      +----------------------+
      | Gi0/0  Gi0/1  Gi0/2  |
      +--+------+------+---+
         |      |      |
         |      |      |
+--------+--+ +--+---+ +--+--------+
|PC1-Admin | |PC2-Sales| | PC3-Eng  |
+----------+ +---------+ +-----------+
192.168.1.10  192.168.1.11  192.168.1.12
```

### IP Addressing Table

| Device      | Interface | IP Address      | Subnet Mask   | Department    |
|-------------|-----------|-----------------|---------------|---------------|
| `PC1-Admin` | `eth0`    | `192.168.1.10`  | `255.255.255.0` | Administration|
| `PC2-Sales` | `eth0`    | `192.168.1.11`  | `255.255.255.0` | Sales         |
| `PC3-Eng`   | `eth0`    | `192.168.1.12`  | `255.255.255.0` | Engineering   |
| `SW1`       | `VLAN 1`  | (Unassigned)    | (N/A)         | L2 Forwarding |

---

## 3. Step-by-Step Lab Procedure

### Part 1: Initial State Analysis - The "Before" Snapshot

A proficient engineer always captures the baseline state before initiating any action.

**Step 1: Host-Level Investigation (PC1-Admin)**

On `PC1-Admin`, we will check its Layer 3 configuration and its Layer 2 ARP table.

* **Check IP Configuration:** The `ip addr` command is the Linux equivalent of `ipconfig`.
    ```
    PC1-Admin# ip addr show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 00:be:75:e6:73:00 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
           valid_lft forever preferred_lft forever
    ```
* **Check ARP Table:** The `ip neigh show` command displays the ARP table (neighbor cache).
    ```
    PC1-Admin# ip neigh show
    ```
    **Expected Outcome:** The command will produce no output. The ARP table is empty because `PC1-Admin` hasn't tried to communicate with anyone yet.

**Step 2: Switch-Level Investigation (SW1)**

Now, let's examine `SW1` to confirm it is also a "blank slate."

* **Check Interface Status:** This command gives a quick summary of all interfaces.
    ```
    SW1# show interfaces status
    
    Port      Name               Status       Vlan       Duplex  Speed Type
    Gi0/0     Link PC1 -> SW1    connected    1          auto    auto  10/100/1000BaseTX
    Gi0/1     Link PC2 -> SW1    connected    1          auto    auto  10/100/1000BaseTX
    Gi0/2     Link PC3 -> SW1    connected    1          auto    auto  10/100/1000BaseTX
    Gi0/3                        notconnect   1          auto    auto  10/100/1000BaseTX
    ```
    This confirms our devices are physically connected and the links are up.

* **Check MAC Address Table:**
    ```
    SW1# show mac address-table
              Mac Address Table
    -------------------------------------------
    
    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    
    Total Mac Addresses for this criterion: 0
    ```
    As expected, the switch's forwarding table is empty.

### Part 2: The Catalyst - Admin Pings Engineering

Let's trigger the network learning process.

**Step 3: Initiate a Ping from PC1-Admin to PC3-Eng**

From `PC1-Admin`, send a single ping to the Engineering workstation, `PC3-Eng`.

```
PC1-Admin# ping -c 1 192.168.1.12

PING 192.168.1.12 (192.168.1.12) 56(84) bytes of data.
64 bytes from 192.168.1.12: icmp_seq=1 ttl=64 time=1.58 ms

--- 192.168.1.12 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.580/1.580/1.580/0.000 ms
```

### Part 3: The "After" Snapshot - Correlating the Evidence

The ping was successful. Now, let's conduct our investigation to see what changed on both the switch and the hosts.

**Step 4: Re-Examine the Switch (SW1)**

* **Check the MAC Table:**
    ```
    SW1# show mac address-table
              Mac Address Table
    -------------------------------------------
    
    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    
       1    00be.75e6.7300    DYNAMIC     Gi0/0
       1    00be.75e6.7302    DYNAMIC     Gi0/2
    
    Total Mac Addresses for this criterion: 2
    ```
    **Analysis:** The switch has now learned the MAC addresses for `PC1-Admin` (source of the ping) and `PC3-Eng` (source of the ping reply) and mapped them to the correct ports.

* **Check Interface Details:** Let's zoom into `Gi0/0` to see traffic counters.
    ```
    SW1# show interfaces GigabitEthernet0/0
    GigabitEthernet0/0 is up, line protocol is up (connected)
      ... (output truncated) ...
      5 minute input rate 0 bits/sec, 0 packets/sec
      5 minute output rate 0 bits/sec, 0 packets/sec
         1 packets input, 84 bytes, 0 no buffer
         ...
         1 packets output, 84 bytes, 0 underruns
      ... (output truncated) ...
    ```
    **Analysis:** We can see counters for `packets input` (the ARP Reply and/or ICMP Echo Reply from PC3) and `packets output` (the initial ARP Request and/or ICMP Echo Request from PC1). This confirms traffic flowed through this specific port.

**Step 5: Re-Examine the Host (PC1-Admin)**

This is the final piece of the puzzle. Let's see what `PC1-Admin` learned.

* **Check ARP Table:**
    ```
    PC1-Admin# ip neigh show
    192.168.1.12 dev eth0 lladdr 00:be:75:e6:73:02 REACHABLE
    ```
    **Analysis:** Success! `PC1-Admin` now has a permanent entry mapping the IP address `192.168.1.12` to the MAC address (`lladdr`) of `PC3-Eng`. It no longer needs to broadcast an ARP request to talk to the Engineering PC.

> **Exam-Level Deep Dive: MAC Table vs. ARP Table**
> It is CRITICAL to understand the difference between these two tables:
> * **MAC Address Table (on Switches):** Maps a **MAC Address** to a **Switch Port**. It answers the question, "To which physical interface should I send a frame destined for MAC address X?" It is purely a Layer 2 concept.
> * **ARP Table (on Hosts/Routers):** Maps a **Layer 3 IP Address** to a **Layer 2 MAC Address**. It answers the question, "What is the hardware MAC address for the device that owns IP address Y?" It is the bridge between L3 and L2.

**Verification Question:** You have successfully pinged from Admin to Engineering. If you now go to `PC2-Sales` (192.168.1.11) and check its ARP table (`ip neigh show`), what do you expect to see? Why?

---

## 4. Lab Conclusion

By methodically inspecting the state of the network before and after a single `ping`, you have observed the intricate dance between hosts and switches. You've gone beyond simply verifying MAC learning and have now correlated it with the host-side ARP process.

**Skills Validated:**
* **Comprehensive Verification:** You used `ip addr`, `ip neigh show`, `show mac address-table`, `show interfaces status`, and `show interfaces <if>` to build a complete operational picture.
* **Correlation Analysis:** You successfully linked the creation of an ARP entry on a host to the population of the MAC address table on a switch.
* **Understanding L2/L3 Interaction:** You have demonstrated a practical understanding of how ARP enables Layer 3 communication over a Layer 2 medium.

---

## 5. Answer Key

**Verification Question Answer:** You would expect the ARP table on `PC2-Sales` to be **empty**. `PC2-Sales` was not a part of the communication between Admin and Engineering. It never sent or received any frames related to that exchange, so it had no opportunity to learn an ARP entry, and no device had a reason to learn its MAC address.
