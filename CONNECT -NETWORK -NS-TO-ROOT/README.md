# CONNECT NETWORK NS TO ROOT

#Connecting a container network namespace to root network namespace

#Let's create a custom network namespace `ns0` and a bridge `br0` In Linux networking, a bridge is a virtual network device that connects multiple network interfaces, allowing them to function as a single logical network.
```
kakon@DevOps:~$ sudo ip netns add ns0
kakon@DevOps:~$ sudo ip link add br0 type bridge
kakon@DevOps:~$ ip netns list
ns0
kakon@DevOps:~$ ip link
9: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 5a:9b:78:51:4e:01 brd ff:ff:ff:ff:ff:ff
```

### First step
```
pip install requirement.txt
cd main
python app.py
```

### Second step
`git clone www.youtube.com`
