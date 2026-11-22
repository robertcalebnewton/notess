#cisco #networking
__[[VLAN]] trunking protocol__ is a proprietary __Cisco__ protocol that allows you to configure [[VLAN]] on a central switch which acts as a server which other [[VLAN|VLAN's]] can sync to such that each switch on a network need not be configured.  

- __VTP__ will form a central server and then __VTP clients__ will will sync there [[VLAN]] database to that server 
- three __VTP__ versions 1,2,3
- three modes `server` `client` and `transparent` 
- __CISCO [[switches]] operate in VTP server mode by default__

> [!warning] 
> It is not best practice to use __VTP__ for the following reasons
> 1. network-wide outage risk
> 	- if a switch has a higher revision number all switches in the VTP domain will discard there current configurations in favor of the new one 
> 2.  lack of granular control
> 3. opens up potential for injections

__DISPLAY__
```
show vtp status 
```

# VTP servers

- can add/modify/delete [[VLAN|VLAN's]] 
- the [[VLAN]] database is stored in [[NVRAM]]
- will increase the revision number each time a VLAN is updated 
- will advertise the most recent version of the __VLAN__ database on trunk interfaces and the __VTP__ clients will synchronize their __VLAN__ database to it
- __VTP servers__ will also function as clients syncing to servers with higher revision numbers 

# VTP clients

- can not alter [[VLAN|VLAN's]]  
- Do not store the [[VLAN database]] in __NVRAM__ (this is not true for VTPv3)
- will sync to the server with the highest revision number in their [[VTP domain]] 
- will advertise their __VLAN databases__ and forwarded __VTP__ advertisements to other clients over their trunk ports

# VTP transparent mode

- does not participate in the __VTP domain__ ie does not sync its __VLAN database__ 
- maintains its own __VLAN database__ in its own __NVRAM__ it can alter __VLAN's__ and these alterations will not be advertised. 
- it will forward __VTP__ advertisements that are with in it's domain











