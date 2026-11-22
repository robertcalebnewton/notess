__Spanning Tree Protocol__ is an IEEE 802.1D protocol that is designed to prevent [[broadcast storms]] in a network. It prevents Layer 2 loops by placing redundant interfaces in a blocking state. They act as backups and can switch to a forwarding state in the event that an active interface fails. 

- switches from all vendors run __STP__ by default 
- interfaces in a __forwarding__ state behave normally 
- interfaces in a __blocking__ state only accept __STP__ messages (Bridge Protocol Data Units)
- __STP__ enabled switches send _hello BPDU's_  out of all interfaces the default timer is 2 seconds 
	- if a switch receives such a message it knows that that interface is connected to another switch as they are the only devices that use __STP__

> [!note]
> 1) One switch is elected as the root bridge. All ports on the root bridge are designated ports (forwarding state). Root bridge selection:
1: Lowest bridge ID
> 2) Each remaining switch will select ONE of its interfaces to be its root port (forwarding state). Ports across from the root port are always designated ports.
Root port selection:
1: Lowest root cost
2: Lowest neighbor bridge ID
3: Lowest neighbor port ID
>3) Each remaining collision domain will select ONE interface to be a designated port (forwarding state). The other port in the collision domain will be non-designated (blocking)
Designated port selection:
# BPDU

- [[switches]] use the Bridge ID field to elect a [[root bridge]]
- each remining switch will select one of its interfaces to be its [[root port]], the interface with the lowest [[root cost]] will be the root port. 
- the default priority is 32768 on all switches 
- so by default the [[MAC]] address is used as the tie breaker(lowest MAC address becomes the root bridge) 

![[Pasted image 20261021174742.png]]

# port states 
| STP Port State | Send/Receive BPDUs | Frame forwarding (regular traffic) | MAC address learning | Stable/Transitional |
| :------------- | :----------------: | :--------------------------------: | :------------------: | :-----------------: |
| **Blocking**   |       NO/YES       |                 NO                 |          NO          |       Stable        |
| **Listening**  |      YES/YES       |                 NO                 |          NO          |    Transitional     |
| **Learning**   |      YES/YES       |                 NO                 |         YES          |    Transitional     |
| **Forwarding** |      YES/YES       |                YES                 |         YES          |       Stable        |
| **Disabled**   |       NO/NO        |                 NO                 |          NO          |       Stable        |
