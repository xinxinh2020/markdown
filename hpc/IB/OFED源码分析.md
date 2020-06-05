[TOC]

OFED源码：http://www.mellanox.com/downloads/ofed/MLNX_OFED-4.6-1.0.1.1/MLNX_OFED_SRC-debian-4.6-1.0.1.1.tgz



# 目录

## 数据结构
### ibv_sge
Data is being gathered/scattered using scatter gather elements, which include:
- Address: address of the local data buffer that the data will be gathered from or scattered to.

- Size: the size of the data that will be read from / written to this address.

- L_key: the local key of the MR that was registered to this buffer.

struct ibv_sge implements scatter gather elements.

### ibv_device

### ibv_port_attr


## VPI Verbs API

### 初始化

#### ibv_fork_init 

ibv_fork_init初始化libibverb的数据结构，以安全地处理fork()函数并避免数据损坏，无论fork()是显式调用还是隐式调用，比如在system()调用中调用。
如果所有父进程线程都被阻塞，那么在所有子进程通过exec()操作结束或更改地址空间前，没有必要调用ibv_fork_init。

这个函数适用于支持madvise()的MADV_DONTFORK标志的Linux内核(2.6.17和更高版本)。

将环境变量RDMAV_FORK_SAFE或IBV_FORK_SAFE设置为任意值，与调用ibv_fork_init()具有相同的效果。将环境变量RDMAV_HUGEPAGES_SAFE设置为任意值，告诉库检查内核用于内存区域的底层页面大小。如果应用程序直接或间接地通过诸如libhugetlbfs之类的库使用巨页，则需要这样做。

调用ibv_fork_init()将降低性能，因为每次内存注册都需要额外的系统调用，并且需要分配额外的内存来跟踪内存区域。精确的性能影响取决于工作负载，通常不会显著。
设置RDMAV_HUGEPAGES_SAFE会进一步增加所有内存注册的开销。



### 设备操作

#### ibv_get_device_list 

ibv_get_device_list返回系统上可用的VPI设备列表。列表中的每个条目都是指向struct ibv_device的指针。ibv_device结构的列表将保持有效，直到该列表被释放。调用ibv_get_device_list之后，用户应该打开任何需要的设备，并通过ibv_free_device_list命令立即释放该列表。

#### ibv_free_device_list 

ibv_free_device_list释放ibv_get_device_list提供的ibv_device结构的列表。任何需要的设备都应该在调用此命令之前打开。一旦列表被释放，列表上的所有ibv_device结构都是无效的，不能再使用。

#### ibv_get_device_name 

返回指向ibv_device结构中包含的设备名称的指针。

#### ibv_get_device_guid 

ibv_get_device_guid以网络字节顺序返回设备64位全局惟一标识符(GUID)。

#### ibv_open_device 

ibv_open_device为用户提供一个谓词上下文，该上下文是将用于所有其他谓词操作的对象。

#### ibv_close_device 

关闭之前使用ibv_open_device打开的verb上下文。此操作不释放与上下文关联的任何其他对象。为了避免内存泄漏，必须在调用此命令之前独立释放所有其他对象。

#### ibv_node_type_str 

ibv_node_type_str返回一个节点类型的字符串，该字符串描述节点类型node_type值。这个值可以是IB的HCA、交换机、路由器、支持RDMA的NIC或Unknown。

### bv_port_state_str

bv_port_state_str返回一个描述端口状态enum值port_state的字符串



### Verb Context Operations 

#### ibv_query_device 

ibv_query_device检索与设备关联的各种属性。用户应该malloc一个struct ibv_device_attr，将它传递给命令，在成功返回时它的值会被填充。用户有责任释放这个结构体。



#### ibv_query_port 

ibv_query_port检索与端口关联的各种属性。

#### ibv_query_gid 

ibv_query_gid检索端口的全局标识符(GID)表中的条目。子网管理器(SM)为每个端口分配至少一个GID。GID是一个有效的IPv6地址，由全局惟一标识符(GUID)和SM分配的前缀(子网ID)组成。GID[0]是惟一的，并且包含端口的GUID。

#### ibv_query_pkey 

ibv_query_pkey检索端口的分区键(pkey)表中的条目。子网管理器(SM)为每个端口分配至少一个pkey。pkey标识端口所属的分区。pkey与以太网中的VLAN ID大致类似。

#### ibv_alloc_pd 

ibv_alloc_pd创建一个保护域(PD)。PDs限制哪些内存区域可以由哪些队列对(QP)访问，从而在一定程度上防止未经授权的访问。用户必须创建至少一个PD来使用VPI verbs。

#### ibv_dealloc_pd 

ibv_dealloc_pd释放一个保护域(PD)。如果当前有任何其他对象与指定的PD关联，则此命令将失败

#### ibv_create_cq 

ibv_create_cq创建一个完成队列(CQ)。

