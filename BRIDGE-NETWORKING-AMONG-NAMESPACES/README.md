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

#Here, the "UP" flag indicates that the interface is enabled and operational, while the "DOWN" state indicates that the interface is currently inactive or not functioning as there is no any physical connectivity of the interface right now.

###Step 2: Assign an IP address to the bridge interface 'v-net':

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


###Step 2: Assign an IP address to the bridge interface 'v-net':

