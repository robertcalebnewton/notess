# CCNA Packet Tracer Lab Series

A hands-on practice series for the CCNA (200-301). Each lab gives you a scenario, a topology, and requirements. **No subnetting answers and no commands are included.** You work out the addressing and the configuration yourself.

## How to Use This Series

1. Build the topology exactly as described in Packet Tracer.
2. Do all subnet planning on paper first. Document your plan before touching a device.
3. Configure from memory. If you get stuck, use the `?` help in the CLI or the official docs, not a walkthrough.
4. Work through the **Verification Goals**. If you can't prove a goal is met, it isn't done.
5. Complete the **Break/Fix Challenge** at the end of each lab. Troubleshooting is a big part of the exam.
6. Save each lab as its own `.pkt` file so you can revisit and reuse it.

**Tracking sheet:** copy this and tick labs off as you finish.

| # | Lab | Done | Date | Notes / Weak Spots |
|---|-----|------|------|--------------------|
| 1 | Basic Device Setup and Hardening | ☐ | | |
| 2 | VLANs and Trunking | ☐ | | |
| 3 | Inter-VLAN Routing (Router-on-a-Stick) | ☐ | | |
| 4 | Layer 3 Switch Inter-VLAN Routing | ☐ | | |
| 5 | Spanning Tree Protocol | ☐ | | |
| 6 | EtherChannel | ☐ | | |
| 7 | Static Routing and Default Routes | ☐ | | |
| 8 | Single-Area OSPFv2 | ☐ | | |
| 9 | DHCP and DHCP Relay | ☐ | | |
| 10 | NAT and PAT | ☐ | | |
| 11 | Standard and Extended ACLs | ☐ | | |
| 12 | Switch Security | ☐ | | |
| 13 | First Hop Redundancy (HSRP) | ☐ | | |
| 14 | IPv6 Addressing and Routing | ☐ | | |
| 15 | Network Services (NTP, Syslog, SNMP, CDP/LLDP) | ☐ | | |
| 16 | Wireless LAN with a WLC | ☐ | | |
| 17 | Capstone: Full Enterprise Network | ☐ | | |

---

## Lab 1: Basic Device Setup and Hardening

**Difficulty:** Beginner  
**Topics:** Initial configuration, passwords, banners, remote management, interface basics

### Scenario
You've been handed a brand-new router and two switches for a small office. Prepare them for production.

### Topology
- 1 router
- 2 switches
- 2 PCs (one per switch)
- Router connects to Switch 1, Switch 1 connects to Switch 2

### Requirements
1. Give every device a meaningful hostname following a naming convention you design.
2. Secure privileged access, console access, and remote (VTY) access with passwords. Stored passwords must not be readable in plain text in the configuration.
3. Remote management must use a secure protocol only. Insecure remote access must be rejected.
4. Configure a legal warning banner shown at login.
5. Configure a management IP on each switch so you can reach it from the PCs. You decide the network and how many addresses are needed. The management network must be sized for no more than 30 hosts.
6. Add a description to every connected interface.
7. Disable any interface that is not in use.
8. Save the configuration so it survives a reload.

### Verification Goals
- [ ] You can SSH to both switches from a PC, and a Telnet attempt fails.
- [ ] Passwords do not appear in readable form in the running configuration.
- [ ] The banner displays on login.
- [ ] Unused ports are administratively down.
- [ ] After a reload, the configuration is still present.

### Break/Fix Challenge
Have a friend (or future you) change the default gateway on one switch and break SSH from a remote subnet. Find and fix it.

---

## Lab 2: VLANs and Trunking

**Difficulty:** Beginner  
**Topics:** VLAN creation, access ports, trunks, native VLAN, DTP

### Scenario
A company has three departments that must be separated at Layer 2: Sales, Engineering, and Guest. A fourth VLAN is needed for management.

### Topology
- 2 switches connected by a single link
- 2 PCs per switch (4 total), spread across at least 3 different departments
- No router in this lab

