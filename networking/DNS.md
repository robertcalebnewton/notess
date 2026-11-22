#networking #incomplete 

__Domain Name System__ is a hierarchical and distributied system that provides a [[domain name]] to a [[IP]] address. the __DNS__ server that your machine uses can be manually configured or learned by [[DHCP]]  a __Cisco__ [[routing|router]] can act as a __DNS__ server and client. To prevent __DNS__ servers from being hammered with requests devices will store __DNS__ records in a __DNS CACHE__ 

- __A records__ are used to map __Domain Names__ to an __IPv4__ address 
- __AAAA records__ are used to map __Domain Names__ to an __IPv6__ address 
- __CNAME__ canonical name is used to map a name to another name 
- __DNS__ uses both __UDP__ and __TCP__
- [[networking/Ports|Port Number]]: `53`


> [!how it works]
>  if the device decides to reach a domain name that it is not familar with it will send a __DNS__ query to its configured __DNS__ server. The server will then reply with the __IP__ associated with the __Domain Name__. 
> 

# cisco 

## router as DNS server 

```
R1(config)#ip dns server #configure as a DNS server

# configure a list of hostname IP address mappings
R1(config)#ip host R1 192.168.0.1
R1(config)#ip host PC1 192.168.0.101
R1(config)#ip host PC2 192.168.0.102
R1(config)#ip host PC3 192.168.0.103

R1(config)#ip name-server 8.8.8.8 #name server selecetion using googles DNS
R1(config)#ip domain lookup #enables R1 to preform DNS queries 
```

## router as DNS client

```
R1(config)#ip name-server 1.1.1.1
R1(config)#ip domain lookup
```

## host domain name 
```
R1(config)#ip domain name calebsdomain.com
```
_see [[CiscoSSH]]_ 

