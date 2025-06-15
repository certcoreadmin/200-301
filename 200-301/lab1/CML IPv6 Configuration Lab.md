# CML IPv6 Configuration Lab

## **Objective**

This lab guide will walk you through the process of configuring basic IPv6 connectivity on your existing CML topology. You will first verify the initial IPv4 setup, then enable IPv6 routing, configure IPv6 addresses on routers, and finally verify that the end devices (all Alpine Linux) have automatically received IPv6 addresses using Stateless Address Autoconfiguration (SLAAC).

## **Lab Topology**

The lab uses the following topology:

* **Routers:** R-1, R-2, R-3
* **Switches:** SW-1, SW-2, SW-3
* **End Devices:** PC-1 (Alpine), PC-2 (Alpine), PC-3 (Alpine)

## **IPv4 Addressing Plan**

| Device | Interface | IPv4 Address | Default Gateway |
| :--- | :--- | :--- | :--- |
| R-1 | Gi0/0 | 10.0.1.1/24 | N/A |
| R-1 | Gi0/1 | 10.0.4.1/24 | N/A |
| R-1 | Gi0/2 | 10.0.5.1/24 | N/A |
| R-2 | Gi0/0 | 10.0.2.1/24 | N/A |
| R-2 | Gi0/1 | 10.0.6.1/24 | N/A |
| R-2 | Gi0/2 | 10.0.5.2/24 | N/A |
| R-3 | Gi0/0 | 10.0.3.1/24 | N/A |
| R-3 | Gi0/1 | 10.0.4.2/24 | N/A |
| R-3 | Gi0/2 | 10.0.6.2/24 | N/A |
| PC-1 | eth0 | 10.0.1.10/24 | 10.0.1.1 |
| PC-2 | eth0 | 10.0.2.10/24 | 10.0.2.1 |
| PC-3| eth0 | 10.0.3.10/24 | 10.0.3.1 |

## **IPv6 Addressing Plan**

| Device | Interface | IPv6 Address | Default Gateway |
| :--- | :--- | :--- | :--- |
| R-1 | Gi0/0 | `2001:CAFE:1::1/64` | N/A |
| R-1 | Gi0/1 | `2001:CAFE:4::1/64` | N/A |
| R-1 | Gi0/2 | `2001:CAFE:5::1/64` | N/A |
| R-2 | Gi0/0 | `2001:CAFE:2::1/64` | N/A |
| R-2 | Gi0/1 | `2001:CAFE:6::1/64` | N/A |
| R-2 | Gi0/2 | `2001:CAFE:5::2/64` | N/A |
| R-3 | Gi0/0 | `2001:CAFE:3::1/64` | N/A |
| R-3 | Gi0/1 | `2001:CAFE:4::2/64` | N/A |
| R-3 | Gi0/2 | `2001:CAFE:6::2/64` | N/A |
| PC-1 | eth0 | SLAAC | Link-Local of R-1 Gi0/0 |
| PC-2 | eth0 | SLAAC | Link-Local of R-2 Gi0/0 |
| PC-3| eth0 | SLAAC | Link-Local of R-3 Gi0/0 |

---

### **Part 1: Verify Initial IPv4 Connectivity**

Before configuring IPv6, ensure that the existing IPv4 network is fully operational. The initial configuration for the Linux hosts should have already set their IPv4 address and default gateway.

#### **Step 1: Test connectivity from PC-1 to PC-3**

Log into the console of PC-1 (Alpine Linux) and ping the IPv4 address of PC-3.
```
# From the PC-1 console
ping 10.0.3.10
```
*The ping should be successful, indicating that EIGRP is routing traffic correctly.*

#### **Step 2: Test connectivity from PC-2 to PC-3**

Next, log into the console of PC-2 (Alpine Linux) and ping the IPv4 address of PC-3.
```
# From the PC-2 console
ping 10.0.3.10
```
*This ping should also be successful.*

---

### **Part 2: Configure IPv6 on Routers**

