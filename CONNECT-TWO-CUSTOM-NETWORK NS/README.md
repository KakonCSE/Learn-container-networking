# CONNECT TWO CUSTOM NETWORK NS

#Setting up Virtual Network between Namespaces

#This guide outlines the steps to create two namespaces named blue-namespace and lemon-namespace, and establish a virtual Ethernet network between them using veth interfaces. The goal is to enable communication between the namespaces and allow them to ping each

#Prerequisites:
1.Linux operating system
2.Root or sudo access
3.Packages
```
kakon@DevOps:~$ sudo apt update //(local package update)
kakon@DevOps:~$sudo apt upgrade -y //(installed package upgrade)
kakon@DevOps:~$ sudo apt install iproute2
kakon@DevOps:~$ udo apt install net-tools
```
### First Step:

1. Enable IP forwarding in the Linux kernel:
   
```
kakon@DevOps:~$ sudo sysctl -w net.ipv4.ip_forward=1
```

#Explain:

(`net.ipv4.ip_forward` is a kernel parameter that controls whether your Linux system is allowed to forward packets between different network interfaces, i.e., act as a router.)

#This step enables IP forwarding in the Linux kernel, allowing the namespaces to communicate with each other.

### Second Step:

2. Create namespaces:
   
```
kakon@DevOps:~$ sudo ip netns add blue-namespace
kakon@DevOps:~$ sudo ip netns add lemon-namespace
```

#This step creates two namespaces named `blue-namespace` and `lemon-namespace`.

### Third Step:

3. Create the virtual Ethernet link pair:
   
``` 
kakon@DevOps:~$ sudo ip link add `veth-blue` type veth peer name `veth-lemon`
```

#This command creates a virtual Ethernet link pair consisting of veth-blue and veth-lemon at root namespace.
In order to verify, `run sudo ip link list`
   
#Expected Output:

```
6: veth-lemon@veth-blue: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 22:21:fc:9e:d0:2b brd ff:ff:ff:ff:ff:ff
7: veth-blue@veth-lemon: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 2e:34:8e:0c:1c:6e brd ff:ff:ff:ff:ff:ff
```

### Four Step:

4. Set the cable as NIC

```
kakon@DevOps:~$ sudo ip link set veth-blue netns blue-namespace
kakon@DevOps:~$ sudo ip link set veth-lemon netns lemon-namespace
```
#This command acts as NIC link pair consisting of `veth-blue` and `veth-lemon`.

#To verify run sudo ip netns exec `blue-namespace` ip link and sudo ip netns exec `lemon-namespace` ip link

#Expected Output from lemon-namespace

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
7: veth-blue@if6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 2e:34:8e:0c:1c:6e brd ff:ff:ff:ff:ff:ff link-netns lemon-namespace
```

#Expected Output from blue-namespace

```
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
6: veth-lemon@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 22:21:fc:9e:d0:2b brd ff:ff:ff:ff:ff:ff link-netns blue-namespace
```

#But as we see, interface has been created but it's DOWN and has no ip. Now assign a ip address and turn it `UP`.

### Five Step:

5. Assign IP Addresses to the Interfaces

```
kakon@DevOps:~$ sudo ip netns exec blue-namespace ip addr add 192.168.0.1/24 dev veth-blue
kakon@DevOps:~$ sudo ip netns exec lemon-namespace ip addr add 192.168.0.2/24 dev veth-lemon
```

#In this step, IP addresses are assigned to the veth-blue interface in the blue-namespace and to the veth-lemon interface in the lemon-namespace.

#To verify run `sudo ip netns exec blue-namespace ip addr` and `udo ip netns exec lemon-namespace ip addr`

#Expected Output from blue-namespace

```
7: veth-blue@if6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 2e:34:8e:0c:1c:6e brd ff:ff:ff:ff:ff:ff link-netns lemon-namespace
    inet 192.168.0.1/24 scope global veth-blue
       valid_lft forever preferred_lft forever
```

#Expected Output from lemon-namespace

```
6: veth-lemon@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 22:21:fc:9e:d0:2b brd ff:ff:ff:ff:ff:ff link-netns blue-namespace
    inet 192.168.0.2/24 scope global veth-lemon
       valid_lft forever preferred_lft forever
