---
layout: post
title: 《计算机网络概论》学习笔记整理之Transport Layer
date: 2020-05-02
tag: 
- 网络
---

这篇笔记主要是关于传输层的记录，包括两种协议：UDP和TCP。分为下面四部分：

1. Introduce how to convert host-to-host packet delivery service to process-to-process communication channel
2. UDP for unreliable transmission service
3. TCP for reliable transmission service
   1. 3-way handshaking connection establishment
   2. TCP connection state diagram
   3. TCP timeout value calculation
   4. TCP retransmission scenaries
   5. TCP fast retransmission
4. TCP congestion control
   1. AIMD(Additive Increase Multiplecative Decrease)
   2. Slow start
   3. 3-duplicate ACKs(packet lose)
   4. Timeout

<!-- more -->

---

### End-to-end protocols

1. Transport protocol应该能提供下面的功能：
   1. Guaranteed message delivery
   2. Delivers message in the same order they were sent(内容的顺序不能乱)
   3. Delivers at most one copy of each message
   4. Supports arbitrarily large messages(支持任意大小信息的传送)
   5. Supports synchronization between the sender and the receiver
   6. Allows the reveiver to apply folw control to the sender
   7. Supports multiple application processes on each host

2. Transport Layer下一层比如IP提供的服务特点：
   1. Drop messages: overflow/congestion
   2. Reorder messages
   3. Delivery duplicate copies of a given message
   4. Limit messages to some finite size(MTU)
   5. Delivery messages after an arbitrarily long delay

  以上这些都会导致unreliable service

3. Transport protocol面临的挑战
   1. UDP：unreliable service => unreliable service
   2. *TCP：unreliable service => reliable service*

---

### UDP

IP提供的服务是host-to-host，但真正的信息交流发生在application之间，所以UDP的作用在于：IP只能知道信息属于哪台host，UDP再进一步确定信息发送给该台host上的哪个application：

1. Extends host-to-host delivery service of the underlying network into a process-to-process communication service
2. Adds a level of demultiplexing which allows multiple application processes on each host to share the network

不同的application使用port number进行区隔：

  1. DstPort：收该封包的host的应用的port number
  2. SrcPort：丢该封包的host的应用的port number
  3. Checksum
  4. Length
  5. Data

<img src="/images/network/UDP.png" width="800" alt="UDP" />

### TCP

TCP是Transmission Control Protocol的缩写，具有下面的特点：

1. Reliable Byte Stream

  1. Reliable
  2. connection oriented
  3. Byte-Stream service

  第三点是说TCP关注数据的总量，不在乎这些数据使用几个封包传送，简单来说可以看成一连串的byte，只要总量和顺序对就OK，不要求两边的节奏也一样

2. Flow control vs Congestion control

  1. Flow control: involves preventing senders from overrunning the capacity of the reveivers
  2. Congestion control: involves preventing too much data from being injected into the network, thereby causing routers/switchers or links to become overloaded，在处理constion的时候，面临两个问题：
     1. 如何侦测到internet正在congestion
     2. 侦测到congestion如何处理

3. TCP segment
   1. TCP is a byte-oriented protocol
   2. The sender writes byte into a TCP connection and the receiver reads byte out of the TCP connection
   3. However, TCP does not transmit individual bytes over Internet(会将byte累积起来再送)
   4. The source TCP buffers enough bytes from the sending process to fill a reasonable sized packet and then sends this packet to its peer on the destination host
   5. The destination TCP then puts the contents of the packet into a receiver buffer, and the receiving process reads from the buffer
   6. The packets exchanged between TCP peears are called segments

  <img src="/images/network/TCPStream.png" width="800" alt="TCPStream" />

4. TCP header
   1. Source Port：发送该封包的host上的application对应的port，用于确定具体的application
   2. Destination Port：接收该封包的host上的application对应的port，用于确定具体的application
   3. Sequence Number(32bits)：表示封包带的data的第一个byte在整个byte stream中的编号是第几号(TCP是byte-oriented，每个byte都有一个sequence number)
   4. Acknowledgement Number(32bits)：告诉对方这个number之前的byte通通都收到了
   5. Flags
      1. SYN(1bit)：设为1表示建立连线
      2. FIN(1bit)：设为1表示对方要把连线关掉
      3. ACK(1bit)：设为1表示上面的Acknowledgement Number是有意义的值
      4. RST(1bit)：设为1表示强制激励将连接清除
      5. PSH(1bit)：设为1表示不管buffer里的内容为多少，都强制将其冲出去
      6. URG(1bit)：设为1表示该封包带有紧急的资料，从Urgent Pointer字段中可以查看
   6. Urgent Pointer：指向data中某个位置，该位置之前都是紧急的资料(比如控制信息)，之后才是普通数据
   7. Advertised Window(16bits)：用来做flow control(不是congestion control)，2^16=65536byte=64k，表示receiver给sender最多的量就是64k，所以sender可以送过来的量最多就是advertised window中的数字，最大为64k。64k的额度额度(credit)用完，sender就需要停下，等receiver送来下一个额度
   8. Checksum：used exactly the same way as for UDP——it is computed over the TCP header, the TCP data, andt the psendoheader, which is made up of the source address, destination address, and length fields from the IP header，这样能做全面的校验，保证数据的有效性

