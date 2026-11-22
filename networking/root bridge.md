#networking 

acts as the main switch in an [[STP]] topology. with other switches calculating the shortest path to it. The __root bridge__ ensures network stability by determining which ports forward and block traffic. It is selected based on the Bridge ID. 


- the __switch__ with the lowest Bridge ID becomes the __root bridge__
- all ports on the __root bridge__ are in a forwarding state, and other switches in the [[topology]] must have a path to the __root bridge__ 
- when a switch is powered on it assumes that it is the root bridge 
	- it only gives up its position if it receives a superior BPDU
- once the topology has converged and there is agreement on the __root bridge__ only the __root bridge__ sends BPDU's
- other [[switches]] in the network will forward these [[STP#BPDU|BPDU's]] to and from the __root bridge__ but they do not generate their own. 