#### **Step 1: Enable IPv6 Routing on R-1**

On a Cisco router, IPv6 routing is disabled by default. Use the `ipv6 unicast-routing` command to enable it. This command allows the router to forward IPv6 packets and send Router Advertisements (RAs) for SLAAC.

```
! On R-1
configure terminal
 ipv6 unicast-routing
```

#### **Step 2: Configure IPv6 Addresses on R-1 Interfaces**

Assign the IPv6 addresses from the addressing table to the interfaces on R-1.

```
! On R-1
interface GigabitEthernet0/0
 ipv6 address 2001:CAFE:1::1/64
 no shutdown
!
interface GigabitEthernet0/1
 ipv6 address 2001:CAFE:4::1/64
 no shutdown
!
interface GigabitEthernet0/2
 ipv6 address 2001:CAFE:5::1/64
 no shutdown
```

#### **Step 3: Enable IPv6 Routing and Configure Addresses on R-2**

Repeat the process for R-2.

```
! On R-2
configure terminal
 ipv6 unicast-routing
!
interface GigabitEthernet0/0
 ipv6 address 2001:CAFE:2::1/64
 no shutdown
!
interface GigabitEthernet0/1
 ipv6 address 2001:CAFE:6::1/64
 no shutdown
!
interface GigabitEthernet0/2
 ipv6 address 2001:CAFE:5::2/64
 no shutdown
```

#### **Step 4: Enable IPv6 Routing and Configure Addresses on R-3**

Finally, configure R-3.

```
! On R-3
configure terminal
 ipv6 unicast-routing
!
interface GigabitEthernet0/0
 ipv6 address 2001:CAFE:3::1/64
 no shutdown
!
interface GigabitEthernet0/1
 ipv6 address 2001:CAFE:4::2/64
 no shutdown
!
interface GigabitEthernet0/2
 ipv6 address 2001:CAFE:6::2/64
 no shutdown
```

#### **Step 5: Verify Direct Connectivity**

From R-1, you should now be able to ping the directly connected IPv6 interfaces of R-2 and R-3.

```
! On R-1
do ping 2001:CAFE:5::2  ! Pinging R-2
do ping 2001:CAFE:4::2  ! Pinging R-3
```
*You should see a 100% success rate for both pings.*

---

### **Part 3: Verify SLAAC on End Devices**

With IPv6 enabled on the routers, they automatically send Router Advertisement (RA) messages. The Linux hosts will listen for these RAs and use them to configure their own IPv6 addresses and default gateways.

#### **Step 1: Verify IPv6 Configuration on PC-1 (Alpine Linux)**

Log into the console of PC-1 and verify that it has received an IPv6 address and a default route via SLAAC.

```
# From the PC-1 console
ip -6 addr show
ip -6 route show
```
*You should see a global unicast address starting with `2001:CAFE:1::/64` and a default route (`default via fe80...`) pointing to the link-local address of R-1's Gi0/0 interface.*

#### **Step 2: Verify Connectivity from PC-1**

From the PC-1 console, ping the IPv6 address of its gateway (R-1).

```
# From the PC-1 console
ping 2001:CAFE:1::1
```
*This ping should be successful.*

#### **Step 3: Verify IPv6 Configuration on PC-3 (Alpine Linux)**

Log into the console of PC-3 and verify that it has received an IPv6 address and a default route via SLAAC.

```
# From the PC-3 console
ip -6 addr show
ip -6 route show
```
*In the output, you should now see a global unicast address starting with `2001:CAFE:3::/64` and a default route (`default via fe80...`) pointing to the link-local address of R-3's Gi0/0 interface.*

#### **Step 4: Verify Connectivity from PC-3**

From the PC-3 console, use the `ping` command to test connectivity to its gateway (R-3).

```
# From the PC-3 console
ping 2001:CAFE:3::1
```
*This ping should be successful.*

You have now configured and verified basic IPv6 connectivity in your lab! You can perform the same verification steps on PC-2 to ensure it is also working correctly.
