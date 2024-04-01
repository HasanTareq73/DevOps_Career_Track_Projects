 # Multiple NameSpace:

# Module 2 networking projects:

Make two network namespaces using 'red' and 'green' names, connect them with a bridge, and check connectivity. You have to successfully ping Google's public IP from those network namespaces. 

![Untitled](https://github.com/HasanTareq73/DevOps_Career_Track_Projects/blob/main/Project-2.png)

### **Introduction**

In this blog, we’ll explore how to create two namespaces and connect them using virtual Ethernet (veth) pairs in a virtual machine (VM). For this demonstration, I’m using a Linux machine (VM).

### Step-01:   Create NameSpace

```bash
ubuntu@ip-172-31-88-100:~$ sudo ip netns add Red
ubuntu@ip-172-31-88-100:~$ sudo ip netns add Green

### Check NameSpace List ###
ubuntu@ip-172-31-88-100:~$ sudo ip netns list
```

### Step-02: Create Bridge and Turn UP

```bash
ubuntu@ip-172-31-88-100:~$ sudo ip link add name Bridge-SW type bridge
ubuntu@ip-172-31-88-100:~$ sudo ip link set Bridge-SW up

### Check Link list
ubuntu@ip-172-31-88-100:~$ sudo ip link list

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
link/ether 12:7e:a1:ca:f0:8d brd ff:ff:ff:ff:ff:ff
3: Bridge-SW: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether ea:96:54:d1:92:ce brd ff:ff:ff:ff:ff:ff
```

### Step-03: Create Virtual Ethernet (veth)

```bash
ubuntu@ip-172-31-88-100:~$ sudo ip link add veth-red type veth peer name br-veth-red
ubuntu@ip-172-31-88-100:~$ sudo ip link add veth-green type veth peer name br-veth-green

```

### Step-04: Virtual Ethernet (veth) Connect to NameSpace and Bridge

```bash
ubuntu@ip-172-31-88-100:~$ sudo ip link set veth-red netns Red
ubuntu@ip-172-31-88-100:~$ sudo ip link set veth-green netns Green

ubuntu@ip-172-31-88-100:~$ sudo ip link set br-veth-red master Bridge-SW
ubuntu@ip-172-31-88-100:~$ sudo ip link set br-veth-green master Bridge-SW

### Need to turn up bridge interface ###
ubuntu@ip-172-31-88-100:~$ sudo ip link set br-veth-red up
ubuntu@ip-172-31-88-100:~$ sudo ip link set br-veth-green up
```

### Step-05: IP address assign in NameSpace and Bridge

```bash
## IP address Assign in Bridge interface
ubuntu@ip-172-31-88-100:~$ sudo ip addr add 192.168.200.1/24 dev Bridge-SW

## IP address Assign in Red NameSpace interface
ubuntu@ip-172-31-88-100:~$ sudo ip netns exec Red
root@ip-172-31-88-100:/home/ubuntu# sudo ip addr add 192.168.200.10/24 dev veth-red

## IP address Assign in Green NameSpace interface
ubuntu@ip-172-31-88-100:~$ sudo ip netns exec Green
root@ip-172-31-88-100:/home/ubuntu# sudo ip addr add 192.168.200.10/24 dev veth-green

```

### Step-06: Need to turn on IP forwarding

```bash
ubuntu@ip-172-31-88-100:~$ sudo sysctl -w net.ipv4.ip_forward=1
```

### Step-07: Need to add default route in all NameSpace

```bash
ubuntu@ip-172-31-88-100:~$ sudo ip netns exec Red
root@ip-172-31-88-100:/home/ubuntu$ sudo ip route add default via veth-red

ubuntu@ip-172-31-88-100:~$ sudo ip netns exec Green
root@ip-172-31-88-100:/home/ubuntu$ sudo ip route add default via veth-green
```

### Step-08: Need to configure NAT for NameSpace
