# CCNA - Demo Lab: VLAN Configurations

This lab demonstrates how to configure Virtual LANs (VLANs) across multiple switches and how to use a router to enable communication between them. This common configuration is known as a "router-on-a-stick".

## Lab Objectives

* Implement VLAN assignments on two switches.
* Configure an 802.1q trunk link between two switches.
* Configure an 802.1q trunk link between a switch and a router.
* Configure sub-interfaces on the router to act as a gateway for each VLAN.
* Verify end-to-end connectivity between hosts on different VLANs.

---

## Network Topology

![Network Topology](Screenshot 2025-06-16 at 5.29.44 PM.png)

---

## IP Addressing Table

| Device   | Interface  | IP Address          | VLAN | Notes                        |
| :------- | :--------- | :------------------ | :--- | :--------------------------- |
| R1       | Gi0/0.10   | `192.168.10.1/24`   | 10   | Gateway for VLAN 10          |
| R1       | Gi0/0.20   | `192.168.20.1/24`   | 20   | Gateway for VLAN 20          |
| PC-1     | eth0       | `192.168.10.10/24`  | 10   | User PC                      |
| PC-2     | eth0       | `192.168.10.11/24`  | 10   | User PC (IP corrected in lab) |
| server-1 | eth0       | `192.168.20.2/24`   | 20   | Server                       |
| server-2 | eth0       | `192.168.20.200/24` | 20   | Server                       |
| SW1      | N/A        | N/A                 | 10, 20| Layer 2 Switch               |
| SW2      | N/A        | N/A                 | 10, 20| Layer 2 Switch               |

---

## Step 1: Configure VLANs and Access Ports on SW1

First, we will configure **SW1**. This involves creating the VLANs and then assigning the connected host ports to the appropriate VLAN. PC-1 connects to `GigabitEthernet0/1` and will be in **VLAN 10**. Server-1 connects to `GigabitEthernet0/2` and will be in **VLAN 20**.

Log into SW1 and enter the following commands:

```cisco
! Enter global configuration mode
configure terminal

! Create the VLANs
vlan 10
 name Users
exit

vlan 20
 name Servers
exit

! Configure interface Gi0/1 for PC-1 (VLAN 10)
interface GigabitEthernet0/1
 description Port for PC-1
 switchport mode access
 switchport access vlan 10
exit

! Configure interface Gi0/2 for server-1 (VLAN 20)
interface GigabitEthernet0/2
 description Port for server-1
 switchport mode access
 switchport access vlan 20
exit

! Return to privileged EXEC mode
end
```

> **Note:** `switchport mode access` configures the interface to be on a single VLAN. `switchport access vlan <vlan-id>` assigns the port to that specific VLAN.

#### **Verification**

To verify the configuration on SW1, use the `show vlan brief` command.

```
SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/3
10   Users                            active    Gi0/1
20   Servers                          active    Gi0/2
```

This output confirms that VLANs 10 and 20 are active and that `Gi0/1` is assigned to VLAN 10 and `Gi0/2` is assigned to VLAN 20.

---

## Step 2: Configure VLANs and Access Ports on SW2

Next, we will perform a similar configuration on **SW2**. PC-2 connects to `GigabitEthernet0/1` (VLAN 10) and server-2 connects to `GigabitEthernet0/0` (VLAN 20).

Log into SW2 and enter the following commands:

```cisco
! Enter global configuration mode
configure terminal

! Create the VLANs
vlan 10
 name Users
exit

vlan 20
 name Servers
exit

! Configure interface Gi0/1 for PC-2 (VLAN 10)
interface GigabitEthernet0/1
 description Port for PC-2
 switchport mode access
 switchport access vlan 10
exit

! Configure interface Gi0/0 for server-2 (VLAN 20)
interface GigabitEthernet0/0
 description Port for server-2
 switchport mode access
 switchport access vlan 20
exit

! Return to privileged EXEC mode
end
```

