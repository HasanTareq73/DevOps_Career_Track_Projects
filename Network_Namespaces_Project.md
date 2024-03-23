# What is Namespace?

Network namespace is a Linux kernel feature that creates **an isolated environment for networking**, with its own network interfaces, routing table, firewall rules, and NAT rules. This enables network administrators to create virtual networks with increased security and flexibility. It is used in **containerization technologies** like Docker and Kubernetes to create isolated network environments for containerized applications.

# Connect Network Namespaces with a Virtual Ethernet Pair:

**1. Create network namespaces:** 

```bash
	ip netns add NameSpace1
	ip netns add NameSpace2
```

1. **Create a virtual ether pair:** 
    
    ```bash
    ip link add veth1 type veth peer name veth2
    ```
    
2. **Attach the veth pair to the corresponding network namespace**
    
    ```bash
    ip link set veth1 netns NameSpace1
    ip link set veth2 netns NameSpace2
    ```
    
3. **Bring all interfaces up**
    
    ```bash
    ip netns exec NameSpace1 ip link set veth1 up
    ip netns exec NameSpace2 ip link set veth2 up
    ```
    
4. **Configure IP for namespaces**
    
    ```bash
    ip netns exec NameSpace1 ip addr add 192.168.10.1/30 dev veth1
    ip netns exec NameSpace2 ip addr add 192.168.10.2/30 dev veth2
    ```
    
5. **Verify the connectivity between namespaces**
    
    ```bash
    //Ping the IP address of the other namespace.
    
    root@ubuntu-01:~# ip netns exec NameSpace1 ping 192.168.10.2
    PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
    64 bytes from 192.168.10.2: icmp_seq=1 ttl=64 time=0.114 ms
    64 bytes from 192.168.10.2: icmp_seq=2 ttl=64 time=0.067 ms
    64 bytes from 192.168.10.2: icmp_seq=3 ttl=64 time=0.071 ms
    64 bytes from 192.168.10.2: icmp_seq=4 ttl=64 time=0.068 ms
    
    root@ubuntu-01:~# ip netns exec NameSpace2 ping 192.168.10.1
    PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
    64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.010 ms
    64 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=0.068 ms
    64 bytes from 192.168.10.1: icmp_seq=3 ttl=64 time=0.070 ms
    64 bytes from 192.168.10.1: icmp_seq=4 ttl=64 time=0.059 ms
    
    // How to check Namespace list
    root@ubuntu-01:~# ip netns
    NameSpace2 (id: 1)
    NameSpace1 (id: 0)
    
    //How to check namespace IP addressing info
    
    root@ubuntu-01:~# ip netns exec NameSpace1 ip add
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    6: veth1@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 62:04:e3:65:95:5d brd ff:ff:ff:ff:ff:ff link-netns NameSpace2
        inet 192.168.10.1/30 scope global veth1
           valid_lft forever preferred_lft forever
        inet6 fe80::6004:e3ff:fe65:955d/64 scope link 
           valid_lft forever preferred_lft forever
    
    // How Check Namespace Virtual ether information
    
    root@ubuntu-01:~# ip netns exec NameSpace1 ip link
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    6: veth1@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
        link/ether 62:04:e3:65:95:5d brd ff:ff:ff:ff:ff:ff link-netns NameSpace2
    ```
    

# **Connect Network Namespaces Using a Linux Bridge:**

A bridge is a software device that **connects multiple network interfaces together**, allowing them to communicate with each other.

1. **Create network namespaces**
    
    ```bash
    ip netns add BackendApp
    ip netns add Database
    ```
    
2. **Create a Linux bridge:** 
    
    ```bash
    ip link add vBridge-link type bridge
    ```
    
3. **Bring the bridge up:** 
    
    ```bash
    ip link set dev vBridge-link up
    ```
    
4. **Create Virtual-ether pairs:** 
    
    ```bash
    ip link add  veth_ns1 veth peer name veth_ns1_vBridge
    ip link add  veth_ns2 veth peer name veth_ns2_vBridge
    ```
    
5. **Attach veth pairs to the corresponding network namespace and the bridge:** 
    
    ```bash
    ip link add  veth1 type veth peer name veth_ns1_vBridge
    ip link add  veth2 type veth peer name veth_ns2_vBridge
    ```
    
6. **Attach veth pairs to the corresponding network namespace and the bridge:** 
    
    ```bash
    ip link set  veth1 netns BackendApp
    ip link set  veth_ns1_vBridge master vBridge-link
    
    ip link set  veth2 netns Database
    ip link set  veth_ns2_vBridge master vBridge-link
    ```
    
7. **Bring all interfaces up:** 
    
    ```bash
    ip netns exec BackendApp ip link set veth1 up
    ip link set veth_ns1_vBridge up
    
    ip netns exec Database ip link set veth2 up
    ip link set veth_ns2_vBridge up
    ```
    
8. **Configure IP for namespaces:** 
    
    ```bash
    ip netns exec BackendApp ip addr add 10.1.1.1/24 dev veth1
    ip netns exec Database ip addr add 10.1.1.2/24 dev veth2
    ```
    
9. **Test connectivity between the namespaces:** 
    
    ```bash
    ip netns exec BackendApp ping 10.1.1.2
    PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
    64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.099 ms
    64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.096 ms
    
    root@ubuntu-01:~# ip netns exec Database ping 10.1.1.1
    PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
    64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.060 ms
    64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=0.096 ms
    ```
    
10. **Configure the bridge interface with an IP address:** 
    
    ```bash
    ip addr add 10.1.1.4/24 dev vBridge-link
    ```
    
11. **Test connectivity between host and namespaces:** 
    
    ```bash
    root@ubuntu-01:~# ping 10.1.1.1
    PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
    64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.049 ms
    64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=0.184 ms
    64 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=0.080 ms
    
    root@ubuntu-01:~# ping 10.1.1.2
    PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
    64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.049 ms
    64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.077 ms
    64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.083 ms
    64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.071 ms
    ```
    

# **Commands**

1. List existing network namespaces on the system

```bash
$ ip netns
```

2. Delete a network namespace

```bash
$ ip netns delete <namespace>
```

3. Delete a network interface

```bash
$ ip link delete <network_interface>
```