---

### TCP Connection Management

1. TCP sender, receiver establish connection before exchanging data segments
2. initialize TCP variables：
   1. sequence numbers(不一定从0开始，双方商议好即可)
   2. Buffers, flow control info(advertised window)
3. Client: connection initiator
   ```
    Socket clientSocket = new Socket(hostname, port number);
   ```
4. Server：contacted by client

    ```
      server端也事先打开端口，时时准备接受client端发送来的数据

      Socket connectionSocket = weilcomeSocket.accept();
    ```
5. 3-way handshake
   1. Client sends TCP SYN segment to server
      - specifies initial seq#
      - no data
   2. Server receives SYN, replies with SYN/ACK segment
      - server allocates(分配) buffers
      - specifies server initial seq#
   3. client receives SYN/ACK, replies with ACK segment, which may contain data

  <img src="/images/network/establishTCPConnection.png" width="600" alt="establishTCPConnection" />

6. 关闭一个连接

  关闭连接双方多可以发起，以客户端发起为例:

  1. clientSocket.close();
  2. client sends TCP FIN control segment to server
  3. server receives FIN, replies with ACK, closes connection and sends FIN
  4. client receives FIN, replies with ACK and enters "timed wait" will respond with ACK to received FINs
  5. server receives ACK, connection closed

  <img src="/images/network/closeTCPConnection.png" width="600" alt="closeTCPConnection" />

7. TCP状态图

  <img src="/images/network/TCPStateDiagram.png" width="600" alt="TCPStateDiagram" />

---

### TCP timeout的设计

在每一个封包送出去时，都会涉及一个timer，希望能在这个timer timeout之前收到对方的acknowledgement

1. Original Algorithm
   1. Measure sampleRTT for each segment/ACK pair
   2. Compute weighted average of RTT
   
    ```
      EstRTT = α × EstRTT + (1-α) × sampleRTT

      α between 0.8 and 0.9
    ```
    3. Set timeout based on EstRTT: Timeout = 2 × EstRTT

  这种算法的问题在于如果出现封包重送，无法分辨ACK是属于第一次送出的还是后续送出的，导致算不准sampleRTT

2. Karn/Partridge Algorithm
   1. Do not sample RTT when retransmitting
   2. Double timeout after each retransmission

  这种方法没有办法消除网络的congestion

3. Jscobson/Karels Algorithm

  原版算法最主要的问题是没有考虑variance of SampleRTT
     1. For small variance among sampleRTTs
        1. Then the EstRTT can be better trusted
        2. There is no need to multiply this by 2 to compute the timeout
     2. For large variance among sampleRTTs
        1. The timeout value should not be tightly coupled to the EstRTT

  也就是说为了准确的计算timeout，要将EstRTT的variance的变动幅度也考虑进来

  1. Difference = sampleRTT - EstRtt
  2. EstRTT = EstRTT + (δ × Difference)
  3. Deviation(偏差) = Deviation + δ × (|Difference| - Deviation), where δ is a factor between 0 and 1
  4. Timeout = μ × EstRTT + φ × Deviation

    - where based on experience, μ = 1 and φ = 4
    - Thus, *when the variance is small, Timeout is close to EstRTT; a large variance causes the deviation term to dominate the calculation*

---

### TCP retransmission scenarios

  <img src="/images/network/TCPRetransmissionScenarios.png" width="1000" alt="TCPRetransmissionScenarios" />

  左侧是2中重送的场景，右侧是ACK累计的场景

---

### TCP fast Retransmission

  <img src="/images/network/TCPFastRetransmission.png" width="500" alt="TCPFastRetransmission" />

1. Every time a data packet arrives at the receiving side, the receiver responds with an acknowledgement, even if this sequence number has already been acknowledged
2. Thus, when a packet arrives out of order —— TCP resends the same acknowledgement it sent the last time
3. This second transmission of the same acknowledgement is called a duplicate ACK
4. When the sending side sees a duplicate ACK, it knows that the other side must have received a packet out of order, which suggests that an earlier packet might have been lost
5. Since it is also possible that the earlier packet has only been delayed rather than lost, the sender waits until it sees some number of duplicate ACKs and then retransmits the missing packet
6. TCP waits until it has seen *three duplicate ACKs* before retransmitting the packet

不用非得等到timeout再重送！！！

---

### TCP congestion control

