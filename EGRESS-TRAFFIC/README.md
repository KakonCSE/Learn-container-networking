# EGRESS-TRAFFIC

#Connecting a container network namespace to root network namespace

#Let's create a custom network namespace `ns0` and a bridge `br0`. In Linux networking, a bridge is a virtual network device that connects multiple network interfaces, allowing them to function as a single logical network.

```
kakon@DevOps:~$ sudo ip netns add ns0
kakon@DevOps:~$ sudo ip link add br0 type bridge
kakon@DevOps:~$ ip link
9: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode D                                                                                        EFAULT group default qlen 1000
    link/ether 5a:9b:78:51:4e:01 brd ff:ff:ff:ff:ff:ff
```
#Configure a bridge interface

#A new device, the `br0` bridge interface, has been created, but it's now in a `DOWN` state. Let's assign ip address and turn it into `UP` state.
```
kakon@DevOps:~$ sudo ip link set br0 up
kakon@DevOps:~$ sudo ip addr add 192.168.0.1/16 dev br0
kakon@DevOps:~$ ip addr show dev br0
9: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 5a:9b:78:51:4e:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/16 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::589b:78ff:fe51:4e01/64 scope link
       valid_lft forever preferred_lft forever
```
#Now let's verify whether br0 is able to receive the packet or not.
```
kakon@DevOps:~$ ping 192.168.0.1 -c 3
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=0.080 ms
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=0.059 ms

--- 192.168.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2043ms
rtt min/avg/max/mdev = 0.049/0.062/0.080/0.012 ms
```
#Configure virtual ethernet cable

#It's time to set up a virtual Ethernet cable. One cable hand will be configured as a nic card in the `ns0` namespace, while the other hand will be configured in the `br0` interface.
```
kakon@DevOps:~$ sudo ip link add veth0 type veth peer name ceth0
kakon@DevOps:~$ sudo ip netns exec ns0 bash
root@DevOps:/home/kakon# ip link set ceth0 netns ns0
root@DevOps:/home/kakon# exit
kakon@DevOps:~$ sudo ip link set veth0 master br0
```
#Both end of this cable is now in `DOWN` state. Let's turn into `UP` state
```
root@DevOps:/home/kakon# ip netns exec ns0 ip link set ceth0 up
kakon@DevOps:~$ sudo ip link set veth0 up
```
#Configure ns0 namespace

#We need to assign an ip address to  `ceth0` and turn loopback interface into `UP` state.
```
root@DevOps:/home/kakon# ip link set lo up
root@DevOps:/home/kakon# ip addr add 192.168.0.2/16 dev ceth0
```
#Namespace ns0 to root ns Communication

#Let's check the Ip address assigned to primary ethernet interface of host machine.

#ip addr show from root ns
```
2: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:a5:57:0e brd ff:ff:ff:ff:ff:ff
    altname enp2s2
    inet 10.56.80.102/30 brd 10.56.80.103 scope global noprefixroute ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fea5:570e/64 scope link
       valid_lft forever preferred_lft forever
```
#ip addr show from ns0
```
21: ceth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 1a:d6:e0:af:8b:5d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.2/16 scope global ceth0
       valid_lft forever preferred_lft forever
    inet6 fe80::18d6:e0ff:feaf:8b5d/64 scope link
       valid_lft forever preferred_lft forever
```

#Now, ping to this ip address from `ns0`
```
kakon@DevOps:~$ sudo ip netns exec ns0 bash
root@DevOps:/home/kakon# ping 10.56.80.102
ping: connect: Network is unreachable
```
#It says network in unreachable. So, something is not okay. Let's check the route table.
```
root@DevOps:/home/kakon# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.0.1     0.0.0.0         UG    0      0        0 ceth0
192.168.0.0     0.0.0.0         255.255.0.0     U     0      0        0 ceth0
```
#Now we are good to go! Let's ping again.
```
root@DevOps:/home/kakon# ping 10.56.80.102 -c 5
PING 10.56.80.102 (10.56.80.102) 56(84) bytes of data.
64 bytes from 10.56.80.102: icmp_seq=1 ttl=64 time=0.267 ms
64 bytes from 10.56.80.102: icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from 10.56.80.102: icmp_seq=3 ttl=64 time=0.055 ms
64 bytes from 10.56.80.102: icmp_seq=4 ttl=64 time=0.224 ms
64 bytes from 10.56.80.102: icmp_seq=5 ttl=64 time=0.095 ms

--- 10.56.80.102 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4098ms
rtt min/avg/max/mdev = 0.055/0.143/0.267/0.085 ms
```
#`Ping 8.8.8.8 from ns0` [ egress traffic ]

#We have come so far. Now we can ping our root ns or primary ethernet interface from custom namesapce. But can we ping outside the world?

#Let's ping to 8.8.8.8.
```
kakon@DevOps:~$ sudo ip netns exec ns0 bash
root@DevOps:/home/kakon# ping 8.8.8.8 -c 5
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4076ms
```
#Okay, it does not seem that the network is unreachable. It's something else. Maybe somewhere, packets have stuck. Let's dig it out. We can do it using the Linux utility `tcpdump`. We will check two interfaces. One is `br0`, and the other is `ens3` (in the case of my device). 

#Here we can see that packets are coming to those interfaces but are still stuck in ns0. But why?

#Root Cause
See! The source IP address, 192.168.0.2, is attempting to connect to Google DNS 8.8.8.8 using its private IP address. So it's very basic that the outside world can't reach that private IP because they have no idea about that particular private IP address. That's why packets are able to reach Google DNS, but we are not getting any replies from 8.8.8.8.

#Solution
We must somehow change the private IP address to a public address in order to sort out this issue with the help of NAT (Network Translation Address). So, a SNAT (source NAT) rule must be added to the IP table in order to modify the POSTROUTING chain.

###From root-ns
```
kakon@DevOps:~$ sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16  -j MASQUERADE
```
###Enable IP forwarding
```
kakon@DevOps:~$ sudo  sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
```
###For Permanent
```
kakon@DevOps:~$ vim /etc/sysctl.conf
```
#Add or uncomment the following line
```
net.ipv4.ip_forward=1
```
###Then Apply
```
kakon@DevOps:~$ sudo sysctl -p
net.ipv4.ip_forward = 1
```
#Ping from custom namespace `ns0`
```
kakon@DevOps:~$ sudo ip netns exec ns0 ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=30.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=30.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=115 time=30.6 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=115 time=30.6 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=115 time=30.6 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=115 time=30.7 ms
--- 8.8.8.8 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5010ms
rtt min/avg/max/mdev = 30.564/30.654/30.796/0.079 ms
```