```

### Six Step:

6. Set the Interfaces Up

```
kakon@DevOps:~$ sudo ip netns exec blue-namespace ip link set veth-blue up
kakon@DevOps:~$ sudo ip netns exec lemon-namespace ip link set veth-lemon up
```

#These commands set the veth-blue and veth-lemon interfaces up, enabling them to transmit and receive data.

Now run again `sudo ip netns exec blue-namespace ip link` and `sudo ip netns exec lemon-namespace ip link` to verify

#Expected Output from lemon-namespace

```
7: veth-blue@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2e:34:8e:0c:1c:6e brd ff:ff:ff:ff:ff:ff link-netns lemon-namespace
```

#Expected Output from blue-namespace

```
6: veth-lemon@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 22:21:fc:9e:d0:2b brd ff:ff:ff:ff:ff:ff link-netns blue-namespace
```

### Seven Step:

7. Set Default Routes
   
```
kakon@DevOps:~$ sudo ip netns exec blue-namespace ip route add default via 192.168.0.1 dev veth-blue
kakon@DevOps:~$ sudo ip netns exec lemon-namespace ip route add default via 192.168.0.2 dev veth-lemon
```

#These commands set the default routes within each namespace, allowing them to route network traffic.

In order to verify run `sudo ip netns exec blue-namespace ip route` and `sudo ip netns exec lemon-namespace ip route`

#Expected Output from blue-namespace

```
default via 192.168.0.1 dev veth-blue 
192.168.0.0/24 dev veth-blue proto kernel scope link src 192.168.0.1
```

#Expected Output from lemon-namespace

```
default via 192.168.0.2 dev veth-lemon 
192.168.0.0/24 dev veth-lemon proto kernel scope link src 192.168.0.2
```

#In addition, the route command in the context of the ip netns exec allows you to view the routing table of a specific network namespace. The routing table contains information about how network traffic should be forwarded or delivered.

#To view the routing table of the lemon-namespace, you can execute the following command:

```
kakon@DevOps:~$  sudo ip netns exec lemon-namespace route
```

#Output
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.0.1     0.0.0.0         UG    0      0        0 veth-blue
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 veth-blue
```
### Eight Step:

8. Test Connectivity
```
kakon@DevOps:~$ sudo ip netns exec blue-namespace ping 192.168.0.2
kakon@DevOps:~$ sudo ip netns exec lemon-namespace ping 192.168.0.1
```
#Use these commands to test the connectivity between the namespaces by pinging each other's IP address.

#Expected Output from blue-namespace

```
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=64 time=0.063 ms
64 bytes from 192.168.0.2: icmp_seq=4 ttl=64 time=0.064 ms
64 bytes from 192.168.0.2: icmp_seq=5 ttl=64 time=0.063 ms

--- 192.168.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4099ms
rtt min/avg/max/mdev = 0.024/0.056/0.069/0.016 ms
```

#Expected Output from lemon-namespace

```
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.033 ms
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=0.072 ms
64 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=0.071 ms
64 bytes from 192.168.0.1: icmp_seq=4 ttl=64 time=0.074 ms
64 bytes from 192.168.0.1: icmp_seq=5 ttl=64 time=0.070 ms

--- 192.168.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4099ms
rtt min/avg/max/mdev = 0.033/0.064/0.074/0.015 ms
```

#Furthermore, the arp command in the context of the ip netns exec allows you to view the ARP cache of a specific network namespace. The ARP cache contains mappings of IP addresses to MAC addresses.

#To view the ARP cache of the blue-namespace, you can execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec blue-namespace arp
Output
Address                  HWtype  HWaddress           Flags Mask            Iface
127.0.0.53                       (incomplete)                              veth-blue
192.168.0.2              ether   22:21:fc:9e:d0:2b   C                     veth-blue
```

#To view the ARP cache of the lemon-namespace, you can execute the following command:

```
kakon@DevOps:~$ sudo ip netns exec lemon-namespace arp

Output

Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.0.1              ether   2e:34:8e:0c:1c:6e   C                     veth-lemon
127.0.0.53                       (incomplete)                              veth-lemon
```

