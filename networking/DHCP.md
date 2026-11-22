#networking #incomplete 

> [!warning]
> the information with in is accurate but surface level and highly reductive

__Dynamic Host Configuration Protocol__ is a protocol that is used on [[IP]] networks for automatically assigning __IP__ addresses and other communication parameters to devices on a network. It is made up of two parts the __DHCP__ server and the clients. most residential routers have an instance of __DHCP__ running. It allows hosts to automatically and dynamically learn about the networks configuration eg. __IP addresses__, __subnet mask__, __[[default gateway]]__, __DNS server__ and others. 


__ports__ 
- __DHCP__ servers use [[protocols#UDP|UDP]] on port `67`
- __DHCP__ clients use [[protocols#UDP|UDP]] on port `68`

# DHCP pool 
is a pool of addresses that the DHCP server can pull from when giving new __IP's__ to clients 
# DHCP relay 

 some engineers configure each router to act as a __DHCP__ server for its [[LAN|LAN's]] but larger enterprises often choose a centralize __DHCP__ server. If the server is centralized, it won't receive the DHCP clients' broadcast DHCP messages. (broadcast messages don't leave the local subnet). To fix this, you can configure a router to act as a **DHCP relay agent**. The router will forward the clients' broadcast DHCP messages to the remote DHCP server as [[unicast]] messages.

# DHCP leasing

__leases__ are given to clients for there __IP__ addresses and at the end of the lease they will be given a new address. though a device can request a certain address in this process such that it will maintain its current address. 

### 1.  DHCP Discover Message
```mermaid
flowchart LR
	A["Client"] ----> B["Server"]
```
the __discover message__ is a [[broadcast]] message the purpose of this message is to inspect the network for any __DHCP__ servers. 

### 2. DHCP Offer
```mermaid
flowchart RL
	A["Server"] ----> B["Client"]
```
This message contains an __IP__ address for the client to use as well as additional information such as the __DNS__ server, the [[Default Gateway]] ,and more

### 3. DHCP Request
```mermaid
flowchart LR
	A["Client"] ----> B["Server"]
```
This is a [[broadcast]] message that informs the server that it would like to use the __IP__ address that it was offered. There may be many DHCP servers that exist in a network all of which will respond with an offer to the discover message. So the client must request the __IP address__ from a particular server 

### 4. DHCP ACK 
```mermaid
flowchart RL
	A["Server"] ----> B["Client"]
```
this is a direct response to the client confirming the usage of this __IP__ address. Upon receiving this the client then updates its __IP__ address accordingly 

# cisco 

```
R1#show ip dhcp binding
```
## DHCP server config

```
R1(config)#ip dhcp exlcuded-address LOW HIGH

R1(config)#ip dhcp NAME

R1(dhcp-config)#network 192.168.1.0 /24

R1(dhcp-config)#dns-server 8.8.8.8

R1(dhcp-config)#domain-name example.com

R1(dhcp-config)#domain-router 192.168.1.1

R1(dhcp-config)#lease DAYS HOURS MINUTES
```

## DHCP relay agent config 
```
R1(config)#interface g0/1
R1(config-if)#ip helper-address 192.168.10.10
R1(config-if)#do show ip interface g0/1
```

## DHCP client config 
```
R1(config)#interface g0/1 
R1(config-if)#ip address dhcp
R1(config-if)#do show ip interface g0/1
```