### Requirements
1. Create four VLANs with names and IDs of your choosing (avoid VLAN 1 for user traffic).
2. Assign access ports so each department's PCs land in the correct VLAN. Ensure at least one department has hosts on both switches.
3. Configure the inter-switch link as a trunk that allows only the VLANs you actually use.
4. Change the native VLAN on the trunk to an unused VLAN, and make sure both ends match.
5. Disable dynamic trunk negotiation on all ports.
6. Give each department its own address range. Use one /24 as your starting block and divide it so that Sales needs 50 hosts, Engineering needs 25 hosts, Guest needs 10 hosts, and Management needs 5 hosts. Work out the smallest suitable subnet for each.

### Verification Goals
- [ ] PCs in the same VLAN on different switches can ping each other.
- [ ] PCs in different VLANs cannot ping each other (and you can explain why).
- [ ] The trunk shows only the allowed VLANs and the correct native VLAN.
- [ ] You can list which ports belong to which VLAN.

### Break/Fix Challenge
Mismatch the native VLAN on one end of the trunk. Identify the warning messages and fix it.

---

## Lab 3: Inter-VLAN Routing (Router-on-a-Stick)

**Difficulty:** Beginner–Intermediate  
**Topics:** Subinterfaces, 802.1Q encapsulation, default gateways

### Scenario
Extend Lab 2 so the departments can communicate through a router with a single physical link to the switch.

### Topology
- 1 router, 1 or 2 switches
- 3 PCs, one in each of three VLANs
- A single router-to-switch link

### Requirements
1. Reuse your VLAN design from Lab 2, or build a new one with three user VLANs.
2. Configure the router so each VLAN has its own default gateway on one physical interface.
3. Use a consistent convention for the gateway address in each subnet.
4. Make the switch-to-router link a trunk.
5. Ensure all VLANs can reach each other.

### Verification Goals
- [ ] Every PC can ping its own gateway.
- [ ] Every PC can ping PCs in the other VLANs.
- [ ] A trace from one VLAN to another shows the router as the hop.
- [ ] The routing table lists every VLAN subnet as directly connected.

### Break/Fix Challenge
Put the wrong VLAN ID on one subinterface. Work out which symptoms you see and fix it.

---

## Lab 4: Layer 3 Switch Inter-VLAN Routing

**Difficulty:** Intermediate  
**Topics:** SVIs, routed ports, `ip routing` concepts, multilayer switching

### Scenario
The router-on-a-stick design is a bottleneck. Replace it with a multilayer switch acting as the core.

### Topology
- 1 multilayer switch (core)
- 2 access switches, each uplinked to the core
- 4 PCs across 3 VLANs
- 1 router acting as the "edge" that connects to a server in a separate network

### Requirements
1. Create the VLANs on all switches and place PCs appropriately.
2. Configure the core switch to route between all VLANs.
3. Connect the core to the edge router using a routed (non-switchport) link on its own small point-to-point subnet. Choose the most efficient subnet size.
4. Make the router and core exchange reachability so PCs can reach the server and the server can reach the PCs. Use static routes for now.

### Verification Goals
- [ ] All PCs reach each other across VLANs.
- [ ] All PCs reach the server.
- [ ] The core's routing table includes connected, local, and static entries.
- [ ] You can explain the difference between an SVI and a routed port.

### Break/Fix Challenge
Forget to enable routing on the multilayer switch, then diagnose why SVIs are up but nothing routes.

---

## Lab 5: Spanning Tree Protocol

**Difficulty:** Intermediate  
**Topics:** Root bridge election, port roles/states, PortFast, BPDU Guard, Rapid PVST+

### Scenario
Three switches are connected in a triangle for redundancy. Control which switch is root and protect the network from loops.

### Topology
- 3 switches in a full triangle
- 1 PC on each switch
- All PCs in the same VLAN, plus at least one second VLAN

### Requirements
1. Before changing anything, identify the root bridge and each switch's port roles. Document them.
2. Choose which switch should be primary root and which should be secondary root for each VLAN. Make different switches root for different VLANs to balance load.
3. Switch to a rapid spanning tree mode on all switches.
4. Enable edge-port behavior on PC-facing ports and protect those ports from rogue switches.
5. Influence which link is blocked using a method other than just changing the root.

### Verification Goals
- [ ] Root bridge is what you intended, per VLAN.
- [ ] You can identify every root, designated, and alternate port and explain why.
- [ ] Unplugging the active path causes fast reconvergence.
- [ ] Plugging a switch into a protected port shuts that port down.

