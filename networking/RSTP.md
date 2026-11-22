#networking 

__Rapid Spanning Tree Protocol__  was a introduced by the [[IEEE]]. __RSTP__ is far quicker than [[STP]] and is the default on most devices. It is simaler to __STP__ in many ways blocking specific ports to prevent Layer 2 loops. __RSTP__ elects a [[root bridge]] and a [[root port]] with the same rules as __STP__. Designated ports also have the same rules

- Much faster at adapting to network changes than 802.1D
- All [[VLAN|VLAN's]] share the same instance of __STP__
- can not load balance unlike [[Rapid PVST+]]

> [!Cisco summary]
> "RSTP is not a Timer based spanning tree algorithm lime 802.1D Therefor, RSTP offers an improvemnt over the 30 seconds or more that it takes to move a link to forwarding. The heart of the protocol is the new Bridge to Bridge handshake mechanism, which allows ports to move directly to forwarding"

> [! fetures]
> __UplinkFast__ and __BackboneFast__ are two optional features in classic STP, that must be configured to operate on a switch
> Both are built into RSTP and operate by default

| Speed    | STP Cost | RSTP Cost |
| :------- | :------: | :-------: |
| 10 Mbps  |   100    | 2,000,000 |
| 100 Mbps |    19    |  200,000  |
| 1 Gbps   |    4     |  20,000   |
| 10 Gbps  |    2     |   2000    |
| 100 Gbps |    X     |    200    |
| 1 Tbps   |    X     |    20     |
# port states 

| STP Port State | Send/Receive BPDUs | Frame forwarding (regular traffic) | MAC address learning | Stable/Transitional |
| :------------- | :----------------: | :--------------------------------: | :------------------: | :-----------------: |
| **Discarding** |       NO/YES       |                 NO                 |          NO          |       Stable        |
| **Learning**   |      YES/YES       |                 NO                 |         YES          |    Transitional     |
| **Forwarding** |      YES/YES       |                YES                 |         YES          |       Stable        |
- If a port is administratively disabled it is in the `discarding state` 
- if a port is enabled but blocking traffic to prevent Layer 2 loops it is in the `discarding state` 

# port roles 

- __Root Port__: the port that is closest to the [[root bridge]] becomes the [[root port]] for the switch 
	- the __root bridge__ is the only [[switches|switch]] that does not have a root port 
- __Designated Port__: the port on a [[collision domain]] that sends the best [[STP#BPDU|BPDU]] is that collision domains designated port(only one per segment). 
- __alternate port__: is a discarding port that receives superior __BPDU__ from an other switch 
	- functions like a blocking port in traditional __STP__ 
	- also functions as a backup to the root port in the event of a failure. 
	- the best alternate port will immediately become the root port (this is provided by a feature called UplinkFast)
- __backup port__: is a discarding port that revives a superior BPDU from another interface on the same switch. 
	- this happens whe two interfaces are connected to teh same collision domain (on a hub)
	- _hubs are rarely used and thus so are backup ports_ 

# link types 

- __Edge__ a port that is connected to an end host, moves to forwarding without negotiation 
	- functions like classic STP with port fast enabled 
	- `SW1(config-if)# spanning-tree portfast`
- __Point to Point__: is a direct connection between two switches 
	- operate in full duplex mode 
- __Shared__ is used for hubs and there for dumb and bad 



