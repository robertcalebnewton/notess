
# Ethernet
 __Ethernet__ a collection of network and protocol standerds
- defined in the [[IEEE]] standerds


## cable memes 
- __RJ45__ the connectpor at the end of an ethernet port 
- __UTP(unsheilded twised pairs)__ the copper wire used to connect the __RJ45__ the twist protects from electrical interfercance 
- **MDI Ports** (used by end devices like PCs, servers, and routers) transmit on pins 1 & 2 and receive on pins 3 & 6.
- **MDI-X Ports** (used by networking devices like switches and hubs) flip this mapping—they receive on pins 1 & 2 and transmit on pins 3 & 6.
- __SFP(small forfactor plugable)
- __FIber Optic Cable__ sends light rather than electrical signals for anolog comunticaiton between wires  
	- single mode 
		- longer
		- more costly 
	- multi mode 
		- wider core
		- shorter 



### copper 

| **Speed** | **Common Name**  | **IEEE Standard** | **Informal Name** | **Maximum Length** | pairs of wire |
| --------- | ---------------- | ----------------- | ----------------- | ------------------ | ------------- |
| 10 Mbps   | Ethernet         | 802.3i            | 10BASE-T          | 100 m              | 2             |
| 100 Mbps  | Fast Ethernet    | 802.3u            | 100BASE-T         | 100 m              | 2             |
| 1 Gbps    | Gigabit Ethernet | 802.3ab           | 1000BASE-T        | 100 m              | 4             |
| 10 Gbps   | 10 Gig Ethernet  | 802.3an           | 10GBASE-T         | 100 m              | 4             |
|           |                  |                   |                   |                    |               |
### fiber 
| **Informal Name** | **IEEE Standard** | **Speed** | **Cable Type**           | **Maximum Length**                    |
| ----------------- | ----------------- | --------- | ------------------------ | ------------------------------------- |
| **1000BASE-LX**   | 802.3z            | 1 Gbps    | Multimode or Single-Mode | 550 m (MM)<br><br>  <br><br>5 km (SM) |
| **10GBASE-SR**    | 802.3ae           | 10 Gbps   | Multimode                | 400 m                                 |
| **10GBASE-LR**    | 802.3ae           | 10 Gbps   | Single-Mode              | 10 km                                 |
| **10GBASE-ER**    | 802.3ae           | 10 Gbps   | Single-Mode              | 30 km                                 |

#### single vs multi mode 
j

| Feature / Specification       | Single Mode Fiber (SMF)                            | Multimode Fiber (MMF)                                |
| :---------------------------- | :------------------------------------------------- | :--------------------------------------------------- |
| **Common Standards**          | OS1, OS2                                           | OM1, OM2, OM3, OM4, OM5                              |
| **Core / Cladding Diameter**  | 9 / 125 µm                                         | 50 / 125 µm or 62.5 / 125 µm                         |
| **Light Source**              | Laser (1310 nm / 1550 nm)                          | VCSEL / LED (850 nm / 1300 nm)                       |
| **Max Reach (10 Gbps)**       | Up to 10 km (LR) – 40 km+ (ER/ZR)                  | 300 m (OM3) – 400 m (OM4)                            |
| **Max Reach (100 Gbps)**      | Up to 10 km (LR4) – 40 km+ (ER4)                   | 70 m (OM3) – 100 m (OM4/OM5)                         |
| **Signal Degradation Factor** | Attenuation (chromatic dispersion over long runs)  | Modal dispersion (light pulses overlap/spread out)   |
| **Bandwidth Capacity**        | Practically unlimited (high capacity per strand)   | Limited by distance and modal bandwidth              |
| **Cable / Patch Cord Cost**   | Lower per meter                                    | Slightly higher per meter                            |
| **Transceiver Optic Cost**    | Higher (requires precise lasers)                   | Lower (uses cheaper VCSEL optics)                    |
| **Standard Jacket Color**     | Yellow (OS1 / OS2)                                 | Aqua (OM3/OM4), Erika Violet (OM5), Orange (OM1/OM2) |
| **Typical Transceiver Types** | 10GBASE-LR, 100GBASE-LR4, 100GBASE-DR              | 10GBASE-SR, 40GBASE-SR4, 100GBASE-SR4                |
| **Primary Use Cases**         | WAN, MAN, ISP / Telco, long-range campus backbones | Data center racks, intra-building runs, patch panels |

### 802.3i and 802.3u

| **Device Type** | **Transmit (Tx) Pins** | **Receive (Rx) Pins** |
| --------------- | ---------------------- | --------------------- |
| Router          | 1 and 2                | 3 and 6               |
| Firewall        | 1 and 2                | 3 and 6               |
| PC              | 1 and 2                | 3 and 6               |
| SwitchZ         | 3 and 6                | 1 and 2               |