#### **Verification**

To verify the configuration on SW2, use the `show vlan brief` command.

```
SW2# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3
10   Users                            active    Gi0/1
20   Servers                          active    Gi0/0
```

This output confirms that VLANs 10 and 20 are active and that the correct ports are assigned to them.

---

## Step 3: Configure Trunk Links on Switches

To allow traffic from multiple VLANs to travel between devices, we must configure the links between the switches (`SW1 <-> SW2` on `Gi0/3`) and between SW1 and R1 (`SW1 <-> R1` on `Gi0/0`) as "trunks".

**On SW1:**

```cisco
! Enter global configuration mode
configure terminal

! Configure the link to R1 as a trunk
interface GigabitEthernet0/0
 description Trunk to R1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

! Configure the link to SW2 as a trunk
interface GigabitEthernet0/3
 description Trunk to SW2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

end
```

**On SW2:**

```cisco
! Enter global configuration mode
configure terminal

! Configure the link to SW1 as a trunk
interface GigabitEthernet0/3
 description Trunk to SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

end
```

> **Note:** `switchport mode trunk` enables the interface to carry traffic for multiple VLANs. The `switchport trunk encapsulation dot1q` command is required on some switch models to specify the trunking protocol.

#### **Verification**

Verify the trunk configuration using the `show interfaces trunk` command on both switches.

**On SW1:**

```
SW1# show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/3       1-4094
...
```

This confirms that both `Gi0/0` and `Gi0/3` are in trunking mode.

**On SW2:**

```
SW2# show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/3       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/3       1-4094
...
```

This confirms that `Gi0/3` on SW2 is trunking correctly with SW1.

---

## Step 4: Configure Inter-VLAN Routing on R1

We will now configure **R1** to route traffic between VLAN 10 and VLAN 20. This is achieved by creating a sub-interface for each VLAN on the physical interface connected to the switch (`GigabitEthernet0/0`).

> **ALERT:** The physical interface must be enabled with `no shutdown` for the sub-interfaces to function.

Log into R1 and enter the following commands:

```cisco
! Enter global configuration mode
configure terminal

! Enable the physical interface
interface GigabitEthernet0/0
 no shutdown
exit

! Create a sub-interface for VLAN 10
interface GigabitEthernet0/0.10
 description Gateway for VLAN 10 (Users)
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

! Create a sub-interface for VLAN 20
interface GigabitEthernet0/0.20
 description Gateway for VLAN 20 (Servers)
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

end
```

> **Note:** This configuration is called "Router-on-a-Stick". The `encapsulation dot1Q <vlan-id>` command tells the router which VLAN the sub-interface is for. The IP address assigned to the sub-interface becomes the default gateway for all devices on that VLAN.

#### **Verification**

Verify that the sub-interfaces are up and have the correct IP addresses.

```
R1# show ip interface brief

Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES manual up                    up
GigabitEthernet0/0.10      192.168.10.1    YES manual up                    up
GigabitEthernet0/0.20      192.168.20.1    YES manual up                    up
...
```

You can also check the router's routing table to ensure it has connected routes for the two VLANs.

```
R1# show ip route

...
C        192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L        192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
C        192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L        192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

---

## Step 5: Correct IP Address on PC-2

The default lab configuration assigns a duplicate IP address to PC-1 and PC-2 (`192.168.10.10`). To ensure connectivity, we must change the IP address on **PC-2**.

On PC-2, open a terminal and run the following commands:

```bash
# Delete the incorrect IP address
sudo ip addr del 192.168.10.10/24 dev eth0