### Break/Fix Challenge
Lower the priority on a "bad" access switch and watch it hijack the root role. Fix it and make sure it can't happen again.

---

## Lab 6: EtherChannel

**Difficulty:** Intermediate  
**Topics:** LACP, PAgP, static bundling, load balancing, consistency requirements

### Scenario
The single uplink between two switches is saturated. Bundle multiple links together.

### Topology
- 2 switches connected by 3 parallel links
- PCs on each switch in at least 2 VLANs

### Requirements
1. Bundle two of the links using an open-standard negotiation protocol.
2. Bundle the third link into a second separate EtherChannel using a Cisco-proprietary protocol (use a different pair of switch ports or add a link as needed).
3. Make each bundle a trunk carrying the VLANs you need.
4. Ensure STP treats each bundle as a single link.
5. Pick and justify a load-balancing method.

### Verification Goals
- [ ] The bundles are up and show all member ports bundled.
- [ ] Shutting down one member link doesn't interrupt traffic.
- [ ] You can state what parameters must match on all member ports.

### Break/Fix Challenge
Change the speed or VLAN list on one member port and watch it get suspended. Find which setting is inconsistent and repair it.

---

## Lab 7: Static Routing and Default Routes

**Difficulty:** Intermediate  
**Topics:** Static routes, default routes, floating static routes, summarization

### Scenario
Connect three branch sites and a headquarters with no dynamic routing.

### Topology
- 4 routers in a chain or partial mesh (your choice, but include at least one redundant path)
- 1 PC or server behind each router on its own LAN
- Serial or Ethernet links between routers

### Requirements
1. Starting from a block of your choosing, carve out unique LAN subnets and point-to-point subnets. Size point-to-point links as efficiently as possible. Branch LANs need 60, 28, and 12 hosts respectively; HQ needs 120.
2. Configure static routes so every LAN can reach every other LAN.
3. Use a default route on the branch routers instead of listing every remote network.
4. Summarize at least two contiguous networks into a single route where possible.
5. Configure a backup route that is used only if the primary path fails.

### Verification Goals
- [ ] Every PC can ping every other PC.
- [ ] The routing table distinguishes static, default, and connected routes.
- [ ] Failing the primary link causes the backup route to take over.
- [ ] You can explain administrative distance and why the backup route works.

### Break/Fix Challenge
Use the wrong next-hop address on one route. Trace the packet hop-by-hop to find where it dies.

---

## Lab 8: Single-Area OSPFv2

**Difficulty:** Intermediate  
**Topics:** OSPF neighbors, router ID, DR/BDR, cost, passive interfaces, default route propagation

### Scenario
Replace the static routing in Lab 7 with OSPF.

### Topology
- Reuse Lab 7, or build 4 routers where at least one segment is a multi-access Ethernet segment with 3 or more routers

### Requirements
1. Configure OSPF area 0 on all routers using a router ID you set deliberately (do not rely on automatic selection).
2. Advertise all LANs and links.
3. Prevent OSPF hellos from being sent out LAN-facing interfaces.
4. On the multi-access segment, control which router becomes the DR.
5. Change the cost on one link so that traffic prefers a different path.
6. Have the edge router advertise a default route to the rest of the network.
7. Tune the reference bandwidth so that fast links are differentiated properly.

### Verification Goals
- [ ] All routers form full adjacencies with the expected neighbors.
- [ ] Routing tables show OSPF-learned routes with the correct metric.
- [ ] You can name the DR and BDR on the multi-access segment and explain how they were elected.
- [ ] Traffic takes the path you engineered with cost.
- [ ] Downstream routers learn a default route dynamically.

### Break/Fix Challenge
Mismatch the hello/dead timers or the area ID on one link. Describe the symptom, then find the cause.

---

## Lab 9: DHCP and DHCP Relay

**Difficulty:** Intermediate  
**Topics:** DHCP server on router, exclusions, options, relay agent

### Scenario
Stop configuring PCs by hand. Hand out addresses automatically across multiple VLANs and a remote site.

### Topology
- 1 router acting as DHCP server for the local LAN
- 1 router (or Layer 3 switch) at a remote site with its own LAN
- A switch with at least two VLANs
- Several PCs

