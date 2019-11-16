[TOC]

1. IB网络是无损的，不丢包的

2. 采用了基于信用的流控制

3. 由于采用了基于信用的流控制，有可能形成HoL现象，造成拥塞扩散

4. 通过设置包头的一个指定位来标记这个包是和拥塞有关的

5. 拥塞避免：显式拥塞通知（Explicit Congestion Notiﬁcation，ECN）包标记来解决拥塞问题。

6. HCA在操作系统的控制下提供地址转换机制，允许应用程序直接访问HCA。

7. InfiniBand子网管理器将本地标识符(LIDs)分配给连接到IB网络中每个端口，维护了一个基于所分配的LID的路由表。

8. 许多交换机包括子网管理器。配置IB结构至少需要一个子网管理器

9. (面向消息而不是包的)在IB中，完整的消息直接传递给应用程序。一旦应用程序请求传输RDMA读或写，IB硬件根据需要将超出大小的消息分割成数据包，数据包的大小由fabric路径最大传输单元决定。这些数据包通过IB网络传输，并直接交付到接收应用程序的虚拟缓冲区，在那里它们被重新组装成完整的消息。一旦接收到整个消息，接收应用程序就会收到通知

10. TCP 数据包没有长度限制，理论上可以无限长，但是为了保证网络的效率，通常 TCP 数据包的长度不会超过IP数据包的长度。IP包数据部分的最大长度为65515字节。以太网数据包的最大长度是1500个字符。如果需要传输的数据很长，就必须分割成多个帧进行发送。

11. IB网络需不需要检测数据包是否损坏？

12. 为什么freeflow拦截是是消息，ovs拦截的就是包? 因为verb发送的消息是在物理层才分段发的。

13. 可靠传输 VS 不可靠传输 ？

14. 一个WR完成后（不管失败与否），会放一个CQE到CQ中，发送一个通知给ibv_get_cq_event ，ibv_ack_cq_events 确认ibv_get_cq_event产生的事件

15. RDMA的控制操作可以使用TCP协议交换控制信息

16. QP的Send Queue和Receive Queue扮演的角色分别是什么？当应用需要通信时，就会创建一条Channel连接，每条Channel的首尾端点是两对Queue Pairs（QP）。每对QP由Send Queue（SQ）和Receive Queue（RQ）构成，这些队列中管理着各种类型的消息。QP会被映射到应用的虚拟地址空间，使得应用直接通过它访问RNIC网卡。

17. RDMA Write with Immediate ?支持立即数的RDMA写操作本质上就是给远程系统Push(推送)带外(OOB)数据, 这跟TCP里的带外数据是类似的。

    可选地，immediate 4字节值可以与数据缓冲器一起发送。 该值作为接收通知的一部分呈现给接收者，并且不包含在数据缓冲器中。

18. Host提交工作请求(WR)到工作队列(WQ): 工作队列包括发送队列(SQ)和接收队列(RQ)。工作队列的每一个元素叫做WQE, 也就是WR。
    Host从完成队列(CQ）中获取工作完成(WC): 完成队列里的每一个叫做CQE, 也就是WC。
    具有RDMA引擎的硬件(hardware)就是一个队列元素处理器。 RDMA硬件不断地从工作队列(WQ)中去取工作请求(WR)来执行，执行完了就给完成队列(CQ)中放置工作完成(WC)。
    原文链接：https://blog.csdn.net/qq_21125183/article/details/86522475

19. rdma_cm event只用于通知建连相关事件

## 关键概念

### WQ(Work Queue)

### WR(Work Requests)

### SR(Send Request) 

An SR defines how much data will be sent, from where, how and, with RDMA, to where.
struct ibv_send_wr is used to implement SRs. 

### RR(Receive Request)

An RR defines buffers where data is to be received for non-RDMA operations. If no buffers are
defined and a transmitter attempts a send operation or a RDMA Write with immediate, a receive
not ready (RNR) error will be sent. struct ibv_recv_wr is used to implement RRs. 

### CQ(Completion Queue)

A CQ contains completed work requests (WR) 。A completion queue holds completion queue
entries (CQE). Each Queue Pair (QP) has an associated send and receive CQ. A single CQ can be
shared for sending and receiving as well as be shared across multiple QPs. 

### CQE(Completion Queue Entry)

Each WR will generate a completion queue entry (CQE) that is placed on the CQ. The CQE will specify if the WR was completed successfully or not. 

### SQ(Send Queue)

The send queue holds information about memory buffers to be sent.

### SRQ(Shared Receive Queue)



### RQ(Receive Queue)

The receive queue holds information about which buffers to receive the incoming data. 

### QP(Queue Pair)

For every connection, each of two endpoints has a send queue (SQ) and a receive queue (RQ), together called QP.

#### CC(Completion Channel)

The CC is the mechanism that allows the user to be notified that a new CQE is on the CQ. A CQ is merely a queue that does not have a builtin notification mechanism. When using a polling paradigm for CQ processing, a CC is unnecessary. The user simply polls the CQ at regular intervals.  

### MR(Memory Registration or Memory Region )

### MW(Memory Window )





### GRH(Global Routing Header)

### PD(Protection Domain)

A protection domain is used to associate Queue Pairs with Memory Regions and Memory Windows, as a means for enabling and controlling network adapter access to Host System memory.PDs are also used to associate Unreliable Datagram queue pairs with Address Handles, as a means of controlling access to UD destinations.struct ibv_pd is used to implement protection domains. 

### LID (Local Identifier)

16位标识符

### GID (Global Identifier)

128位的单播或组播标识符，用于标识一个endport或组播组。GID是一个128位的地址，它利用了IPv6地址的许多概念和语义(RFC 2373)，但是，IBA没有定义GID和IPv6地址之间的关系，也就是说，没有为任何端口号或组播组定义GID和IPv6地址之间的映射。注意:这些规则只适用于IBA操作，除非特别指出，否则不适用于原始IPv6操作。

### GUID(Globally Unique Identifier)

Each channel adapter has a globally unique identifier (GUID) assigned by the channel adapter vender. Since local IDs assigned by the subnet manager are not persistent (i.e., might change from one power cycle to the next), the channel adapter GUID (called Node GUID) becomes the primary object to use for persistent identification of a channel adapter. Additionally, each port has a Port GUID assigned by the channel adapter vender.   

Each subnet is uniquely identified with a subnet ID known as the Subnet Prefix. The subnet manager programs all ports (via the PortInfo attribute) with the Subnet Prefix for that subnet. When combined with a Port GUID, this combination becomes a port’s natural GID. Ports may have other locally administrated GIDs. 



### XRC(eXtended Reliable Connection)  

#### AH(Address Handler)

An AH contains all of the necessary data to reach a remote destination. In connected transport modes (RC, UC) the AH is associated with a queue pair (QP). In the datagram transport modes (UD), the AH is associated with a work request (WR). 

### Fabric

A local-area RDMA network is usually referred to as a fabric. 

### CA(Channel Adapter)

A channel adapter is the hardware component that connects a system to the fabric.

#### HCA(Host Channel Adapter)

支持"verbs"接口的CA

#### TCA(Target Channel Adapter)

可以理解为"weak CA", 不需要像HCA一样支持很多功能

#### RNIC(RDMA Network Interface Card）

iWARP把一个CA称之为一个RNIC


## 系统文件
/sys/class/infiniband_verbs/abi_version ：abi版本