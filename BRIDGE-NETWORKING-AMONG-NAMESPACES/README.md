# BRIDGE NETWORKING AMONG NAMESPACES

#Setting up Linux Bridge Network among Namespaces

#This guide outlines the steps to create three namespaces named blue-ns, gray-ns and lime-ns. Then establish a linux bridge network among them using veth interfaces. The goal is to enable communication among the namespaces and allow them to ping each other.

#Prerequisites

1.Linux operating system
2.Root or sudo access
3.Packages

```
kakon@DevOps:~$ sudo apt update
kakon@DevOps:~$ sudo apt upgrade -y
kakon@DevOps:~$ sudo apt install iproute2
kakon@DevOps:~$ sudo apt install net-tools
```

###Step 1:

Create a Linux bridge:
```
kakon@DevOps:~$ sudo ip link add dev v-net type bridge
```

#Lets run sudo ip link	 show to check and the expected output might look like

```
12: v-net: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d2:22:49:28:a4:c8 brd ff:ff:ff:ff:ff:ff
```

#But here v-net is now in down state. So lets turn into up.

```
kakon@DevOps:~$ sudo ip link set dev v-net up
12: v-net: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether d2:22:49:28:a4:c8 brd ff:ff:ff:ff:ff:ff
```

#Here, the `UP` flag indicates that the interface is enabled and operational, while the `DOWN` state indicates that the interface is currently inactive or not functioning as there is no any physical connectivity of the interface right now.

###Step 2: Assign an IP address to the bridge interface `v-net`:

```
sudo ip address add 10.0.0.1/24 dev v-net
```

#Run sudo ip addr show dev v-net

```
12: v-net: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether d2:22:49:28:a4:c8 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 scope global v-net
       valid_lft forever preferred_lft forever
```


###Step 3: Create custom namespace
```
kakon@DevOps:~$ sudo ip netns add blue-ns
kakon@DevOps:~$ sudo ip netns add gray-ns
kakon@DevOps:~$ sudo ip netns add lime-ns
```

#Run sudo ip netns list to check the list of namespaces.


###Step 4: Create virtual Ethernet pairs:

```
kakon@DevOps:~$ sudo ip link add veth-blue-ns type veth peer name veth-blue-br
kakon@DevOps:~$ sudo ip link add veth-gray-ns type veth peer name veth-gray-br
kakon@DevOps:~$ sudo ip link add veth-lime-ns type veth peer name veth-lime-br
```

#Each cable now has two ends. Before connecting them to the appropriate bridge and namespaces, run sudo ip link show command to verify if the cables have been successfully created.

#Expected Output

```
13: veth-blue-br@veth-blue-ns: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fa:b8:a7:3c:41:0a brd ff:ff:ff:ff:ff:ff
14: veth-blue-ns@veth-blue-br: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ea:ff:ec:28:f3:aa brd ff:ff:ff:ff:ff:ff
15: veth-gray-br@veth-gray-ns: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether be:2a:cb:68:4d:18 brd ff:ff:ff:ff:ff:ff
16: veth-gray-ns@veth-gray-br: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1a:08:51:d7:ab:95 brd ff:ff:ff:ff:ff:ff
17: veth-lime-br@veth-lime-ns: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 02:9a:fd:3e:85:8a brd ff:ff:ff:ff:ff:ff
18: veth-lime-ns@veth-lime-br: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fe:c0:37:07:25:e4 brd ff:ff:ff:ff:ff:ff

```

###Step 5: Move each end of veth cable to a different namespace:

```
kakon@DevOps:~$ sudo ip link set dev veth-blue-ns netns blue-ns
kakon@DevOps:~$ sudo ip link set dev veth-gray-ns netns gray-ns
kakon@DevOps:~$ sudo ip link set dev veth-lime-ns netns lime-ns
```
#Run sudo ip netns exec <namespace-name> ip link show to verify

###Step 6: Add the other end of the virtual interfaces to the bridge:

```
kakon@DevOps:~$ sudo ip link set dev veth-blue-br master v-net
kakon@DevOps:~$ sudo ip link set dev veth-gray-br master v-net
kakon@DevOps:~$ sudo ip link set dev veth-lime-br master v-net
```

#Again run sudo ip link show command to verify.

###Step 7: Set the bridge interfaces up:

```
kakon@DevOps:~$ sudo ip link set dev veth-blue-br up
kakon@DevOps:~$ sudo ip link set dev veth-gray-br up
kakon@DevOps:~$ sudo ip link set dev veth-lime-br up
```

#Run sudo ip link show and expected output might look like

```
13: veth-blue-br@if14: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue master v-net state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether fa:b8:a7:3c:41:0a brd ff:ff:ff:ff:ff:ff link-netns blue-ns
15: veth-gray-br@if16: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue master v-net state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether be:2a:cb:68:4d:18 brd ff:ff:ff:ff:ff:ff link-netns gray-ns
17: veth-lime-br@if18: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue master v-net state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
    link/ether 02:9a:fd:3e:85:8a brd ff:ff:ff:ff:ff:ff link-netns lime-ns
```

#The `LOWERLAYERDOWN` state typically applies to virtual interfaces that are dependent on another interface or component. If one end is not currently in the 'up' state, it suggests that there may be an issue with the namespaces at that end (in our case).

###Step 8: Set the namespace interfaces up:

