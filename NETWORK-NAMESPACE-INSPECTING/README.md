#NETWORK-NAMESPACE-INSPECTING

<p>In Linux networking, within the Linux network stack, routes define traffic paths, iptables configures packet filtering, lo is a local loopback interface for testing, and ens34 is the primary Ethernet interface for external connections. Let's inspect the network stack, in short.</p>

<p>Network interfaces allow us to establish communication between a network and a device.</p>

```
kakon@DevOps:~$ ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:a5:57:0e brd ff:ff:ff:ff:ff:ff
    altname enp2s2
```

`lo` is the loopback interface, allowing local network communication within a device without external network involvement. Verify the loopback interface is up

```
kakon@DevOps:~$ ifconfig lo
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2061216  bytes 2072885209 (2.0 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2061216  bytes 2072885209 (2.0 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#A route in networking specifies the path for network traffic from source to destination. View the routing table:

```
kakon@DevOps:~$  ip route show
default via 10.56.80.101 dev ens34 proto static metric 100
10.56.80.100/30 dev ens34 proto kernel scope link src 10.56.80.102 metric 100
```

#iptables is a user-space utility for configuring packet filter rules in the Linux kernel's Netfilter framework. View iptables rules:

kakon@DevOps:~$ sudo iptables -L

#Let's check for `iptable` rules for custom namespace

#Create Custom Network Namespace

```
kakon@DevOps:~$ sudo ip netns add kakon
kakon@DevOps:~$ sudo ip netns list
kakon
```

#Now, entering a network namespace in Linux:

```
kakon@DevOps:~$ sudo ip netns exec kakon bash
root@DevOps:/home/kakon# ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

#The nsenter utility is commonly used to enter into namespaces in Linux, including network namespaces.

```
sudo nsenter --net=/var/run/netns/kakon bash
```