### Requirements
1. Create DHCP pools for each VLAN with sensible ranges, gateways, and a DNS server address.
2. Exclude addresses reserved for infrastructure and servers.
3. Serve the remote site's clients from the central DHCP server without placing a DHCP server at the remote site.
4. Reserve a specific address for one device.
5. Set a lease time appropriate for a guest network versus an employee network.

### Verification Goals
- [ ] Every PC receives an address, mask, gateway, and DNS automatically.
- [ ] No PC is given an excluded address.
- [ ] Remote-site PCs get addresses from the central server.
- [ ] You can view the binding table and identify which client has which address.

### Break/Fix Challenge
Remove the relay configuration and describe what a client's request looks like and why it never arrives.

---

## Lab 10: NAT and PAT

**Difficulty:** Intermediate  
**Topics:** Static NAT, dynamic NAT, PAT (overload), inside/outside, address translation tables

### Scenario
Your company has a few public addresses and many private hosts. Give them internet access and publish an internal web server.

### Topology
- Internal LAN with several PCs and a web server
- 1 edge router
- 1 "ISP" router
- 1 external PC/server representing the internet

### Requirements
1. Use a private block for the inside network. Use a small public-style block between the edge and ISP.
2. Allow all inside hosts to reach the internet using a single public address.
3. Make the internal web server reachable from outside using its own dedicated public address.
4. Configure a second approach using a pool of public addresses and demonstrate how it differs from the first.
5. Provide a default route from the edge router to the ISP.

### Verification Goals
- [ ] Inside PCs reach the external host.
- [ ] The external host reaches the web server by its public address.
- [ ] The translation table shows entries for each type of NAT you configured.
- [ ] You can explain inside local, inside global, outside local, and outside global for one entry.

### Break/Fix Challenge
Swap the inside and outside designations on one interface. Predict the symptoms before testing.

---

## Lab 11: Standard and Extended ACLs

**Difficulty:** Intermediate–Advanced  
**Topics:** Numbered and named ACLs, placement, wildcard masks, VTY protection

### Scenario
Lock down a network with a security policy handed down by management.

### Topology
- 3 LANs: Users, Servers, Management
- 1 or 2 routers connecting them
- A web/FTP/DNS server in the Servers LAN

### Policy to Implement
1. Users may reach the web server only on web ports. No other server access.
2. Users may not reach the Management LAN at all.
3. Management may reach everything.
4. Only the Management LAN may remotely administer the routers.
5. One specific user host is blocked from everything outside its own LAN.
6. Users may ping their gateway but not hosts in other LANs.
7. All denied traffic should be counted so you can see policy hits.

### Requirements
- Use at least one **standard** and at least one **extended** ACL.
- Use at least one **named** ACL.
- Choose placement (interface and direction) for each ACL and be able to justify it.
- Calculate your own wildcard masks.

### Verification Goals
- [ ] Each policy statement is proven with a passing test and a failing test.
- [ ] ACL hit counters increase when expected.
- [ ] Legitimate traffic is not accidentally blocked by the implicit deny.
- [ ] You can explain why standard ACLs go near the destination and extended near the source.

### Break/Fix Challenge
Put an ACL in the wrong direction or in the wrong order. Describe the symptoms and fix it without deleting the whole ACL.

---

## Lab 12: Switch Security

**Difficulty:** Intermediate–Advanced  
**Topics:** Port security, DHCP snooping, Dynamic ARP Inspection, storm control concepts, unused port hardening

### Scenario
Harden the access layer against rogue devices, rogue DHCP servers, and ARP spoofing.

### Topology
- 1 or 2 access switches
- 1 router or Layer 3 device
- A legitimate DHCP server
- A "rogue" device pretending to be a DHCP server
- Several user PCs

### Requirements
1. Restrict each user port to a limited number of MAC addresses and choose a violation response.
2. Allow one port to learn its MAC address automatically and remember it.
3. Allow only the real DHCP server to answer DHCP requests, and block the rogue one.
4. Validate ARP traffic against the DHCP bindings.
5. Place unused ports in a dead-end VLAN and shut them down.
6. Make sure trunk and server-facing ports are handled correctly in the trust model.