```
kakon@DevOps:~$ sudo ip netns exec blue-ns ip link set dev veth-blue-ns up
kakon@DevOps:~$ sudo ip netns exec gray-ns ip link set dev veth-gray-ns up
kakon@DevOps:~$ sudo ip netns exec lime-ns ip link set dev veth-lime-ns up
```

#Now, run sudo ip link show command again and it should show all interfaces are currently in UP state.

###Step 9: Assign IP addresses to the virtual interfaces within each namespace and set the default routes:

#In the blue-ns namespace:

```
kakon@DevOps:~$ sudo ip netns exec blue-ns ip address add 10.0.0.11/24 dev veth-blue-ns
kakon@DevOps:~$ sudo ip netns exec blue-ns ip route add default via 10.0.0.1
```

#In the gray-ns namespace:

```
kakon@DevOps:~$ sudo ip netns exec gray-ns ip address add 10.0.0.21/24 dev veth-gray-ns
kakon@DevOps:~$ sudo ip netns exec gray-ns ip route add default via 10.0.0.1
```

#In the lime-ns namespace:

```
kakon@DevOps:~$ sudo ip netns exec lime-ns ip address add 10.0.0.31/24 dev veth-lime-ns
kakon@DevOps:~$ sudo ip netns exec lime-ns ip route add default via 10.0.0.1
```

###Step 10: Firewall rules:

```
kakon@DevOps:~$ sudo iptables --append FORWARD --in-interface v-net --jump ACCEPT
kakon@DevOps:~$ sudo iptables --append FORWARD --out-interface v-net --jump ACCEPT
```

#These rules enabled traffic to travel across the v-net virtual bridge.These are useful to allow all traffic to pass through the v-net interface without any restrictions.However, keep in mind that using such rules without any filtering can expose your system to potential security risks. But for now we re good to ping!

#Test Connectivity

```
kakon@DevOps:~$ sudo ip netns exec lime-ns ping -c 2 10.0.0.11
```

#Expected Output

```
PING 10.0.0.11 (10.0.0.11) 56(84) bytes of data.
64 bytes from 10.0.0.11: icmp_seq=1 ttl=64 time=0.064 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=64 time=0.169 ms

--- 10.0.0.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.064/0.116/0.169/0.052 ms
```
#Another one:

```
kakon@DevOps:~$ sudo ip netns exec gray-ns ping -c 2 10.0.0.11
```
#Expected Output

```
PING 10.0.0.11 (10.0.0.11) 56(84) bytes of data.
64 bytes from 10.0.0.11: icmp_seq=1 ttl=64 time=0.051 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=64 time=0.112 ms

--- 10.0.0.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1024ms
rtt min/avg/max/mdev = 0.051/0.081/0.112/0.030 ms
```

#Another one:

kakon@DevOps:~$ sudo ip netns exec blue-ns ping -c 2 10.0.0.21

#Expected Output

```
PING 10.0.0.21 (10.0.0.21) 56(84) bytes of data.
64 bytes from 10.0.0.21: icmp_seq=1 ttl=64 time=0.056 ms
64 bytes from 10.0.0.21: icmp_seq=2 ttl=64 time=0.109 ms

--- 10.0.0.21 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1018ms
rtt min/avg/max/mdev = 0.056/0.082/0.109/0.026 ms
```

#Another one:

```
kakon@DevOps:~$ sudo ip netns exec blue-ns ping -c 2 10.0.0.31
```

#Expected Output:

```
PING 10.0.0.31 (10.0.0.31) 56(84) bytes of data.
64 bytes from 10.0.0.31: icmp_seq=1 ttl=64 time=0.073 ms
64 bytes from 10.0.0.31: icmp_seq=2 ttl=64 time=0.127 ms

--- 10.0.0.31 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
rtt min/avg/max/mdev = 0.073/0.100/0.127/0.027 ms
```

#In addition, the `route` command in the context of the `ip netns exec` allows you to view the routing table of a specific network namespace. The routing table contains information about how network traffic should be forwarded or delivered.

#To view the routing table of the blue-ns namespace, execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec blue-ns route

Output:

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.0.0.1        0.0.0.0         UG    0      0        0 veth-blue-ns
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 veth-blue-ns
```

To view the routing table of the gray-ns namespace, execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec gray-ns route

Output

Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.1                 ether   d2:22:49:28:a4:c8   C                     veth-gray-ns
10.0.0.11                ether   ea:ff:ec:28:f3:aa   C                     veth-gray-ns
```

To view the ARP cache of the lime-ns namespace, execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec lime-ns arp

Output:

Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.1                 ether   d2:22:49:28:a4:c8   C                     veth-lime-ns
10.0.0.11                ether   ea:ff:ec:28:f3:aa   C                     veth-lime-ns
```

#To view the ARP cache of the lime-ns namespace, execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec lime-ns arp
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.1                 ether   d2:22:49:28:a4:c8   C                     veth-lime-ns
10.0.0.11                ether   ea:ff:ec:28:f3:aa   C                     veth-lime-ns
```

#Clean Up (optional)
If you want to remove the namespaces and bridge network device run these commands to clean up the setup.

```
kakon@DevOps:~$ sudo ip netns del <namespace>
kakon@DevOps:~$ sudo ip link delete <bridge network name> type bridge
```




