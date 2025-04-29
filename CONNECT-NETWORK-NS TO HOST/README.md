#CONNECT-NETWORK-NS-TO-HOST

#Connecting a container to host using virtual Ethernet cable



#Let's create a custom namespace using ip netns add utiliy.
```
kakon@DevOps:~$ sudo ip netns add red
kakon@DevOps:~$ sudo ip netns list
red
kakon
```
#From the root network namespace, let's create a veth cable:
```
sudo ip link add veth-red type veth peer name veth-host
```
#just created a pair of interconnected virtual Ethernet devices. Both veth-red and veth-host lies inside the root ns.
```
kakon@DevOps:~$ ip link list
19: veth-host@veth-red: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fe:6b:51:67:6e:9c brd ff:ff:ff:ff:ff:ff
20: veth-red@veth-host: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fa:e2:49:eb:32:56 brd ff:ff:ff:ff:ff:ff
```
#To connect the root namespace with the red namespace, we need to keep one end of the cable in the root namespace and move another one end into the red ns.

kakon@DevOps:~$ sudo ip link set veth-red netns red

This moves one end of the veth pair (veth-red) into the "red" namespace. The other end (veth-host) remains in the default network namespace.Now, let's configure IP Addresses to both end of this veth cable and once we turn up the interfaces, the peer device will instantly display any packet that appears on one of the devices.

This moves one end of the veth pair (veth-red) into the "red" namespace. The other end (veth-host) remains in the default network namespace.

Now, let's configure IP Addresses to both end of this veth cable and once we turn up the interfaces, the peer device will instantly display any packet that appears on one of the devices.

In the "red" namespace:

sudo ip netns exec red ip addr add 192.168.1.1/24 dev veth-red
sudo ip netns exec red ip link set veth-red up



On the host side:

sudo ip addr add 192.168.1.2/24 dev veth-host
sudo ip link set veth-host up




The virtual ethernet pair is now ready. A route is needed to add on the host to direct traffic destined for 192.168.1.1 through the veth-host interface.

sudo ip route add 192.168.1.1 dev veth-host



Why Add a Route?
Without adding the route, the system doesn't know how to reach 192.168.1.1. When we ping 192.168.1.1, the system checks its routing table to determine where to send the ping packets. If there's no specific route for 192.168.1.1, it won't know which interface to use.

By adding the route, we are explicitly telling the system that to reach 192.168.1.1, it should send the traffic through the veth-host interface, which is part of the veth pair connected to the "red" namespace.

Test connectivity
Let's try to ping the red ns from the veth-host interface:

ping 192.168.1.1 -c 3



Again, let's try to ping the veth-host interface from the red namespace:

sudo ip netns exec red bash
ping 192.168.1.2 -c 3







