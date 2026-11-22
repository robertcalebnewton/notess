#networking 

[practice](https://subnetting.org/)
[practice](https://subnettingpractice.com/)

> [!disclamer]
> [[IPv6]] rules found elsewhere 

A __subnet__ is a logical subset of an [[IP]] network. Dividing a network into one or more networks is known as subnetting. Computers that are of the same __subnet__ must be addressed with an identical group of [[most significant bits]] of there [[IP]] addresses. 

__purpose__ 
- __security:__ sensitive devices are separated from other devices due to being isolated within their own subnet. 
- __reduced traffic:__ broadcast messages only reach devices that are located on the same subnet, not on the network 
- __efficiency:__ rather than wasting space on an IP address you could limit the amount of space used 
- enables [[VLAN]] subnetting

>[!example]
> Network block: 192.168.1.0/24 (256 total addresses)
>- **168.1.0/26 → Sales team (62 usable hosts)**
>- **168.1.64/26 → IT department (62 usable hosts)**
>- **168.1.128/26 → Server room (62 usable hosts)**
>- **168.1.192/26 → Guest Wi-Fi (62 usable hosts)**

# Subnet Mask
each [[IP]] address is made up of a __network portion__ and a __host portion__. The __Subnet Mask__ tells the devices and routers exactly where the __network portion__ ends and the __host portion__ begins. it is a 32-bit number the rules are fairly simple the __network portions__ are denoted by `1's` and the __host portion__ by `0's`. 

![[Pasted image 20260921225353.png|681]]

_the above subent mask will give us 1 network with 254 host devices_ 

> [!note]
> the number of posible hosts can be calculated by doing the following where `n` is the number of host bits or `0`'s
> $$ 
> 2^n - 2
> $$

## class C example Lets say for example that we have a [[IP#classes|class 3]] ip address and we want to make four networks that have 50 devices. The ideal subnet for this one is a `/26` subnet this will give us 64 host devices on each subnet of which there are 4. 

![[Pasted image 20260928173627.png]]


__table of subnets for class C__
![[Pasted image 20260928173055.png]]

# CIDR 

__Class Inter-Domain Routing__ is a way of denoting the __Subnet mask__ having a `/` followed by a number. It effectively is found by counting up the number of `1`'s ie the network portion of the subnet mask

>[!note]
>`/31` configurations are valid for point to point networks as they do not require a broadcast address or a network address.

# VLSM
__Variable Length Subnet Mask__ is the process creating subnets that have a different number of host devices

_for example lets say we need 4 networks on network `192.168.1.0/24`_

- Network A _needs 110 hosts_
- Network B _needs 29 hosts_
- Network C _needs 8 hosts_
- Network D _needs 45 hosts_
- A point to point connection 
  
> [!note]
>  the odder is largest to smallest

__Network A__ 
- Network address: `192.168.1.0/25`
- Broadcast: `192.168.1.128/25`
- First usable: `192.168.1.1/25`
- Last usable: `192.168.1.127/25`
- number of hosts: 126
_this is half of the address space of `192.168.1.0/24`_
__Network C__
- Network address: `192.168.1.128/26`
- Broadcast: `192.168.1.191/26`
- First usable: `192.168.1.129/26`
- Last usable:  `192.168.1.190/26`
- number of hosts: 62
__Network B__
- Network address: `192.168.1.192/27`
- Broadcast: `192.168.1.223/27`
- First usable: `192.168.1.193/27`
- Last usable:  `192.168.1.222/27`
- number of hosts: 30
__Network D__
- Network address: `192.168.1.224/28`
- Broadcast: `192.168.1.239/28`
- First usable: `192.168.1.225/28`
- Last usable:  `192.168.1.238/28`
- number of hosts: 14
__point to point__
- Network address: `192.168.1.240/30`
- Broadcast: `192.168.1.243/30`
- First usable: `192.168.1.241/30`
- Last usable: `192.168.1.242/30`
- number of hosts: 2







