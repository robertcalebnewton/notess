#ccan #networking 

__dynamic trunking protocol__  is a __cisco__ proprietary protocol that will dynamically and automatically configure ports as access ports or trunk ports. 

- is a proprietary Cisco protocol
- __it is enabled by default__
- can cause unintended changes to a network 
- it will only form a trunk with other switches

> [!warning] 
> for security purposes it should always be disabled 
> 
>__[[switches|switch]] spoofing__ is a technique were an attacker connects a device to the standard user-facing switch port. The device sends __DTP__ meassages pretending to be a Cisco switch. The port then accepts the negotation and dynamically forms a trunk link that grants the malicous user access to all the [[VLAN]]'s that the switch has configured on it.

__disable command__
```
switchport nonegotiate 
```
- `switchport mode access` _will disable this by default_
###### trunk encapsulation negotiation
[[switches]] that support both [[802.1Q]] and [[ISL]] encapsulation can use __DTP__ to negotiate the encapsulation that they use 
- __this too is enabled by default__
- [[ISL]] is preferred over [[802.1Q]] so if both switches support the two [[ISL]] will be chosen
- __DTP__ [[frames]] are sent in __VLAN1__ when using [[ISL]]  or in the native __VLAN__ for [[802.1Q]]

# configuring 

```
SW(config-if)# switchport mode dynamic ?
auto 
desirable
```

_this command is used to configure DTP_ 

`dynamic disirable` will actively try to form a trunk with an other Cisco switch if they are in the following modes
- `switchport mode trunk`
- `switchport mode dynamic desirable`
- `switchport mode dynamic auto`

`dynamic auto` mode will not actively try to form a trunk with other Cisco switches, but it will form a trunk if the switch it is connected is tying to form a trunk. this will happen in the following modes  
- `switchport mode trunk`
- `switchport mode dynamic desirable`

| Administrative Mode   |      Trunk       | Dynamic Desirable |      Access      | Dynamic Auto |
| :-------------------- | :--------------: | :---------------: | :--------------: | :----------: |
| **Trunk**             |      Trunk       |       Trunk       | X (Incompatible) |    Trunk     |
| **Dynamic Desirable** |      Trunk       |       Trunk       |      Access      |    Trunk     |
| **Access**            | X (Incompatible) |      Access       |      Access      |    Access    |
| **Dynamic Auto**      |      Trunk       |       Trunk       |      Access      |    Access    |
