# FIB (Forward Information Base)

#How FIB network architecture generally works:

1.The Forwarding Information Base (FIB) is a table used by routers to determine packet forwarding.
2.It contains mappings of destination network addresses to the next-hop router or interface.

#Populating the FIB:

1.FIB entries are populated through routing protocols such as OSPF, RIP, and BGP.
2.These protocols exchange routing information among routers to build and update the FIB.

#Forwarding Decisions:

1.When a router receives an incoming packet, it examines the destination IP address.
2.The router looks up the destination address in its FIB.
3.If a matching entry is found in the FIB, the router forwards the packet based on the next-hop information specified in the FIB entry.
4.If no matching entry is found, the router typically either drops the packet or forwards it to a default route if configured.

Proposed network topology:
