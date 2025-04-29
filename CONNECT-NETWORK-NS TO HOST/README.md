# CONNECT-NETWORK-NS-TO-HOST

#Connecting a container to host using virtual Ethernet cable
<img src="NS to host.png" alt="picture" />


#Let's create a custom namespace using ip netns add utiliy.
```
kakon@DevOps:~$ sudo ip netns add red
kakon@DevOps:~$ sudo ip netns list
red
kakon
```
#From the root network namespace, let's create a veth cable:
```
kakon@DevOps:~$ sudo ip link add veth-red type veth peer name veth-host
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
```
kakon@DevOps:~$ sudo ip link set veth-red netns red
```
<p>This moves one end of the veth pair (veth-red) into the "red" namespace. The other end (veth-host) remains in the default network namespace.Now, let's configure IP Addresses to both end of this veth cable and once we turn up the interfaces, the peer device will instantly display any packet that appears on one of the devices.</p>

#In the "red" namespace:
```
kakon@DevOps:~$ sudo ip netns exec red bash
root@DevOps:/home/kakon# ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
20: veth-red@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fa:e2:49:eb:32:56 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.1/24 scope global veth-red
       valid_lft forever preferred_lft forever
    inet6 fe80::f8e2:49ff:feeb:3256/64 scope link
       valid_lft forever preferred_lft forever
```
#On the host side:
```
kakon@DevOps:~$ ip addr
19: veth-host@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fe:6b:51:67:6e:9c brd ff:ff:ff:ff:ff:ff link-netns red
    inet 192.168.1.2/24 scope global veth-host
       valid_lft forever preferred_lft forever
    inet6 fe80::fc6b:51ff:fe67:6e9c/64 scope link
       valid_lft forever preferred_lft forever
```

#The virtual ethernet pair is now ready. A route is needed to add on the host to direct traffic destined for 192.168.1.1 through the veth-host interface.
```
kakon@DevOps:~$ sudo ip route add 192.168.1.1 dev veth-host
kakon@DevOps:~$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 veth-host
```

#Why Add a Route?
<p>Without adding the route, the system doesn't know how to reach 192.168.1.1. When we ping 192.168.1.1, the system checks its routing table to determine where to send the ping packets. If there's no specific route for 192.168.1.1, it won't know which interface to use.By adding the route, we are explicitly telling the system that to reach 192.168.1.1, it should send the traffic through the veth-host interface, which is part of the veth pair connected to the "red" namespace.<p/>

#Test connectivity
Let's try to ping the red ns from the veth-host interface:
```
kakon@DevOps:~$ ping 192.168.1.1 -c 3
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.188 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.059 ms

--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2034ms
rtt min/avg/max/mdev = 0.058/0.101/0.188/0.061 ms
```

#Again, let's try to ping the veth-host interface from the red namespace:
```
kakon@DevOps:~$ sudo ip netns exec red bash
root@DevOps:/home/kakon# ping 192.168.1.2 -c 3
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.054 ms

--- 192.168.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2051ms
rtt min/avg/max/mdev = 0.054/0.067/0.085/0.012 ms
```







