#networking #ccan #hardware #level1

A switch is a piece of specialized hardware that is designed to route [[frames|frame]] data from one physical network port to another with little to no latancey. 

data enters through analog signals is processed by the silicon on the device and is then forwarded to the target port.


# hardware 

- __transformer modules__ Located behind the copper ports to isolate the circuts from electrical noise.
- __Physical Layer Chip__ translates the analog signals to digital signals that is then processed by the processing components
- ASIC (Application-Specific Integrated Circuit): The primary brain of an enterprise switch. Unlike a standard computer processor (CPU) that executes instructions sequentially, a switch ASIC uses hardwired logic circuits. It reads the source and destination MAC addresses of an incoming Ethernet frame and determines where to send it in nanoseconds.
- Packet Buffer (SRAM): High-speed internal memory. If multiple ports send traffic to a single destination port simultaneously, the ASIC temporarily stores the extra data frames here to prevent dropped packets.
- TCAM (Ternary Content-Addressable Memory): A ultra-fast, high-cost memory chip. It holds the switch's MAC address table, VLAN configurations, and Access Control Lists (ACLs). TCAM allows the switch to search its entire memory table simultaneously in a single clock cycle.
- CPU (Control Plane Processor): A low-power processor (often ARM or x86) that runs the switch's Operating System (e.g., Cisco IOS). It handles setup, management web pages, SSH logins, and background maintenance (like Spanning Tree Protocol). It does not touch individual data frames during normal switching—that is handled strictly by the ASIC.
- Flash Memory & RAM: Flash stores the operating system image and configuration files; RAM runs the OS environment.

## basic function 
1. the analog signal is received at a given port 
2. it is then sent to the __PHY__ chip 
3. the __ASIC__ stores the frame in the __SRAM__ buffer 
4. it also queris the __TCAM__ to match the destination mac against the internal forwarding table
5. the __ASIC__ determines the correct exit port, if the correct port is not found in the forwarding table the frame is broadcast through all ports on the [[VLAN]]
6. the frame is then added to the ports output queue then the __PHY__ converts the data 

# layer 2 switches 
A [[models#osi|layer 2]] switch forwards traffic based on [[MAC]] addresses, when the frame arrives at the port, the switch reads the destination mac address. It then consults the [[MAC]] address table and forwards the frame to the correct port. On the occasions that the address is not in the table it floods the frame to all ports except the port that the traffic arrived on.

A [[models#osi|layer 2]] switch operates within a single [[broadcast domain]]. Every device connected to receives traffic from every other device. except in the cases where a [[VLAN]] has been configured. They do not appreciate [[IP]] and cannot route traffic between a [[subnetting]]. 

__pros__
- cheaper 
- faster 
- easier to configure 
__cons__
- no inter-[[VLAN]] routing(requires an external router)
- susceptible to traffic flooding
- Scalability(due to the previous reason adding more switches increases the overhead and causes issues)
- no subnetting as it does not deal with [[models#osi|layer 3]] 
## function 

## when to use 

# layer 3 switches 

A [[models#osi|layer 3]] switch combines the port density and hardware based switching of the layer 2 switch and the [[IP]] routing abilities found in a router.  It can forward frames using MAC addresses (like a Layer 2 switch) and route packets between different subnets using IP addresses, all in hardware via dedicated ASICs.

This makes the [[models#osi|layer 3]] switch much faster at inter-[[VLAN]] routing than a traditional router would be. Due to there routing being software based and having fewer [[the many ANs|LAN]] ports. __Layer 3__ switches are there for more commonly used in most enterprise environments as they can effectively route traffic between multiple [[VLAN|VLANs]] and [[subnetting|subnets]] 

__pros__ 
- Inter-[[VLAN]] routing 
- better scalability
- more secure([[ACL]] and granular traffic control at the [[IP]] level)
- faster than a router due to its hardware routing
- QoS enforcement
__cons__
- expensive 
- more complex
- Limited [[the many ANs#WAN's|WAN]] capability

## Layer 3 vs router 

though both can route [[IP]] they are optimized for different jobs. 

Routers function by forwarding packets primarily in software. They are designed for [[the many ANs#WAN's|WAN]] connectivity. There are generally far fewer [[the many ANs|LAN]] ports and there __LAN__ throughput is far lower than that of a __Layer 3 switch__. though they handle much more __WAN__ interactions, [[NAT]], and have advanced routing protocols. 

A **Layer 3 switch** performs routing in dedicated hardware ASICs, making it significantly faster for LAN-to-LAN routing between VLANs and subnets. It offers much higher port density, lower per-port cost, and wire-speed performance across all ports. However, most Layer 3 switches do not have WAN interfaces and are not designed to replace a router for internet connectivity or VPN aggregation.


# Layer 2 vs 3 switch 

__Layer 2__
- small simple networks (under 30 devices on the same subnet)
- when cost is a concern 
- layered networks
__layer 3__ 
- large or growing networks 
- Multi [[VLAN]] networks with many [[subnetting|subnets]] 
- secure and traffic controlled environments
- layered networks
# interfaces 

- __throughput__ the amount of data that is running through a system at a given point in time. 
- __duplex__ the ability for a device to send and receive data simultaneously 
	- __half__ can not send and receive at the same time 
	- __full__ can send and receive simultaneously 

> [!interface errors]
> - __Runts__: Frames taht are smaller thanthe minimum frame size 64 bytes
> -  __Giants__ Frames that are larger than the maximum frame size of 1518 bytes
>   - __CRC__ Frames that failed the CRC check
>   - __Frame__ Frames with the incorrect format
>   - __input errors__ total counter of many errors
>   - __output errors__ Frames the switch tried to send but failed 
>    ```
>    show interfaces f0/0
>    ```


