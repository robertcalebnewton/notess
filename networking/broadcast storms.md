#networking #definition 

a __broadcast storm__ also known as __broadcast radiation__ takes place when a [[broadcast]] or [[Multicast]] [[frames|frame]] is sent into a network that has multiple layer 2 routes to a client device or a router. Due to the lack of a __TTL__ feature in an [[ethernet frame]] the frame is constantly broadcasted in a loop. this will consume a fair amount of network resources in the network causing the network to be unable to transmit normal traffic. 




# example

Take for example the simple network topology listed below. lets say that __PC1__ sends a request to __PC2__. once it reaches the first switch it will flood the frame out of each interface. once the frame reaches its destination __PC2__ will send its [[ARP]] reply but there is still a series of frames that are going to be flooding the network long after the reply reaches its destination. These frames will continue to be sent causing a broadcast storm. 


![[Pasted image 20261021172044 1.png]]