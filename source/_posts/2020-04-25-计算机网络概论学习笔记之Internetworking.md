---
layout: post
title: 《计算机网络概论》学习笔记整理之Internet working
date: 2020-04-25
tag: 
- 网络
---

这篇笔记主要是关于IP协议的，内容超级多，其中subnet mask和CIDR的关系或是区别感觉不是很明白，但不予深究，仅把理解的记下来，日后碰到再说。

本篇主要包含下面内容：

1. router的作用：用来建立大规模的异质网络
2. IP(Internet Protocol)的作用是用于整个网络中的node和router的沟通使用，有两个特点：
   1. Connectionless model for data deliverying
   2. Best-effort(Unreliable Service)，由于网络中的不可靠因素导致，不可靠因素包括：
      1. packets lost
      2. packets out of order during deliverying
      3. duplicate copies of a packet
      4. packets delay for a long time
3. router：最重要的工作是routing table lookup
4. IP subnetting
   1. subnet mask
   2. subnet number
5. classless Inter-Domain Routing(CIDR)
   1. 合并路由表(routes aggregated to reduce routing table size)
   2. prefix and prefix length
      1. 192.4.16/21 表示8个class c网络
      2. 192.4.16/22 表示4个class c网络
6. DHCP
7. ICMP
8. Distance Vector(routing protocol)，RIP(Routing Information Protocol)使用这种方法
9. Link State(routing protocol)，OSPF(Open Shortest Path First)使用了这种方法

<!-- more -->

---

### Internet Working

1. 什么是internet working

  An arbitrary collection of networks interconnected to provide some sort of host-to-host packet delivery service

2. 什么是IP

  1. 用来构建可扩展的异质网络的重要工具
  2. 在网络中的所有节点上都会运行
  3. 对使用者来看，就好像是一个logical Internet Working

3. IP service model

  1. packet delivery model(类似邮政系统)
     1. connectionless model for data delivery
     2. best-effort delivery(unreliable service)，可能会出现下面几种情况：
        1. packets are lost
        2. packets are delivered out of order
        3. duplicate copies of a packet are delivery
        4. packets can be delayed for a long time
  2. Global Addressing Scheme: Provides a way to identify all hosts in the network

---

### router(layer 3 device)是如何工作的

1. router使用*store and forward*的方式处理封包

  router通过routing protocol一共会维护2张表：
  1. Forwarding Table：从这张表中查找destination IP，对应的下一步去向，目的地是和该router直接相连还是间接相连，然后去下表中查找对应的MAC address
  2. IP/MAC mapping table：拥有该IP的主机或router和其对应的MAC address的映射(真正收封包的是硬件网卡，所以需要知道网卡的MAC address)

  如果在第一张表中没有找到destination IP，这个封包就会从default port出去，继续后面的处理

2. 这种工作方式的router最大的问题是查表的速度

  IP地址共有2^32（40亿）个，如果查表速度不够快，就会在router中累积，出现丢包。现代的router为了提高查表速度，都使用硬件的解决方案，使用专门的芯片，能够实现线速(wire-speed forwarding)

3. Not plug-and-play

  router必须经过设定才能正常工作，无法像switch即插即用

---

### IP封包的组成

  每个网络都有一个MTU(Maxium Transmission Unit)特性，用来表示能传输的单个封包的大小上限

  封包的重要字段：
  1. identification：在切割封包时，用于标识小封包是否属于同一个大封包
  2. Fragment：在封包被切割时，该字段用于记录每一个片段在原封包的什么位置
  3. Protocol：上一层协议，比如TCP或者UDP
  4. Time to Live(TTL)：该处为一个数字，每经过一个router就减1，当减为0时，router就会将其丢弃，防止封包永久存在于网络上
  5. flags：用来标记封包是否允许切割，不能切割的封包有两条选择：
     1. 走另外一条允许大封包通过的路
     2. 无路丢弃

      flags由3个bit组成：
      1. 固定值0
      2. DF：0表示 may fragment，1表示don't fragment
      3. MF：0表示 last fragment，1表示more fragment
  6. Total length：封包大小
  7. Header checksum：由IP Header的内容生成的校验码，校验封包内容在传输过程中是否错误
  8. Source IP Address和Destination IP Address：封包发送方和接收方的IP地址，IPv4版本是32bits，IPv6版本是128bits
  9. Type of Service：8个bit，用来标识这个IP封包的功能

    ```
      |precedence|D|T|R|O|O|
    ```
    
    precedence：3个bit，
    - 111：Network Control
    - 110：Internetwork Control
    - 101：CRITIC/ECP
    - 100：Flash Override
    - 011：Flash
    - 010：Immediate
    - 001：Priority
    - 000：Routine
    
    D：1个bit，表示Delay
    - 0：Normal
    - 1：Low
    
    T：1个bit，表示Through put
    - 0：Normal
    - 1：High
    
    R：1个bit，表示Reliability
    - 0：Normal
    - 1：High