1. The idwa of TCP congestion control is for each source to determine *how much capacity is available in the network*, so that it knows how many packets it can safely have in transit
2. TCP is said to be self-clocking by using ACKs to pace the transmission of packets
3. Additive Increase Multiplicative Decrease(AIMD)

  线性加速，发现壅挤马上减一半

  1. CongestionWindow: used by the source to limit how much data it is allowed to have in transmit simultaneously for a connection
  2. The CongestionWindow is congestion control's counterpart to flow control's AdvertisedWindow
  3. The maximum number of bytes of unacknowledged data allowed is now the minimum of the CongestionWindow and the AdvertisedWindow
  4. Transmission rate

    ```
      Rate = CongestionWindow / RTT bytes/sec
    ```
  5. TCP's effective window is revised as follows:

    - MaxWindow = MIN(CongestionWindow, AdvertisedWindow)
    - EffectiveWindow = MaxWindow - (LastByteSent - LastByteAcked)
  6. A TCP source is allowed to send no faster than the slowest component can accommodate(容纳)

    - the network or
    - the destination host
  7. CongestionWindow的值如何得到？
    TCP source sets the CongestionWindow based on the congestion level it observed(MSS 开始最大的一个segment，和面有提到MSS)

    - Decreasing the CongestionWindow when the level of congestion goes up and
    - Increase the CongestionWindow when the level of congestion goes down
  8. TCP 如何判断network congestion？

    - TCP interpres packet lose(3-duplicate ACK) as a sign of congestion and reduces the rate
    - Each time a packet lose occurs, the source sets CongestionWindow to half of its previous value
  9. CongestionWindow 被定义为bytes的量，这里理解为封包的数量便于理解，不能小于1个MSS(maximum segment size)
  10. 增加CongestionWindow:Every time the source sucessfully sends a CongestionWindow's worth of packets(all packets sent out during the last RTT have been ACKed), it adds the equivalent of 1 packet of CongestionWindow，也就是1个MSS
  11. 更积极的方法：不等所有ACK都收到才能加1MSS，而是每收到1个ACK就先增加一部分，比如：
  ```
    - cw = 5 x MSS, 每收到一个ACK就先增加1/5MSS
    - cw = 8 x MSS, 每收到一个ACK就先增加1/8MSS

    Increment = MSS x (MSS / CongestionWindow)
    CongestionWindow += Increment
  ```
4. Slow Start

AIMD比较适用的场景是传输的速度快逼近上限，问题在于一开始时，网络上比较空，按AIMD线性加速太慢了

- The additive increase mechanism is good when the source is operating close to the available capacity of the network, but it takes too long to ramp up(加强，加快) a connection when it is starting from scratch
- Slow Start: to increase the CongestionWindow rapidlly from a cold start
- Slow Start effectively incerases the CongestionWindow exponentially rather than linearly
- 工作过程
  1. Initially, the CongestionWindow = 1 packet
  2. 比如 MSS = 500bytes, RTT = 200msec, initial rate = 20kbps(500 x 8 / 200 = 20, 起始速度很慢，所以叫做slow start)
  3. When the ACK for this packet arrives, TCP adds 1 to CongestionWindow and then sends 2 packets，也就是说，每收到1个ACK就增加1个MSS
  4. TCP effectively double the numebr of packets it has in transmit every RTT

5. After 3 duplicate ACKs:
  - CW is cut in half(减半)
  - window then grows linearly
6. But after timeout event(timeout 更严重)
  - CW instead set to 1 MSS
  - window then grows exponentially
  - to a threashold(原来cw的一半), then grows linearly

---

### TCP Congestion Control Summary

1. When CW is below Threshold, sender is in slow-start phase, window grows exponentially
2. When CW is above Threshold, sender is in congestion-avoidance phase, window grows linearly
3. When a triple duplicate ACK occursm  threshold set to CW/2 and CW set to threshold
4. When timeout occurs, threshold set to CW/2 and CW is set to 1 MSS

  <img src="/images/network/TCPCongestionCurve.png" width="900" alt="TCPCongestionCurve" />

| State          | Event                                   | TCP Sender Action                                            | Commentary                                                   |
| -------------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| slow start(SS) | ACK receipt for previously unacked data | CW=CW+MSS<br />if(cw > threshold) set state to Conegstion Avoidance(CA) | Resulting in a doubling of CW every RTT                      |
| CA             | ACK receipt for previously unacked data | CW=CW+MSSxMSS/CW                                             | Addivtive increase resulting in increase of CW by MSS/RTT    |
| SS/CA          | Loss event detected by 3-dup-ACK        | threshold=CW/2<br />CW=CW/2<br />state=CA                    | Fast recovery, implementing multiplicative decrease CW will not drop below 1 MSS |
| SS/CA          | Timeout                                 | threshold=CW/2<br />CW=1MSS<br />state=SS                    | Enter SS                                                     |
| SS/CA          | duplicate ACK                           | Increaent count of  duplicate ACK for segment added          | CW和threshold不变                                            |



---

### TCP throughput

1. Let w be the window size when loss occurs
2. When window is w, throughtput is w/RTT
3. Just after loss, window drops to w/2, throughput to w/2RTT
4. Average throughput is 0.75 w/RTT