参数cqe定义队列的最小大小。队列的实际大小可以大于指定的值。
参数cq_context是一个用户定义的值。如果在CQ创建期间指定了该值，那么当使用completion channel （CC)时，该值将作为ibv_get_cq_event中的参数返回。参数通道用于指定CC。CQ只是一个没有内置通知机制的队列。当为CQ处理使用轮询范式时，CC是不必要的。用户只需定期轮询CQ。但是，如果希望使用pend范式，则需要CC。CC是一种机制，它允许用户被通知有一个新的CQE正在进行中。



#### ibv_resize_cq 

ibv_resize_cq调整完成队列(CQ)的大小。参数cqe必须至少是队列中未完成条目的数量。队列的实际大小可以大于指定的值。当CQ被调整大小时，它可能包含(也可能不包含)完成，因此，它可以在使用CQ的工作期间调整大小。

#### ibv_destroy_cq 

ibv_destroy_cq释放一个完成队列(CQ)。如果有任何队列对，此命令将失败(QP)，它仍然与指定的CQ相关联。

#### ibv_create_comp_channel 

ibv_create_comp_channel创建一个完成通道(CC)。完成通道是一种机制,当新的完成队列事件(CQE)被放置在一个完成队列(CQ)上时，用户会接收通知。

#### ibv_destroy_comp_channel 

ibv_destroy_comp_channel释放一个完成通道。如果仍然有任何与此完成通道关联的完成队列(CQ)，则此命令将失败。

### Protection Domain Operations 

一旦您建立了一个保护域(PD)，您就可以在该域中创建对象。本节描述PD上可用的操作。这包括注册内存区域(MR)，创建队列对(QP)或共享接收队列(SRQ)和地址句柄(AH)。

#### ibv_reg_mr 

ibv_reg_mr注册一个内存区域(MR)，将其与一个保护域(PD)关联，并为其分配本地和远程密钥(lkey, rkey)。所有使用内存的VPI命令都需要通过该命令注册内存。相同的物理内存可以映射到不同的MRs，甚至允许根据用户需求将不同的权限或PDs分配到相同的内存。

访问标志可以是位的，也可以是下列枚举之一:

- IBV_ACCESS_LOCAL_WRITE ： Allow local host write access
- IBV_ACCESS_REMOTE_WRITE ：Allow remote hosts write access
- IBV_ACCESS_REMOTE_READ：Allow remote hosts read access
- IBV_ACCESS_REMOTE_ATOMIC：Allow remote hosts atomic access
- IBV_ACCESS_MW_BIND：Allow memory windows on this MR

本地读访问是隐含的和自动的。

任何违反给定内存操作的访问权限的VPI操作都将失败。注意，队列对(QP)属性也必须具有正确的权限，否则操作将失败。如果设置了IBV_ACCESS_REMOTE_WRITE或IBV_ACCESS_REMOTE_ATOMIC，则I还必须设置IBV_ ACCESS_LOCAL_WRITE。

#### ibv_dereg_mr 

释放内存区域(MR)。如果有任何内存窗口仍绑定到MR，操作将失败。

#### ibv_create_qp 

ibv_create_qp创建一个QP。当一个QP创建时，它将进入重置状态。

#### ibv_destroy_qp 

ibv_destroy_qp释放队列对(QP)

#### ibv_create_srq 

ibv_create_srq创建一个共享接收队列(SRQ)。读取srq_attr->max_wr和srq_attr->max_sge，以确定请求的SRQ大小，并将其设置为返回时分配的实际值。如果ibv_create_srq成功，那么max_wr和max_sge将至少与请求值一样大。

#### ibv_modify_srq 

#### ibv_destroy_srq 

#### ibv_open_xrc_domain 

#### ibv_create_xrc_srq 

#### ibv_close_xrc_domain 

#### ibv_create_xrc_rcv_qp 

ibv_create_xrc_rcv_qp创建一个XRC队列对(QP)，只作为接收端QP，并通过xrc_rcv_qpn返回QP号。这个数字必须传递到远程(发送方)节点。远程节点将在ibv_post_send中使用xrc_rcv_qpn，当它向该主机上的一个XRC SRQ发送消息时
位，与XRC接收QP相同的XRC域中

#### ibv_modify_xrc_rcv_qp 

### Queue Pair Bringup  

队列对(QP)必须通过一个递增的状态序列进行转换，然后才能用于通信。状态：RESET -> INIT -> RTR -> RTS 

#### RESET to INIT 

新创建队列对(QP)时，它处于重置状态。需要发生的第一个状态转换是将QP置于INIT状态。

一旦QP转换为INIT状态，用户就可以开始通过ibv_post_recv命令将接收缓冲区post到接收队列。在QP可以转换为RTR状态之前，至少应该发布一个接收缓冲区

#### INIT to RTR 

一旦队列对(QP)有了发送到它的接收缓冲区，现在就可以将QP转换为ready to receive (RTR)状态

#### RTR to RTS 

