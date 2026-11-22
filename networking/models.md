The following serves as an introdcution and clarification to and on the concept of models in networking. it focuses on the the __TCP/IP__ model and the __OSI__ model explaing and the deminstrating how they can be used to elusidate many concepts int the networking world.
_function_
- __OSI__ model is a tool for explaining network concepts and troubleshooting - __TCP/IP__ a set of rules on which networks operate _each have there place_

![[Pasted image 20260824212212.png]]

# osi 
__open systems intercomunications__ model is a model that seeks to discribe how software and hardware interact with one an other through out the whole of the network stack.

| Layer            | Functional Description                                                                                                                                                                                                  |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application (7)  | Refers to interfaces between network and application software. Also includes authentication services.                                                                                                                   |
| Presentation (6) | Defines the format and organization of data. Includes encryption.                                                                                                                                                       |
| Session (5)      | Establishes and maintains end-to-end bidirectional flows between endpoints. Includes managing transaction flows.                                                                                                        |
| Transport (4)    | Provides a variety of services between two host computers, including connection establishment and termination, flow control, error recovery, and segmentation of large data blocks into smaller parts for transmission. |
| Network (3)      | Refers to logical addressing, routing, and path determination.                                                                                                                                                          |
| Data link (2)    | Formats data into frames appropriate for transmission onto some physical medium. Defines rules for when the medium can be used. Defines the means by which to recognize transmission errors.                            |
| Physical (1)     | Defines the electrical, optical, cabling, connectors, and procedural details required for transmitting bits, represented as some form of energy passing over a physical medium.                                         |
## traversing the OSI model 
as data passes down the protocol stack and across the various pieces of hardware and software, the data goes through changes at every level. this is a process that is called encapsulation. The data at any given layer is known as the _protocol data unit_ or PDU. 

| OSI Layer   | PDU         |
| ----------- | ----------- |
| Application | Data        |
| Presntation | Data        |
| Session     | Data        |
| Transport   | Segment<br> |
| Network     | packet      |
| Data link   | Frame<br>   |
| Physical    | Bits        |

# tcp/ip

| TCP/IP Layer | Function                                                               | Example Protocols                                    |
| ------------ | ---------------------------------------------------------------------- | ---------------------------------------------------- |
| Application  | Represents data to the user and controls dialogue                      | DNS, Telnet, SMTP, POP3, IMAP, DHCP, HTTP, FTP, SNMP |
| Transport    | Supports communication between diverse devices across diverse networks | TCP, UDP                                             |
| Internet     | Determines the best path through the network                           | IP, ARP, ICMP                                        |
| link         | Controls the hardware devices and media that make up the network       | Ethernet, Wireless                                   |

## internet layer 

the internet layer of the __TCP/IP__ modle is the __IP__. it defines unique addresses on a network that allows trafic to be directed into the correct location. it also defines a process of routing that allows the router to determine the best path to send data to. 

_below is an example of and ip addressing data and sending from the transport layer to the internet layer_ 

1. The web client send an http request 
2. __TCP__ encapsulates the HTTP request
3. __IP__ encapsulates the transport segment into a packet adding the source and destination addresses 
4. Lower layers process and send the request to the web server 
5. the web server receives the HTTP requests and sends a __TCP__ acknowledgment back to the client 
6. the web server send the HTTP response down to the transport layer
7. __TCP__ encapsulates the HTTP data
8. __IP__ encapsulates the transport segment into a packet adding source and destination addresses 
9. Lower Layers process and send the response to the requesting web client 
10. the requesting web client sends an acknowledgment back to the web server 

## data encapsulation 

each layer adds its own header information. as the data travels down through the layers it is encapsulated. 

1.  creates and encapsulated the application data with any required application layer headers. for example the, HTTP OK message is returned in an HTTP header followed by the part of contents of a web page 
2. Encapsulates the data supplied by the application layer inside a transport layer header for the end user applications a __TCP__ or __UDP__ header is often used
3. Encapsulate the data supplied by the transport layer inside an Internet layer (IP) header. IP is the only protocol available in the TCP/IP network model at the Internet layer.
4. Encapsulate the data supplied by the Internet layer inside a network access layer header and trailer. This is the only layer that uses both a header and a trailer.
5. Transmit the bits. The physical layer encodes a signal onto the medium to transmit the frame.