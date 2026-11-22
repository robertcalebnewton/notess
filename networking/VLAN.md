#incomplete 
A __Virtual local area network__ is a [[LAN]] where the devices are logically grouped regardless of there physical location. it exists as a [[virtual network]] at [[models#osi|layer 2]]. it allows for the separation of traffic without the need for more hardware. This does not mean that each computer or device can not communicate with each other rather that there broadcast frames are limited to the aforementioned __VLAN__.

__purpose__
- security 
- traffic management
- simplicity 

![[Pasted image 20260928201005.png]]


> [!note]
> each traffic from each subnet are sent through the router for each network to communicate with each other. This MUST be configured in the switch so that frames are not broadcasted between networks

- __access port__  is a switchport which belongs to one __VLAN__ and usually connects end host

# how it works 


# trunk ports
__trunk port__ is a switch port that carries multiple __VLAN's__
- __TAGGING__ switches will tag all frames that are sent over a trunk link this allows them to know which __VLAN__ a frame belongs to _only applies to trunk ports_
- __protocols__ 
	- ISL Inter switch link
	- IEEE [[802.1Q]]
- __Native VLAN__ is  1 by default on all trunk ports and must be manually configured 
  - [[switches]] wont add a [[802.1Q]] tag to frames in the native VLAN
  - when a switch revives an untagged frame on a trunk port it assumes that the frame belongs to  the __native VLAN__
- all vlans are allowed on a trunk by default 

## commands 

### view trunk port's status
```
show interfaces INTERFACE switchport
```



# multi layer switching 
having a router conduct all of the inter__VLAN__ routing can congest a network thus it is wise to use [[switches#layer 3 switches|layer 3 switches]]. the muti layer switch will then have  a routing table allowing for intelligent direction of traffic 

- __SVI's__  _switch virtual interfaces_ are virtual interfaces that can have multiple [[IP]] addresses assigned to them
  