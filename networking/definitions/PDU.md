#networking #definition #incomplete 
__Protocol data units__ are the fundimental segments of data specified by a given protocol at each  layer of the [[models#osi|osi model]]. data moving down the stack will __encapsulate__ its payload and up the stack will decapsulate the data. as the relevant information has been read and striped from the data by the protocol that required it.

- Layer 4 (Transport): Segment (for TCP) or Datagram (for UDP)
- Layer 3 (Network): Packet (or Layer 3 PDU)
- Layer 2 (Data Link): Frame
- Layer 1 (Physical): Bits (or symbols transmitted over the medium)
