#networking #definition 

The Control Plane is the "brain" of the network architecture. It decides where traffic should go and determines the best paths.
- Role: It runs routing and switching protocols (such as OSPF, BGP, or spanning-tree) to discover network topology and build routing tables.
- Characteristics: It communicates with control planes on other devices to share network state information. Once a path is calculated, the control plane pushes these forwarding rules down to the data plane.