### Verification Goals
- [ ] Connecting an unauthorized device to a protected port triggers your configured response.
- [ ] A recovery process brings an error-disabled port back.
- [ ] The rogue DHCP server cannot hand out addresses.
- [ ] You can show the snooping binding table.

### Break/Fix Challenge
Forget to trust the uplink port and describe why every client suddenly fails to get an address.

---

## Lab 13: First Hop Redundancy (HSRP)

**Difficulty:** Advanced  
**Topics:** Virtual gateways, priority, preemption, tracking

### Scenario
A single default gateway is a single point of failure. Build gateway redundancy.

### Topology
- 2 distribution routers (or Layer 3 switches) connected to one access switch
- Both connected upstream to a core/edge router
- 2 to 3 PCs on the LAN

### Requirements
1. Create a virtual default gateway address shared by both routers.
2. Make one router the preferred active gateway and the other the standby.
3. Ensure the preferred router reclaims its role after recovering from a failure.
4. Make the active router give up its role if its uplink to the core fails.
5. Load-balance: split the LAN into two VLANs where each router is active for one of them.

### Verification Goals
- [ ] PCs use the virtual address as their gateway.
- [ ] Shutting down the active router's LAN interface moves traffic to the standby with minimal loss.
- [ ] Shutting down the uplink triggers a role change.
- [ ] You can describe the differences between HSRP, VRRP, and GLBP at a conceptual level.

### Break/Fix Challenge
Misconfigure the group number or virtual IP on one router and observe what happens to the roles.

---

## Lab 14: IPv6 Addressing and Routing

**Difficulty:** Intermediate–Advanced  
**Topics:** Global unicast, link-local, SLAAC, static IPv6 routes, OSPFv3 (optional)

### Scenario
Deploy IPv6 alongside IPv4 in a small three-router network.

### Topology
- 3 routers in a line or triangle
- A PC behind each end router
- A switch on at least one LAN

### Requirements
1. Obtain or invent a documentation-range IPv6 prefix and plan your own /64 subnets for each LAN and link.
2. Enable IPv6 routing on all routers.
3. Address the router interfaces manually, and set explicit link-local addresses on at least one router.
4. Let one LAN's PCs auto-configure their addresses without any server.
5. Configure another LAN's PCs using a stateful method.
6. Provide full reachability using static routes, including a default route.
7. As a stretch goal, replace the static routes with a dynamic IPv6 routing protocol.

### Verification Goals
- [ ] Every interface has the expected global and link-local addresses.
- [ ] PCs on both LANs reach each other over IPv6.
- [ ] You can identify the EUI-64 portion of an auto-generated address and explain how it's built.
- [ ] You can explain why link-local addresses are used for next hops.

### Break/Fix Challenge
Forget to enable IPv6 routing on one router and trace why routes won't forward.

---

## Lab 15: Network Services (NTP, Syslog, SNMP, CDP/LLDP)

**Difficulty:** Intermediate  
**Topics:** Time sync, logging, monitoring, neighbor discovery

### Scenario
Make the network manageable and observable.

### Topology
- 2 routers, 2 switches
- 1 server acting as NTP, syslog, and SNMP/management host
- A few PCs

### Requirements
1. Make one device the authoritative time source, and have everything else sync to it.
2. Send logs from all devices to a central server, with timestamps and an appropriate severity level.
3. Configure SNMP so a management station can read from the devices but not change them.
4. Use a neighbor discovery protocol to map the network without looking at the topology diagram. Then do it again using the open-standard protocol and disable the proprietary one on edge-facing ports.
5. Configure a login banner and an inactivity timeout for management sessions.

### Verification Goals
- [ ] All devices show synchronized time.
- [ ] The syslog server receives messages when you shut and unshut an interface.
- [ ] The SNMP read-only community works and write attempts fail.
- [ ] You can draw the physical topology purely from neighbor discovery output.

### Break/Fix Challenge
Break time sync by pointing a device at the wrong source and note the symptoms in the logs.

---

## Lab 16: Wireless LAN with a WLC

**Difficulty:** Intermediate  
**Topics:** WLC, lightweight APs, WLANs, SSIDs, WPA2/WPA3, VLAN mapping

### Scenario
Roll out secure Wi-Fi for employees and a separate guest network.