---

### IP Address的特点

1. IPv4 32 bits，每个都是全球唯一的，*用来定位唯一一台host*
2. 层级：*network + host*
3. class A type：network(8) + host(24)
4. class B type: network(16) + host(16)
5. class C type: newwork(24) + host(8)

---

### Intra-LAN and Inter-LAN Communications

<img src="/images/network/InterLanCommunication.png" width="800" alt="InterLanCommunication" />

1. B => Y(Intra LAN)

  send the frame to the destination directly

  ```
    |MAC(Y)|MAC(B)|IP(Y)|IP(B)|IP datagram|
  ```

2. B => A(Inter-LAN)

  1. send the frame to attached Router first
  2. Router will forward to the destination

  ```
    |MAC(R)|MAC(B)|IP(A)|IP(B)|IP Datagram|

    |MAC(A)|MAC(R)|IP(A)|IP(B)|IP Datagram|
  ```

---

### IP Datagram Forwarding

router的每一个port都有1个IP

策略：

  - every datagram contains destination's address
  - if directly connected to destination network, then forward to host
  - if not directed connected to destination network, then forward to some router
  - forwarding table maps network number into next hop
  - each host has a default router
  - each router maintains a forwarding table

---

### IP Fragmentation and Reassembly

1. MTU(Maximum Transmission Unit)

  每一种网络都有一个MTU，表示这种网络可以接受的封包的最大的size

  - Ethernet：1518bytes
  - IEEE802.11 wireless：2312bytes
  - FDDI：4500bytes

  封包的size大过MTU就不能送进这个网络

2. 策略

  - Fragmentation occurs in a router when it receives a datagram that it wants to forward over a network which has MTU < datagram(封包切割发生在router上)
  - Reassembly is done at the receiving host(封包组合发生在host上)
  - All the fragments carry the same identifier(IP的header会被复制不切，切的是内容)
  - Fragments are self-contained datagrams
  - IP does not recover from missing(封包组合不起俩就会被丢弃)

3. 组合

下图中：

- MF：表示该封包后面是否还有封包，0表示没有(就是说该封包就是最后一个)，1表示还有
- offset：表示该封包中第一个byte的位置，0表示该封包位于整个资料的最前面
- MF和offset都为0表示该封包没有被切割过
- 封包组合时，会设置一个时间上限，超时未组合完毕就会被丢弃

<img src="/images/network/reassemblyFrafmentation.png" width="600" alt="reassemblyFrafmentation" />

---

### Router的特征

1. Network Layer Routing

  - Network layer protocol dependent
  - Filter * MAC broadcast and multicast packets *
  - Easy to support mixed media(每一个端口都可以支持不同的媒介)
  - Packet fragmentation and reassembly(router本身具有组合封包的能力，但一般交给host来处理)
  - Filtering on network(IP) address and information(ACL: Access Control List)，也就是说router本身具有防火墙的功能
  - accounting：计费能力，根据封包的流量计费

2. Direct Communication between endpoints and routers

  - Highly configurable and hard to get right
  - Handle speed mismatch(通过buffer来平衡)
  - Congestion control and avoidance，拥挤控制方法包括：
    1. 丢弃封包
    2. 通过router之间的交流，告诉其他的封包送的慢点儿

3. Routing Protocols
   1. Interconnect layer 3 networks and exploit arbitrary topologies
   2. Determing which route to take
   3. Static routing:固定两个IP之间的路线，不受网络状态影响，不受距离远近、网速快慢影响
   4. Dynamic routing protocol support
      1. RIP：Routing Information Protocol
      2. OSPF：Open shortest Path First
   5. Provides reliability with alternate routes
4. Router management
   1. Troubleshooting capabilities

---

### Differences between Bridges and Routers