# Add a unique IP address
sudo ip addr add 192.168.10.11/24 dev eth0
```

> **ALERT:** The original configuration for PC-2 uses the IP address `192.168.10.10`, which is already in use by PC-1. This step is critical to prevent an IP address conflict.

#### **Verification**

On PC-2, verify the IP address has changed.

```bash
# On PC-2
ip addr show dev eth0
```

The output should show `inet 192.168.10.11/24`.

---

## Step 6: Verify End-to-End Connectivity

Finally, we will verify that the entire network is correctly configured by pinging between devices. Note that PC-1's IP is `192.168.10.10` and server-2's IP is now `192.168.20.200`.

**1. Ping from PC-1 to its gateway (R1):**

```bash
# On PC-1
ping 192.168.10.1
```

The ping should be successful, confirming VLAN 10 devices can reach their gateway.

**2. Ping from PC-1 to PC-2 (same VLAN):**

```bash
# On PC-1
ping 192.168.10.11
```

The ping should be successful, demonstrating communication within VLAN 10 across both switches.

**3. Ping from server-1 to server-2 (same VLAN):**

```bash
# On server-1
ping 192.168.20.200
```

The ping should be successful, demonstrating communication within VLAN 20.

**4. Ping from PC-1 (VLAN 10) to server-1 (VLAN 20):**

```bash
# On PC-1
ping 192.168.20.2
```

**5. Ping from PC-1 (VLAN 10) to server-2 (VLAN 20):**

```bash
# On PC-1
ping 192.168.20.200
```

The pings in tests 4 and 5 should be successful. This is crucial as it confirms that R1 is correctly routing traffic between the two VLANs. The first ping might time out while ARP resolves.

---

## Lab Questions

Test your knowledge of the concepts covered in this lab.

### True/False

1.  **True or False:** The command `switchport mode access` is used to allow a port to carry traffic for multiple VLANs.
2.  **True or False:** On router R1, the physical interface `GigabitEthernet0/0` must be assigned an IP address for inter-VLAN routing to work.
3.  **True or False:** A ping from PC-1 to server-2 requires a router to forward the traffic between VLANs.

### Multiple Choice

1.  Which command correctly assigns a switch interface to the "Servers" VLAN?
    * A. `switchport trunk vlan 20`
    * B. `switchport native vlan 20`
    * C. `switchport access vlan 20`
    * D. `interface vlan 20`

2.  What is the IP address of the default gateway for PC-1?
    * A. `192.168.10.10`
    * B. `192.168.20.1`
    * C. `192.168.10.1`
    * D. `192.168.20.2`

3.  To verify that an interface is configured as a trunk, which command should you use?
    * A. `show vlan brief`
    * B. `show ip interface brief`
    * C. `show ip route`
    * D. `show interfaces trunk`

4.  On router R1, what is the purpose of the `encapsulation dot1Q 10` command?
    * A. To assign the IP address for VLAN 10
    * B. To associate the sub-interface with traffic tagged for VLAN 10
    * C. To enable the physical interface
    * D. To create VLAN 10 on the router

---

## Answer Key

### True/False Answers

1.  **False.** The `switchport mode trunk` command allows a port to carry traffic for multiple VLANs. `switchport mode access` restricts a port to a single VLAN.
2.  **False.** The physical interface should not have an IP address. Instead, the sub-interfaces are configured with the IP addresses that serve as default gateways for each VLAN.
3.  **True.** PC-1 is in VLAN 10 and server-2 is in VLAN 20. Communication between different VLANs requires a Layer 3 device, like a router, to route the packets.

### Multiple Choice Answers

1.  **C. `switchport access vlan 20`**. This command is used on an access port to assign it to a specific VLAN.
2.  **C. `192.168.10.1`**. This is the IP address configured on router R1's `GigabitEthernet0/0.10` sub-interface, which serves as the gateway for all devices in VLAN 10.
3.  **D. `show interfaces trunk`**. This command displays the status of all trunking interfaces on a switch, including the encapsulation type and the VLANs allowed.
4.  **B. To associate the sub-interface with traffic tagged for VLAN 10**. This command tells the router's sub-interface to process any Ethernet frames that arrive on the trunk with an `802.1q` tag for VLAN 10.

