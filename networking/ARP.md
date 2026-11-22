#networking 

__ARP(address resolution protocol)__ is used to discover the __MAC__ address of a known __Layer 3__ [[IP]]. regardless of weather or not the [[IP]] of a device is known the [[MAC]] address of the device is needed to establish communication. 


1. __ARP cache lookup__
   - the device checks its __ARP__ cache table to see if the devices [[MAC]] already is located there 
   - if the MAC exits there then the communication is initiated if not the process is continued
1. __ARP request__ 
   - is a [[broadcast]] [[ethernet frame]] sent to all host on a network 
   - it then waits for an __ARP__ reply 
3. __ARP reply__
  - is sent to the host that made the request as a [[unicast]]