| Bridges                               | Routers                                                   |
| ------------------------------------- | --------------------------------------------------------- |
| operation at layer 2                  | operation at layer 3                                      |
| protocol independent                  | protocol dependent                                        |
| automatic address learning/filtering  | administration required for address, internet and routers |
| pass MAC multicast/broadcast          | MAC multicast/broadcast can be filtered                   |
| lower cost                            | higher cost                                               |
| no flow/congestion control            | flow/congestion control                                   |
| limited security                      | complex security                                          |
| transparent to end systems            | non-transparent                                           |
| well suited for simple/small networks | for wan\larger networks                                   |
| no frames segmentation/reassembly     | frames segmentation/reassembly                            |
| spanning tree based routing           | optimal routing and load sharing(routing选的路线更好)     |
| plug and play                         | requires central administrator(需要设定才能运作)          |

---

### IP subnetting

- subnet: another level to address/routing hierarchy(在a、b、c三层网络下再加一层子网络)
- subnet masks(用来定义子网络的大小): define variable partition of host part of class A and class B address

  ```
    |network number|host number|
    
    IP地址的层级结构如上，一个class B的ip地址，host unmber由16个bit组成，共有2^16=65536个

    |11111111|11111111|11111111|00000000|

    每个1表示一个mask，上面就是255.255.255.0，这个子网共有2^8=256台host
    submask为255.255.255.128时，有2^7=128台host
    submask为255.255.255.192时，有2^6=64台host
  ```

---

### Subnet Forwarding Algorithm

下面这个表格是子网和下一站的映射


| subnet Number | Subnet Mask     | Next Hop    |
| ------------- | --------------- | ----------- |
| 128.96.34.0   | 255.255.255.128 | Interface 0 |
| 128.96.34.128 | 255.255.255.128 | Interface 1 |
| 128.96.33.0   | 255.255.255.0   | R2          |

```
 D = destination IP address
 for each entry < Subnet Number, Subnet Mask, Next Hop >
 D1 = (Subnet Mask) AND (D)
 if D1 = Subnet Number(每一个子网络都有一个编号)
    if Next Hop is an interface
      deliver datagram directly to destination
    else
      deliver datagram to Next Hop(router)
```

注意：
  1. A default router is used if nothing matches
  2. Not necessary for all ones in subnet mask to be contiguous(mask的1不一定要连续)
  3. subnets not visible from the rest of the Internet(子网络是router内部规划的结果)
  4. Can put multiple subnets on one physical network

---

### Classless Addressing

1. Classless Inter-Domain Routing(CIDR)

  用来解决网络上的2个规模问题：
  1. The growth of backbone routing table as more and more network numbers need to be stored in them
  2. Potential exhaustion of the 32-bit address space

2. CIDR uses aggregate routes(合并路由)
   1. Uses a *single entry* in the forwarding table to tell the router how to reach a lot of different newworks
   2. Breaks the regid(严格的) boundaries between address classes

3. 示例
   1. For example, an AS with 16 class c network numbers
   2. Instead of handling out 16 addresses at random, handle out a block of contiguous class c address
   3. Suppose we assign the class C network numbers from 192.4.16(prefix,可以是2~32bits的任何长度) through 192.4.31
   4. Observe that top 20 bits of all the addresses in this range are the same(11000000 00000100 0001)
   5. Requires to handle out blocks of class c addresses that share a common prefix
   6. The convention is to place a /x after the prefix where x is the prefix length in bits
   7. For example, the 20-bit prefix for all the networks 192.4.16 through 192.4.31 is represented as 192.4.16/20
   8. By contrast, if we wanted to represent a single class c network number 192.4.16, which is 24 bits long,we would write it 192.4.16/24

  ```
    对于class c，host前用24bits表示network
    192.4.16/21时，24-21=3，2^3=8，表示有8个连续的网络
    192.4.16/20时，24-20=4，2^4=16，表示有16个连续的网络
    192.4.16/23时，24-23=1，2^1=2，表示有2个连续的网络
  ```

  这种方式可以让多个entry合并为1个，减小router table的大小

4. longest prefix matching

  比对prefix可能出现的多个结果，比如：

  171.69.10.5

  - 171.69/16
  - 171.69.10/24

  这两种情况都能匹配上，router对这种情况的处理方式就是使用最长的prefix

---

### Address Resolution Protocol(ARP)

1. 用途

  Map IP address(1. destination host 2. next hop router) into physical(MAC) address

2. 过程

  1. table of IP to physical address bindings
  2. broadcast request if IP address not in the table
  3. target machine responds with its physical address
  4. table entries are discarded if not refreshed