一旦队列对(QP)达到了ready to receive (RTR)状态，就可以将其转换为ready to send (RTS)状态。

一旦QP转换到RTS状态，QP就开始发送处理并完全运行。用户现在可以使用ibv_post_send命令发布发送请求。

### Active Queue Pair Operations 

QP可以从创建它的地方开始查询，一旦队列对完全运行，您可以查询它，得到事件通知，并对它执行发送和接收操作。
本节描述可用于执行这些操作的操作。

#### ibv_query_qp 

ibv_query_qp检索前面通过ibv_create_qp和ibv_modify_qp设置的队列对(QP)的各种属性。
用户应该分配一个struct ibv_qp_attr和一个struct ibv_qp_init_attr，并将它们传递给命令。这些结构将在成功返回时填充。用户有责任释放这些结构。

#### ibv_query_srq 

ibv_query_srq返回指定SRQ的属性列表和当前值。它通过指针srq_attr返回属性，该指针是上面在ibv_create_srq下描述的ibv_srq_attr结构。如果srq_attr中的srq_limit值为0，则达到的SRQ limit(“低水印”)事件没有或不再武装。在重新武装事件之前，不会生成任何异步事件

#### ibv_query_xrc_rcv_qp 

#### ibv_post_recv 

ibv_post_recv向队列对的接收队列(QP)发送一个WRs链表。至少应该将一个接收缓冲区发布到接收队列，以便将QP转换为RTR。接收缓冲区在远程对等端执行Send、Send with Immediate和RDMA Write时使用直接操作。接收缓冲区不用于其他RDMA操作。在第一个错误时停止对WR列表的处理，并在bad_wr中返回指向违规WR的指针



#### ibv_post_send 

ibv_post_send将一个WRs链表发布到队列对的(QP)发送队列。此操作用于初始化所有通信，包括RDMA操作。在第一个错误时停止对WR列表的处理，并在bad_wr中返回指向违规WR的指针。
在请求完全执行并从相应的完成队列(CQ)检索到一个完成队列条目(CQE)以避免意外行为之前，用户不应该更改或销毁与WRs关联的AHs。WR所使用的缓冲区只能在WR完全执行完之后才能安全地重用



#### ibv_post_srq_recv 

ibv_post_srq_recv将工作请求列表发布到指定的SRQ。它停止处理
在第一次失败时从这个列表中获取WRs(可以在发出请求时立即检测到)，并通过bad_recv_wr参数返回这个失败的WR。WR使用的缓冲区只能在WR完全执行请求并从相应的完成队列(CQ)检索工作完成之后安全地重用。
如果一个WR被发送到一个UD QP，则传入消息的全局路由头(GRH)将缓冲区的第40个字节开始

####  ibv_req_notify_cq 

bv_req_notify_cq为指定的完成队列(CQ)配置通知机制。
当一个完成队列条目(CQE)放在CQ上时，一个完成事件将被发送到与CQ关联的完成通道(CC)。如果该CQ中已经存在CQE，则不会为该事件生成事件。如果设置了solicited_only标志，那么只有具有已请求标志集的WRs的CQEs才会触发通知。用户应该使用ibv_get_cq_event操作来接收通知。通知机制将只装备一个通知。一次没有。

#### ibv_get_cq_event 

ibv_get_cq_event等待在指定的(CC)上发送的过来的通知。注意，这是一个阻塞操作。必须使用ibv_ack_cq_events操作确认发送的每个通知。因为ibv_destroy_cq操作等待所有要确认的事件，所以如果没有正确确认任何事件，它将挂起。



#### ibv_poll_cq 

ibv_poll_cq从完成队列(CQ)检索CQEs。

cq必须定期进行轮询，以防止出现超限。在发生溢出时，CQ将被关闭，并发送一个异步事件IBV_EVENT_CQ_ERR



#### ibv_init_ah_from_wc 

#### ibv_attach_mcast 

ibv_attach_mcast将指定的QP QP附加到其组播组的,组播组,GID是GID组播Lib是lib

只有传输服务类型IBV_QPT_UD的QPs可以附加到多播组.为了接收组播消息，必须将组播组的连接请求发送给子网管理员(SA)，以便将fabric的组播路由配置为将消息发送到本地端口



#### ibv_detach_mcast  

ibv_detach_mcast从组播组GID为GID、组播LID为LID的组播组中分离指定的QP。



### Event Handling Operations 

## RDMA_CM API

### Event Channel Operations 

#### rdma_create_event_channel 

打开用于报告通信事件的事件通道。异步事件通过事件通道报告给用户。

每个事件通道都映射到一个文件描述符。可以像其他fd一样使用和操作关联的文件描述符来更改其行为。用户可以使fd非阻塞、poll或select fd等。



### Connection Manager (CM) ID Operations 

#### rdma_create_id 

**rdma_cm_ids are conceptually equivalent to a socket for RDMA communication.** 


