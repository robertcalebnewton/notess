#networking #incomplete 

an [[IP]] __packet__ is made up of a __header__ section and and a __data__ section. The header is typically sent out first and then the data section or the payload follows after. An [[IP]] __packet__ does not use a [[checksum]] or anything concluding the data portion of the __packet__. Typically the __packets__ themselves are encapsulated inside of [[frames]] which detect most errors. 


# life of a packet 

This will be a walk through of the various systems and [[protocols]] a __Packet__ goes through as it traverses a network. The _figure_ below will be referenced often as it is the example network.

- __the packet in question will be one from PC1 to PC2__
- we will assume that no traffic has been sent prior to this 


![[Pasted image 20260928161931.png]]

1. `PC1` sees that the desination [[IP]] is not on its network and thus needs to send the packet to it's [[Gateway]]
2. `PC1` checks its [[ARP]] cache _no match found_
3. `PC1` then makes an [[ARP]] request for the gateway this is [[broadcast]]
4. _note switches learn the [[MAC]] addresses assigned to each interface when the [[Frames|frame]] is receved this will not be mentioned again_
5. `R1` then send a [[unicast]] [[ARP]] reply 
6. `PC1` then sends a header with its [[ip]] as scource and `PC2` [[ip]] as destination the [[frames|frame]] that this is encapsulated in will have `PC1's` [[MAC]] as source and `R1`'s [[MAC]] as the destination 
7.  `R1` removes the [[frames|frame]] and checks its [[routing#routing table|routing table]] for the next hop for the headers destination [[ip]]
8.  The [[ARP]] and next hop steps are repeated for `R2` and `R4` 
9. `R4` removes the [[frames|frame]] and checks its routeing table
10. it then sends an ARP request for `PC4`'s [[MAC]] address
11. `PC2` then sends a the reply
12. `R4` encapsulates the packet in a frame and sends it to `PC2`
13.  the packet has reached its destination 

>[!note]
> - the packet never changed during this process
> - the switches never modified the [[frames]]
> - [[ARP]] request only are sent when there is nothign in the [[ARP]] cache


# IPv4

### header 

The __IPv4__ header has 14 fields, 13 of which are necessary the 14th is optional and is named options. 

![[Pasted image 20260917180544.png]]

### break down 
- Version (4 bits): Identifies the Internet Protocol version being used; always set to binary 0100 (4 in decimal) for IPv4 and 0110 (6 in decimal) for ipv4 so routers know how to parse the header.   
- Internet Header Length / IHL (4 bits): Specifies the total length of the header in 32-bit words, helping routers locate where the header ends and the data payload begins.   
- DSCP (6 bits) real time data streaming makes use of this field ie (VoIP), stands for differentiated services code point, this is used for [[QoS]]
- ECN (2 bits): this field allows end-to-end notification of congestion when it occurs without dropping packets. it is a feature that must be enabled by the network 
- Total Length (16 bits): Defines the entire packet size (header plus the Layer 4 segment) in bytes, ranging from 20 to 65,535 bytes.   
- Identification (16 bits): Assigns a unique ID value created by the sender so the destination host can group and reconstruct fragmented packets.   
- Flags (3 bits): Controls fragmentation using three 1-bit flags (Reserved, Don't Fragment/DF, and More Fragments/MF).  
- Fragment Offset (13 bits): Measures the relative position of a fragment within the original payload in 8-byte blocks for proper reassembly order.   
- Time to Live / TTL (8 bits): Decrements by 1 at each router hop; if it drops to 0, the packet is discarded to prevent infinite routing loops. default is 64  
- Protocol (8 bits): Specifies the upper-layer transport protocol handling the payload data (e.g., 1 for ICMP, 6 for TCP, 17 for UDP, 89 for OSPF).  
- Header Checksum (16 bits): Validates header integrity and is recomputed at every router hop since fields like TTL constantly change.   
- Source IP Address (32 bits): Stores the IPv4 address of the node that originally sent the packet for return traffic and error tracking.  
- Destination IP Address (32 bits): Stores the final recipient's IPv4 address, which routers inspect to determine the next forwarding hop. 
- Options and Padding (0–40 bytes, Optional): Used for advanced routing, testing, or security, with trailing padding added to align the field to a 32-bit boundary.    