3. ARP封包的重要字段
  1. target protocol addr：对方的IP
  2. source protocol addr：问的IP
  3. source hardware addr：自己的MAC
  4. target hardware addr：对方的MAC
  5. operation：是问是答

4. Dynamic Host Configuration Protocol(DHCP)

  一台host要设置如下参数，才能正常工作：
     1. IP
     2. default router
     3. subnet mask
     4. domain name

  1. 关于DHCP
     1. DHCP server is responsible for providing configuration information to host
     2. There is at least one DHCP server for an administrative domain
     3. DHCP server maintains a pool of available address(这种方式允许动态分配时可以对IP的时效性做控制)
  2. 过程
     1. Newly booted on attached host sends DHCP DISCOVER message to a special IP address(255.255.255.255)
     2. DHCP relay(中继设备) agent unicasts the message to DHCP server and waits for the reponse

---

### Internet Control Message Protocol(ICMP)

1. Defines a collection of error messages that are sent back to the source host whenever a router or host is unable to process an IP datagram successfully，下面这些场景都会导致router发出ICMP封包
   1. Destination host unreachable due to link/nodd failure
   2. Reassembly process failed
   3. TTL(time-to-live) had reached to(so dataframs don't cycle forever)
   4. IP header checksum failed

2. ICMP-Redirect
   1. From router to a source host
   2. with a better route information

---

### Routing Protocol

router有两项重要的工作：forwarding和routing，同时对应的也会构建两张表：forwarding table和routing table

1. Forwarding: to select an output port based on destination address and routing table
2. Routing: process to build the routing table

  find the lowest-cost path between any two nodes，通过routing protocol，每一个router都会获得整个网络的拓扑结构

3. Forwarding table
   1. Used when a packet is being forwarded
   2. An entry in the forwarding table contains the mapping from a network umber(哪一个网络) to an outgoing interface(哪一个port) and some MAC information, such as Ethernet Address of the next hop

| prefix/length | Interface | MAC address      |
| ------------- | --------- | ---------------- |
| 140.114/16    | 0         | 8:0:2c:e3:b:2:20 |

4. Routing table
   1. Built by the routing algorithm
   2. Generally contains mapping from network numbers to next hops

| cost | prefix/Length | Next Hop     |
| ---- | ------------- | ------------ |
| 2    | 140.114/16    | 171.34.45.12 |

每一个routing只需记住下一站，不需要记住全程

5. Distance Vector Protocol(一种分布式动态routing protocol)
   1. Each node constructs a one dimensinal array(a vector) containing the "distances"(costs) to all other nodes and distributes that vector to its immediate neightbours
   2. Assume that each node knows the cost of the link to each of its directly connected neightbours
   3. Every T seconds each router sends its routing table to its neighbours
   4. Each router then updates its routing table based on the new information
   5. problems include:
      1. fast response to good news
      2. slow response to bad news
      3. too much messages to update

  在真实网络中，经过router的数量的最大值是16(可以认为是count-to-infinity中的infinity)。Distance Vector Procol中存在的最大的一个问题是：count-to-infinity problem，简单来说就是相邻的两个router循环向对方发送routing table：`A <=> B`，解决办法如下：

  1. split horizon: when a node sends a routing update to its neightbours, it does not send those routes it learned from each neighbours back to that neightbours
  2. split horizon with poizon reverse: B actially sends that back route to A, but it puts negative information(route(0,oo)) in the route to ensure that A will not eventually use B to get to other node

6. Link State Protocol(另一种分布式动态routing protocol)

  1. 策略：send to all nodes(not just neighbours) information about directly connected links(not entire routing table)
  2. LSP封包(Link State Packet)，执行上述过程的时候借助该类封包，包含：
     1. ID of the node that created the LSP
     2. cost of link to each directly connected neighbours
     3. Sequence number(SEQNO)(link的状况只要有变化就会发送出新的LSP)
     4. Time-to-live(TTL) for this packet
     5. LSP需要可靠传送，收到LSP的router只需保存最新的SEQNO(因为该封包反映了最新的link的状态)，然后该router继续向外广播(不回送)
  3. 实际网络中OSPF(Open Shortest Path First)使用了Link state Protocol
     1. Each router computes its routing table directly from the LSP's it has collected using the Dijkstra's algorithm
     2. Find the shortest path from the router to each other node