# CCNA Lab: Router Basics (Comprehensive)

## Objective

In this lab, you will learn the basic operation of a router and its fundamental role in forwarding packets between computer networks. Routers create separate broadcast domains, and by default, they will only forward packets for networks they have a specific route to. You will explore these concepts, discover why inter-network communication fails, and then implement the solution to achieve full connectivity.

## Command Summary

| Command                 | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `ping <ip-address>`     | Sends an ICMP Echo Request to test Layer 3 reachability.     |
| `show ip interface brief` | Displays the status and IP address of a router's interfaces. |
| `show ip route`         | Displays the contents of the IP routing table.               |
| `ip route <network> <mask> <next-hop>` | Creates a static route to a destination network. |
| `ip addr show`          | (On Alpine Linux) Displays interface IP address information.   |

---

## Network Diagram

![Comprehensive Network Diagram](https://i.imgur.com/k2j1S9p.png)

---

## IP Addressing Table

| Device | Interface | IP Address / Subnet | Default Gateway |
| :----- | :-------- | :------------------ | :-------------- |
| **R1** | G0/0      | `10.0.12.1 /30`     | N/A             |
|        | G0/1      | `192.168.1.1 /24`   | N/A             |
| **R2** | G0/0      | `10.0.12.2 /30`     | N/A             |
|        | G0/1      | `192.168.2.1 /24`   | N/A             |
| **PC1** | eth0      | `192.168.1.10 /24`  | `192.168.1.1`   |
| **PC2** | eth0      | `192.168.2.10 /24`  | `192.168.2.1`   |

---

## Lab Procedure

### Part 1: Verify Local Connectivity

A good network engineer always confirms what *is* working before troubleshooting what isn't.

**Step 1: Access PC1 and Verify its Gateway**
- Open the console for `PC1` and ping its default gateway, which is the `G0/1` interface of `R1`.

```sh
PC1:~$ ping -c 4 192.168.1.1
PING 192.168.1.1 (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: seq=0 ttl=64 time=1.843 ms
...
4 packets transmitted, 4 packets received, 0% packet loss
```

### Part 2: Discover the Communication Barrier

**Step 2: Attempt to Ping PC2 from PC1**
- From `PC1`, attempt to ping the IP address of `PC2`.

```sh
PC1:~$ ping -c 4 192.168.2.10
PING 192.168.2.10 (192.168.2.10): 56 data bytes
...
4 packets transmitted, 0 packets received, 100% packet loss
```
**The pings fail.** Our mission is to find out why and then fix it.

### Part 3: The "Aha!" Moment - Investigating the Routers

**Step 3: Check R1's Interfaces and Routing Table**
- Open the console for `R1` and use `show ip route`.

```sh
R1# show ip route
...
C        10.0.12.0/30 is directly connected, GigabitEthernet0/0
L        10.0.12.1/32 is directly connected, GigabitEthernet0/0
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
```

> **Verification Question 1:** Look at R1's routing table. Is there any entry for the destination network `192.168.2.0/24`, where PC2 lives?

**Step 4: Verify R2's Perspective**
- Check `R2`'s routing table.

```sh
R2# show ip route
...
C        10.0.12.0/30 is directly connected, GigabitEthernet0/0
L        10.0.12.2/32 is directly connected, GigabitEthernet0/0
C        192.168.2.0/24 is directly connected, GigabitEthernet0/1
L        192.168.2.1/32 is directly connected, GigabitEthernet0/1
```
> **Verification Question 2:** Does R2 have a route to PC1's network (`192.168.1.0/24`)? Why is this a problem?

---

### Part 4: Implementing the Fix with Static Routes

Now that we've diagnosed the problem—missing routes—we will manually add them. This is called creating **static routes**.

**Step 5: Configure a Static Route on R1**
- On `R1`, we need to tell it how to reach PC2's network (`192.168.2.0/24`). The path to get there is through R2's WAN interface (`10.0.12.2`).

```sh
R1# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.12.2
R1(config)# end
```

**Step 6: Configure a Static Route on R2**
- On `R2`, we must do the reverse. We need to tell it how to get back to PC1's network (`192.168.1.0/24`). The path is through R1's WAN interface (`10.0.12.1`).

```sh
R2# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.12.1
R2(config)# end
```

### Part 5: Final Verification - Test End-to-End Access

With the static routes configured, let's verify that our fix worked.

**Step 7: Verify the Routing Tables**
- First, check the routing table on `R1` again.

```sh
R1# show ip route
...
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.12.0/30 is directly connected, GigabitEthernet0/0
L        10.0.12.1/32 is directly connected, GigabitEthernet0/0
S     192.168.2.0/24 [1/0] via 10.0.12.2
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/1
L        192.168.1.1/32 is directly connected, GigabitEthernet0/1
```
You can now see a new route, marked `S` for Static! R1 now knows how to reach `192.168.2.0/24`.

- Now, check the routing table on `R2`. You should see a similar static route for `192.168.1.0/24`.

**Step 8: Perform the Final Test**
- Go back to the console for `PC1` and perform the same ping to `PC2` that failed in Step 2.

```sh
PC1:~$ ping -c 4 192.168.2.10
PING 192.168.2.10 (192.168.2.10): 56 data bytes
64 bytes from 192.168.2.10: seq=0 ttl=62 time=2.543 ms
64 bytes from 192.168.2.10: seq=1 ttl=62 time=2.112 ms
64 bytes from 192.168.2.10: seq=2 ttl=62 time=2.088 ms
64 bytes from 192.168.2.10: seq=3 ttl=62 time=2.321 ms

--- 192.168.2.10 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
```
**Success!** The ping now works because both routers have a complete map of the network, allowing them to forward packets in both directions.

---

## Conclusion

You have successfully diagnosed and resolved a classic routing problem. You've learned that:
1. Routers only know about their directly connected networks by default.
2. If a router lacks a route to a destination, it drops the packet.
3. Using **static routes**, you can manually add entries to a router's table, teaching it how to reach remote networks.
4. Communication requires a valid route in **both directions** to be successful.

---

## Answer Key

1.  **Answer 1:** No. There is no entry for `192.168.2.0/24` in R1's routing table initially. This is why it dropped the packet from PC1.
2.  **Answer 2:** No, R2 does not have a route to `192.168.1.0/24`. This is a problem because even if R1 could forward the initial ping, R2 would not know how to send the reply back to PC1's network. It would drop the reply packet.