### Topology
- 1 wireless LAN controller
- 2 or more lightweight access points
- A switch connected to the WLC and APs
- A router or Layer 3 switch for gateway/DHCP
- A few wireless clients (laptops, tablets, phones)

### Requirements
1. Place the WLC, APs, and clients on appropriate VLANs and subnets of your own design.
2. Make sure the APs can discover and join the controller.
3. Create two SSIDs: one for employees and one for guests, each mapped to a different VLAN.
4. Secure the employee network with a modern pre-shared-key method. Leave the guest network with different security settings, and justify your choice.
5. Ensure clients get addresses via DHCP and can reach a server.
6. Verify the guest VLAN cannot reach internal resources.

### Verification Goals
- [ ] APs register with the controller.
- [ ] Clients associate to the correct SSID and land in the correct VLAN.
- [ ] You can describe the difference between autonomous, lightweight, and cloud-based AP architectures.
- [ ] You can describe the roles of the AP and the WLC in split-MAC.

### Break/Fix Challenge
Put the AP port in the wrong VLAN and observe why the AP can't join.

---

## Lab 17: Capstone: Full Enterprise Network

**Difficulty:** Advanced  
**Topics:** Everything above, combined

### Scenario
You are the sole network engineer for a company with a headquarters, a branch office, and an internet connection. Design and build the whole thing from a blank canvas.

### Business Requirements

**Headquarters**
- Core layer with redundant multilayer switches
- Two access switches with EtherChannel uplinks
- Four departments, each in its own VLAN: 100 users, 50 users, 25 users, and 10 users
- Management VLAN and a server VLAN
- Redundant default gateways
- Wireless for employees and guests

**Branch**
- One router and one switch
- Two VLANs, with 30 users and 12 users
- Router-on-a-stick for inter-VLAN routing
- DHCP served from HQ

**WAN and Internet**
- A point-to-point link between HQ and branch
- A single link to an "ISP" router
- Dynamic routing between sites
- Default route propagated to the branch
- All internal users reach the internet through a single public address
- The internal web server is published to the internet

**Security and Management**
- Remote administration over secure protocols only, from the management VLAN only
- Port security, DHCP snooping, and ARP inspection on all access ports
- ACLs enforcing at least five policies of your own design (document each one)
- Centralized logging, time sync, and read-only SNMP
- Dual-stack: IPv6 on at least the WAN link and one LAN

### Deliverables
1. An addressing plan spreadsheet or table (VLANs, subnets, gateways, ranges). Build it yourself.
2. A topology diagram.
3. The working `.pkt` file.
4. A one-page test plan listing at least 15 tests with expected and actual results.
5. A short list of three things you'd improve if this were production.

### Verification Goals
- [ ] Every PC reaches every permitted destination, and every blocked destination is blocked.
- [ ] You can survive a failed core switch, a failed uplink, and a failed gateway with minimal disruption.
- [ ] A new teammate could manage the network using only your documentation.

### Break/Fix Challenge
Ask someone to introduce three faults while you're not looking. Find and fix all three using only a structured troubleshooting method (bottom-up, top-down, or divide-and-conquer).

---

## Bonus: Troubleshooting Drills

Use any completed lab and run these drills. Time yourself.

1. **Wrong subnet mask** on a single host. Can you spot it from symptoms alone?
2. **Duplicate IP address** on the same VLAN.
3. **Missing return route** so pings go out but never come back.
4. **Wrong VLAN on an access port.**
5. **Trunk allowed-list missing a VLAN.**
6. **Interface shut down** or **err-disabled.**
7. **Wrong default gateway** on a PC.
8. **ACL blocking return traffic.**
9. **Overlapping subnets** between two sites.
10. **Duplex mismatch** on a link.

For each drill, write down:
- The symptom the user would report
- Your first three troubleshooting steps
- The layer of the OSI model where the fault lives
- How long it took you to find it

---

## Study Tips

- Do each lab **twice**: once slowly with notes, once from memory under a time limit.
- Draw the topology and addressing plan by hand before you start.
- Practice explaining *why* each piece works. The exam rewards understanding, not memorization.
- Pair these labs with the official exam topics list so you can check off each objective.
- Revisit your weakest labs weekly.
