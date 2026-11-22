#incomplete #networking 

__Internet Protocol__ is the [[models#osi|layer 3]]  protocol that is responsible for delivering [[datagrams]] across a network. Data that is transferred across the network in the form of [[packets]]. Every device and domain has its own unique __IP addresses__. These addresses are used to route the packets to there destination. 

- __IP address__ is a unique identifier assigned to a device. It is made of a series of numbers and is usually __IPv4__ or __IPv6__.

__Internets__
The internet is made up of a series of [[autonomous systems]]. there are a great many routing protocols that are responsible for routing packets across these [[autonomous systems]]. [[routing tables]] indicate the paths that [[packets]] take to travel from AS to AS until its destination is found. 

> [!how it works]
when two computers of a different subnet communicate through a router the following things happen 
>1. __PC A__ constructs an __IP__ [[packets|packet]]
>- __PC A ip:__  `192.168.1.10`
>- __PC B ip:__  `192.168.2.10`
>2.  __PC A__ is checks the IP of __PC B__ sees that the __IP__ is under a different network and has to send its [[packets]] to the default [[Gateway]] eg `192.168.1.1`.
>3. [[ARP]] is used to find the [[MAC]] address of the default [[Gateway]]
>4.  __PC A__ wraps the __IP__ [[packets|packet]] inside an [[frames|ethernet frame]]
>5. __PC A__ puts the frame onto the physical [[switches|switch]]
>6. The routers [[NIC]] picks up the [[frames|frame]]. recognizes its own [[MAC]] address as the destination. removes the header and trailer, and hands the payload to [[models#osi|layer 3]]
>7. checks the time to live of the packet
>8. the router checks the [[routing tables]] for the destination __IP__ and finds the route.
>9. router then  [[ARP]]s __PC B's__ ip address 
>10. the router then makes a new Ethernet frame and sends it off to __PC B__

# IPv4 

__Internet protocol version 4__ is the most widely used system for identifying items on a network. It uses a 32 bit signature represented by 4 [[base conversion|octets]] separated by periods. they consist of a __HOST__ portion and a __Network__ portion. These addresses are unique to a device on a network. not only does the device indicate the device it indicates where the device is located in the networking making routing possible. 

- __HOST portion__ Identifies the specific device on a network 
- __Network portion__ Identifies the specific network segment

  

>[!note]
> __Finding the max hosts per network__ 
$$2^n - 2$$
> - the minus two is for the broadcast and the network address 
> - the __HOST__ portion of the address is all 0's that is the network address 
> - the __HOST__ portion of the address that is all 1's is the [[broadcast]] portion of the network 



![[Pasted image 20260928120447.png]]

_each octet can be represented in decimal form in the manner shown above_



## private networks 
around 18 million of the four billion addresses available in __IPv4__ in three separate ranges are reserved for private networking. [[packets]] that have destination addresses that are in these ranges are not [[routeing|routable]] on the public internet. private devices are incapable of interfacing communicating with __public networks__ and require the use of a [[NAT]] at a routing [[gateway]]. 

| Name         | CIDR block       | Address range                   | Number of addresses | *Classful* description                 |
| ------------ | ---------------- | ------------------------------- | ------------------- | -------------------------------------- |
| 24-bit block | *10.0.0.0/8*     | *10.0.0.0 – 10.255.255.255*     | 16,777,216          | Single Class A                         |
| 20-bit block | *172.16.0.0/12*  | *172.16.0.0 – 172.31.255.255*   | 1,048,576           | Contiguous range of 16 Class B blocks  |
| 16-bit block | *192.168.0.0/16* | *192.168.0.0 – 192.168.255.255* | 65,536              | Contiguous range of 256 Class C blocks |
> [!WARNING]
> two private networks cannot interoperate through the public internet. Other methods must be used to resolve this issue 

## loop back 
the __IPv4__ addresses in `127.0.0.0/8` are reserved for loopback. __IP__ [[packets]] whose source address is inside this network rage typically wont appear outside of a host. Packets received on a non-loopback interface with a loopback source or destination address must be dropped.

## classes
classful networkfing has been replaced by [[subnetting#CIDR|CIDR]] 

| Class   | Address range               | Default subnet mask | Max usable hosts | Typical use                       |
| :------ | :-------------------------- | :------------------ | :--------------- | :-------------------------------- |
| Class A | 1.0.0.0 – 126.255.255.255   | 255.0.0.0           | 16,777,214       | Large enterprises, ISPs           |
| Class B | 128.0.0.0 – 191.255.255.255 | 255.255.0.0         | 65,534           | Universities, government agencies |
| Class C | 192.0.0.0 – 223.255.255.255 | 255.255.255.0       | 254              | Small businesses, home networks   |
| Class D | 224.0.0.0 – 239.255.255.255 | N/A (multicast)     | N/A              | Multicast streaming               |
| Class E | 240.0.0.0 – 255.255.255.255 | N/A (reserved)      | N/A              | Experimental, not in use          |
>[!warning]
> `127.0.0.0/8` is reserved for loop back addresses 

# IPv6 