#incomplete #networking 

__routing__ is the process that __routers__ use to determine the path that [[IP]] [[packets]] should be taking over a network in order to reach their desired destination. routers store routes to known destinations inside of a __routing table__. when [[packets]] are revived __routers__ will check their __routing table__ to find which route to direct the [[packets|packet]] on.

__Methods__
- __Dynamic Routing:__ when routers use dynamic routing [[protocols#OSPF|protocols]] to share routing information with each other automatically growing the table
- __Static Routing:__ when a route has been manually configured inside the routing table 

> [!what is a route?]
> a route tells the router: 
> - send [[packets|packet]] some destination, so send the [[packets]] to the next hop in that path
> - if the destination is connected toh the router send it there
> - if the routers IP is the destination then the packet is received 
> - if the __router__ receives the packet and the destination [[IP]] does not match a route in the system then the [[packets|packet]] is dropped 

# routing table
[routing tabe info](https://blog.cloudmylab.com/what-is-a-routing-table)
A __routing table__ is a data structure stored in routers and  other network devices that maintains a map of network destinations and there subsequent for forwarding path. Each time a packet is received the table is consulted and the [[packets|packet]] is forwarded on its optimal path. 

__parts of  the table__
- __destination network:__ This identifies which [[IP]] addresses the routes applies to. 
- __Next Hop/[[Gateway]]:__  for directly connected networks, this is often __0.0.0.0__. For remote networks this field contains the next hop router. 
- __interface__ which of the routers own connections to use for sending the [[packets|packet]]
- __Metric:__ the cost asscoiated with a route.  different [[protocols#routing|protocols]] use different metrics (__RIP__ uses hop count, __OSPF__ uses cost based bandwidth)
- __[[Administrative Distance]]__  It determines how trustworthy a route source is when different protocols advertise the same route. Lower AD is better 
- __[[Default Route]]:__ is the catch all for any destination not listed. 
## how it works
When a device gets a [[packets|packet]], it checks the destination [[IP]] and scans the routing table for the closets network prefix (the most specific match). If there are multiple equal routes __metrics__ and [[Administrative Distance]] are used to pick the most reliable path. If no exact route is found, then the default route is used. if it can not be found then the [[packets|packet]] is dropped and the [[ICMP]] destination unreachable message is sent to the sender. 

# static vs dynamic routing tables

| Feature                | Static Routing                                | Dynamic Routing                              |
| :--------------------- | :-------------------------------------------- | :------------------------------------------- |
| **Configuration**      | Manual, explicit                              | Automatic via protocol                       |
| **Scalability**        | Poor (doesn't scale)                          | Excellent                                    |
| **Fault Tolerance**    | None (manual intervention required)           | Automatic failover                           |
| **Bandwidth Overhead** | None                                          | Protocol updates consume bandwidth           |
| **CPU/Memory**         | Minimal                                       | Protocol processing requires resources       |
| **Convergence**        | Instant (no convergence needed)               | Seconds to minutes                           |
| **Security**           | More secure (no protocol to attack)           | Requires authentication                      |
| **Best Use Cases**     | Small networks, stub networks, default routes | Medium to large networks, redundant topology |
| **Complexity**         | Simple                                        | Moderate to high                             |
| **Predictability**     | Completely predictable                        | Depends on protocol behavior                 |
# static routing 
each route is manually configured, this provides stability and reduces overhead that protocols introduce. Routes to an [[ISP]] are typically __static__. They also provide more security as they don't advertise network info.  However there are many downsides, lack of fault tolerance, lack of scale, and a lack of load balancing. 

```
ip route IP-ADDRESS NETMASK NEXT-HOP
```

# dynamic routing
[[protocols#routing protocols|routing protocols]] automatically learn about networks and exchange that info as the network changes the __tables__ update themselves. It is better for medium to large networks benefit from the automatic routing updates. It also adds redundancy whenever a network device fails. 

| Protocol | Type                       | Key Feature                        | Use Case                       |
| :------- | :------------------------- | :--------------------------------- | :----------------------------- |
| RIP      | Distance Vector            | Simple, uses hop count             | Small, legacy networks         |
| EIGRP    | Hybrid (Cisco Proprietary) | Fast convergence, efficient        | Cisco enterprise networks      |
| OSPF     | Link State                 | Hierarchical, area-based           | Large enterprise networks      |
| BGP      | Path Vector                | Controls routing between ISPs/ASes | Internet/enterprise WANs       |
| IS-IS    | Link State                 | Area-based, scalable, used by ISPs | Large carrier/service networks |
## cisco 

> [!note]
> the following describes the commands and formats associated with the Cisco ios routing tables

__to display the routing table 
```command
show ip route 
```

__codes__ 
- `L - local`: A route to the exact [[IP]] address configured on the interface (they will display a /32 netmask meaning that that ip address can only match itself)
- `C - connected`: A route to the network the interface is connected to (with the actual netmask configured on the interface)
	_example_ `C     192.168.1.0/24 is direcly connected, GigabitEthernet0/2`
	`192.168.1.0` through `192.168.1.255`  will be sent out the `G0/2` interface
_these two are neither static or dynamic but are created when ever an interface is configured_

>[! note]
> When there is confusion about which route to take when it could be either. the pakcet will take the most specific route.
> __IP__ `192.168.1.1` will take route 2 when 
> __route 1__ = `192.168.1.0/24`
> __route 2__ = `192.168.1.1/32`
