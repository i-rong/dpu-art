# 连接方式
## Reliable Connection (RC)
队列对只与一个其他QP相关联。由一个QP的发送队列发送的消息可靠地传递到另一个QP的接收队列。数据包是按顺序传递的。RC连接与TCP连接非常相似。
## Unreliable Connection (UC)
一个队列对只与另一个QP相关联。连接不可靠，数据包可能丢失。传输层不会重试带有错误的消息，错误处理必须由更高级别的协议提供。
## Unreliable Datagram (UD)
队列对可以向任何其他UD QP发送和接收单包消息。序和消息传递不保证，交付的数据包可能被接收方丢弃。支持多播消息(一对多)。UD连接与UDP连接非常相似。

# 传输上下文相关名词
## Send Request (SR)
SR定义了发送多少数据、从哪里、如何发送以及RDMA发送到哪里。struct ibv_send_wr用于实现SR
```
struct ibv_send_wr {
	uint64_t		wr_id;
	struct ibv_send_wr     *next;
	struct ibv_sge	       *sg_list;
	int			num_sge;
	enum ibv_wr_opcode	opcode;
	unsigned int		send_flags;
	/* When opcode is *_WITH_IMM: Immediate data in network byte order.
	 * When opcode is *_INV: Stores the rkey to invalidate
	 */
	union {
		__be32			imm_data;
		uint32_t		invalidate_rkey;
	};
	union {
		struct {
			uint64_t	remote_addr;
			uint32_t	rkey;
		} rdma;
		struct {
			uint64_t	remote_addr;
			uint64_t	compare_add;
			uint64_t	swap;
			uint32_t	rkey;
		} atomic;
		struct {
			struct ibv_ah  *ah;
			uint32_t	remote_qpn;
			uint32_t	remote_qkey;
		} ud;
	} wr;
	union {
		struct {
			uint32_t    remote_srqn;
		} xrc;
	} qp_type;
	union {
		struct {
			struct ibv_mw	*mw;
			uint32_t		rkey;
			struct ibv_mw_bind_info	bind_info;
		} bind_mw;
		struct {
			void		       *hdr;
			uint16_t		hdr_sz;
			uint16_t		mss;
		} tso;
	};
};
```
## Receive Request (RR)
RR定义了缓冲区，用于接收非rdma操作的数据。如果没有定义缓冲区，并且发送端试图立即进行发送操作或RDMA写操作，则会发送一个接收未就绪(receive not ready, RNR)错误。struct ibv_recv_wr用于实现RRs。
```
struct ibv_recv_wr {
	uint64_t		wr_id;
	struct ibv_recv_wr     *next;
	struct ibv_sge	       *sg_list;
	int			num_sge;
};
```
## Completion Queue
完成队列包含已完成的提交到工作队列(WQ)的工作请求。每一个完成都表示完成了一个特定的WR(包括成功完成的WRs和未成功完成的WRs)。完成队列是一种通知应用程序已结束工作请求信息(状态、操作码、大小、来源)的机制。CQs有n个完成队列条目(CQE)。CQ的数量在创建CQ时指定。当一个CQE被轮询时，它将从CQ中移除。CQ是cqe的先进先出。CQ可以服务发送队列，接收队列，或两者兼而有之。来自多个QPs的工作队列可以与单个CQ相关联。struct ibv_cq用于实现CQ。
```
struct ibv_cq {
	struct ibv_context     *context;
	struct ibv_comp_channel *channel;
	void		       *cq_context;
	uint32_t		handle;
	int			cqe;

	pthread_mutex_t		mutex;
	pthread_cond_t		cond;
	uint32_t		comp_events_completed;
	uint32_t		async_events_completed;
};
```
## Memory Registration
内存注册是一种机制，允许应用程序使用虚拟地址向网络适配器描述一组虚拟连续的内存位置或一组物理上连续的内存位置，作为虚拟连续的缓冲区。
注册过程固定内存页(以防止页被换出，并保持物理<->虚拟映射)。注册时，操作系统会检查注册块的权限。注册过程将虚拟地址表写入到网络适配器。在注册内存时，会为该区域设置权限。权限包括本地写、远程读、远程写、原子和绑定。每个MR有一个远程密钥和一个本地密钥(r_key, l_key)。本地密钥由本地HCA使用来访问本地内存，例如在接收操作期间。远程键被提供给远程HCA，以允许远程进程在RDMA操作期间访问系统内存。同一个内存缓冲区可以被注册多次(即使有不同的访问权限)，每次注册都会产生不同的键。
```
struct ibv_mr {
	struct ibv_context     *context;
	struct ibv_pd	       *pd;
	void		       *addr;
	size_t			length;
	uint32_t		handle;
	uint32_t		lkey;
	uint32_t		rkey;
};
```
## Memory Window
MW允许应用程序更灵活地控制对其内存的远程访问。内存窗口适用于以下情况:\
1.希望以动态的方式授予和撤销对已注册区域的远程访问权限，并且比使用注销/注册或重新注册的性能损失更小。\
2.希望为不同的远程代理授予不同的远程访问权限，并/或在注册区域内的不同范围内授予这些权限。\
3.将MW与MR相关联的操作称为绑定。不同的MWs可以重叠同一个MR(具有不同访问权限的事件)。

## Protect Domain
PD全称是Protection Domain，意为"保护域"。在RDMA中，PD像是一个容纳了各种资源（QP、MR等）的“容器”，将这些资源纳入自己的保护范围内，避免他们被未经授权的访问。一个节点中可以定义多个保护域，各个PD所容纳的资源彼此隔离，无法一起使用。\
一个用户可能创建多个QP和多个MR，每个QP可能和不同的远端QP建立了连接. 由于MR和QP之间并没有绑定关系，这就意味着一旦某个远端的QP与本端的一个QP建立了连接，具备了通信的条件，那么理论上远端节点只要知道VA和R_key（甚至可以靠不断的猜测直到得到一对有效的值），就可以访问本端节点某个MR的内容。由于MR和QP之间并没有绑定关系，这就意味着一旦某个远端的QP与本端的一个QP建立了连接，具备了通信的条件，那么理论上远端节点只要知道VA和R_key（甚至可以靠不断的猜测直到得到一对有效的值），就可以访问本端节点某个MR的内容。Address Handle，Memory Window等也是由PD进行隔离保护的.\

IB协议中规定：每个节点都至少要有一个PD，每个QP都必须属于一个PD，每个MR也必须属于一个PD。

## SQ和RQ
任何通信过程都要有收发两端，QP就是一个发送工作队列和一个接受工作队列的组合，这两个队列分别称为SQ（Send Queue）和RQ（Receive Queue）。\
SQ专门用来存放发送任务，RQ专门用来存放接收任务。在一次SEND-RECV流程中，发送端需要把表示一次发送任务的WQE放到SQ里面。同样的，接收端软件需要给硬件下发一个表示接收任务的WQE，这样硬件才知道收到数据之后放到内存中的哪个位置。上文我们提到的Post操作，对于SQ来说称为Post Send，对于RQ来说称为Post Receive。\
需要注意的是，在RDMA技术中通信的基本单元是QP，而不是节点。如下图所示，对于每个节点来说，每个进程都可以使用若干个QP，而每个本地QP可以“关联”一个远端的QP。我们用“节点A给节点B发送数据”并不足以完整的描述一次RDMA通信，而应该是类似于“节点A上的QP3给节点C上的QP4发送数据”\
每个节点的每个QP都有一个唯一的编号，称为QPN（Queue Pair Number），通过QPN可以唯一确定一个节点上的QP。
##