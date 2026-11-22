# TCP 
#incomplete #networking 
 [wiki](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#)
 [tcp](https://www.rfc-editor.org/info/rfc9293/#section-3.1-6.6.2)
- __TCP__ is disiginded for accurate delivery rather than timely, as such it is not suitable for real time applications. 
- It guarantees that all bytes are delivered accurately and in the order that they where sent

__TCP transfer control protocol__ provides a communication service at a level between the application and the internet protocol. It is a host to host communication that takes at the transport layer. __TCP__ handles the the handshake and the transmission details and provides an abstraction of the connection to the application, most often through a [[network socket]] interface.

The porpise of __TCP__ is to ensure that all of the [[packets]] arrive and in the correct order. Due to the network congestion, [[load ballancing]], or network behavior, packets may be lost duplicated or delivered out of order. __TCP__ detects such issues and requests a re-transmission of lost data, reorganize out of order data and helps to prevent congestion so that these issues do not occur in the futrue. If the data still remains undelivered the source is notified of the failure. Once the __TCP__ receiver has reassembled the octets originally transmitted, it passes them to the revcieving application.

To insure that all of the packets are delivered in the order that they were sent __TCP__ uses a technique known as positive acknowledgement with re transmission. this require the resever to send an acknoldegement message as it resives the data. The sender keeps a record of each packet it sends and maintains a timer for each pakcet sent. The sender will resend the packet if to much time has passed between the packets transmission and the subsequent acknowledgement message. IP handles the delivery of the data the __TCP__ tracks the segments that the data is divided into. 


_example_
- __Step 1__ The web client sends an HTTP request for a specific web server down to the transport layer.
- __Step 2__ TCP encapsulates the HTTP request with a TCP header and includes the destination port number for HTTP.
- __Step 3__ Lower layers process and send the request to the web server.
- __Step 4__ The web server receives HTTP requests and sends a TCP acknowledgment back to the requesting web client.
- __Step 5__ The web server sends the HTTP response down to the transport layer.
- __Step 6__ TCP encapsulates the HTTP data with a TCP header.
- __Step 7__ Lower layers process and send the response to the requesting web client.
- __Step 8__ The requesting web client sends an acknowledgment back to the web server.

## TCP Header 


![[Pasted image 20260826185459.png|700]]


- __source port__
	- is the sending port 
	- __16 bits__
- __destination port__
	- use your damn head 
	- __16 bits__
- __Sequence Number__
	- __32 bits__
	- The sequence number of the first data octet in this segment (except when the SYN flag is set). If SYN is set, the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1
- __ACK Number__
	- If the ACK control bit is set, this field contains the value of the next sequence number the sender of the segment is expecting to receive. Once a connection is established, this is always sent.
- __Data Offest__
	- The number of 32-bit words in the TCP header. This indicates where the data begins. The TCP header (even one including options) is an integer multiple of 32 bits long.
- __Reserved__
	- A set of control bits reserved for future use. Must be zero in generated segments and must be ignored in received segments if the corresponding future features are not implemented by the sending or receiving host.
- __Control Bits__
	- __CWR__ congestion window Reduced
	- __ECE__ ECN-Echo
	- __URG__ Urgent pointer field is significant 
	- __ACK__  Acknowledgement feild 
	- __PSH__ Push Function 
	- __RST__ reset the connection 
	- __SYN__ sync sequence numbers 
	- __FIN__ no more data(set by the sender) 
- __Window__  
	- The number of data octets beginning with the one indicated in the acknowledgment field that the sender of this segment is willing to accept. The value is shifted when the window scaling extension is used
	- The window size MUST be treated as an unsigned number, or else large window sizes will appear like negative windows and TCP will not work (MUST-1). It is RECOMMENDED that implementations will reserve 32-bit fields for the send and receive window sizes in the connection record and do all window computations with 32 bits
- __checksum__ 
	- The checksum field is the 16-bit ones' complement of the ones' complement sum of all 16-bit words in the header and text. The checksum computation needs to ensure the 16-bit alignment of the data being summed. If a segment contains an odd number of header and text octets, alignment can be achieved by padding the last octet with zeros on its right to form a 16-bit word for checksum purposes. The pad is not transmitted as part of the segment. While computing the checksum, the checksum field itself is replaced with zeros.
	- The checksum also covers a pseudo-header (Figure 2) conceptually prefixed to the TCP header. The pseudo-header is 96 bits for IPv4 and 320 bits for IPv6. Including the pseudo-header in the checksum gives the TCP connection protection against misrouted segments. This information is carried in IP headers and is transferred across the TCP/network interface in the arguments or results of calls by the TCP implementation on the IP layer.
```
                +--------+--------+--------+--------+
                |           Source Address          |
                +--------+--------+--------+--------+
                |         Destination Address       |
                +--------+--------+--------+--------+
                |  zero  |  PTCL  |    TCP Length   |
                +--------+--------+--------+--------+
```


## TCP ack 

__TCP__ provides error recovery, during data transfer sessions between the two end points. The acknowledgement fields in the __TCP__ header track each byte of data that is transferred to ensure that data that failed to transfer may be resent. 

_in the below figure we see two end points. all 3000 bytes were sent and the (4000) suggests that the next byte will be the ack byte_

![[Pasted image 20260907160027.png]]

_in the figure below we see an example with errors where the second transmission was lost. so the client sends a message to the effect of "hey i didn't get sequence 2000". it is then resent. This is known as __par(positive ack with re transmission)__. in this way the data is recovered_ 
![[Pasted image 20260907160406.png]]

it should be noted that the server maintains a re transmission timer that awaits the __ACK__ signal in cases where the __ACK__ or all segments are lost. 

## TCP flow control

__TCP__ maintains flow control trough a process known as windowing. The two devices negotiate the size of a window at the moment the connection is established. This size is dynamically renegotiated during the duration of their connection. the size increases until the maximum size of 65,535 bytes is reach or there are errors that occur. The size is set in the Window field residing in the header. After sending the amount of data specified in the window size, the source must receive an acknowledgment before sending the next window size of data.

## TCP connections


### establishment 
Connection establishment is the process of initializing Sequence and Acknowledgment fields and agreeing on port numbers and window size. The three-way connection establishment phase shown below.

![[Pasted image 20260907161518.png]]

In the figure, DPORT and SPORT are the destination and source ports. SEQ is the sequence
number. In bold are SYN and ACK, each representing a 1-bit flag in the TCP header used to signal connection establishment. TCP initializes the Sequence Number and Acknowledgment Number fields to any number that fits into the 4-byte fields. The initial Sequence Number is a random 32-bit number generated with each new transmission. The Acknowledgment Number is received back and increments the sender’s sequence number by 1.

### Termination 

Upon completion of the transmission a four way termination sequence occurs. in the sequence the __FIN__ flag is used. Figure below 

![[Pasted image 20260907161915.png]]

# UDP

TCP establishes and terminates connections between endpoints, whereas UDP does not. Therefore, UDP is called a connection less protocol. It provides no reliability, no windowing, and no reordering of the data. However, UDP does provide data transfer and multiplexing using port numbers, and it does so with fewer bytes of overhead and less processing than TCP. Applications that use UDP, such as VoIP, trade the possibility of some data loss for less delay.

![[Pasted image 20260907162108.png]]



# routing protocols



# 0SPF