# In Search of an Understandable Consensus Algorithm (Extended Version)

# Abstract

Raft is a consensus algorithm for managing a replicated log.

 

key elements of consensus<a id="这是你的锚点"></a>

- leader election
- log replication
- safety



includes a new mechanism for changing the cluster membership



# 1.Introduction

共识算法允许一组机器作为一个连贯的组工作，可以在其一些成员的故障中存活下来



why

1. understandability
2. practical



AD

1. <a href="#这是你的锚点">decomposition</a>
2. state space reduction
   用于复制状态机？



features

• **Strong leader**: Raft uses a stronger form of leadership than other consensus algorithms. For example, log entries only flow from the leader to other servers. This simplifies the management of the replicated log and makes Raft easier to understand.

• **Leader election**: Raft uses randomized timers to elect leaders. This adds only a small amount of mechanism to the heartbeats already required for any consensus algorithm, while resolving conflicts simply and rapidly.

• **Membership changes**: Raft's mechanism for changing the set of servers in the cluster uses a newjoint consensus approach where the majorities of two different configurations overlap during transitions. This allows the cluster to continue operating normally during configuration changes.

# 2.Replicated state machines

共识算法consensus algorithm 常用于 复制状态机replicated state machines



些许复制状态机的知识

# 3. What's wrong with Paxos

1. difficult to understand
2. don't provide a  good foundation



# 4.Designing for understandability

**understandability**



two techniques

1.  <a href="#这是你的锚点">separate</a>
2. simplify the state space
   引入随机化（用于leader election）等

# 5.The Raft consensus algorithm

Raft 通过首先选举一个有区别的领导者来实现共识，然后让领导者完成管理复制日志的责任

领导者接受来自客户端的日志条目，在其他服务器上复制它们，并告诉服务器何时可以安全地将日志条目应用于他们的状态机

![image-20240806163152431](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240806163152431.png)

![image-20240806163202814](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240806163202814.png)

## 5.1 Raft basics

典型 服务器数量：5，可以容忍2 个错误



**三个状态**

1. leader
   领导者处理所有客户端请求（如果客户端接触跟随者，跟随者将其重定向到领导者）
2. follower
   追随者是被动的：他们自己没有请求，而是简单地响应领导者和候选人的请求。
3. candidate
   第三个状态候选，用于选举一个新的领导者，如下所述Figure 4

![image-20240806163905369](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240806163905369.png)



**term**
每个term只有一个leader

![image-20240806164142762](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240806164142762.png)

> term在Raft中充当逻辑始终logical clock，每个服务器存储器信息
>
> term有对应的term number，单调递增
>
> 每当服务器通信时，都会交换当前条款;如果一台服务器的当前项小于另一台服务器的，则将其当前项更新为较大的值。如果候选人或领导者发现其任期已过，则立即恢复为追随者状态。如果服务器接收到一个带有过期术语号的请求，它将拒绝该请求。



**rpc**
Raft服务器使用远程过程调用(rpc)进行通信，基本的一致性算法只需要两种类型的rpc。

> RequestVote rpc由候选人candidate在选举期间发起(章节5.2)，
>
> AppendEntries rpc由领导者leader发起，以复制日志条目并提供心跳形式heartbeat(章节5.3)。

第7节添加了第三个RPC，用于在服务器之间传输快照。如果服务器没有及时收到响应，它们会重试rpc，并且它们会并行地发出rpc以获得最佳性能。



## 5.2 Leader election

 

使用heartbeat mechanism 来触发 leader 选举

heart beat（AppendEntries RPCs with no log entry）



如果跟随者在称为选举超时的一段时间内没有收到通信*election timeout*，那么它假设没有可行的领导者并开始选举以选择新的领导者。



**开始选举**：
term number+1
转变candidate state
投票给自己
并行向其他服务器发送投票请求



候选人一直处于这种状态，直到发生以下三种情况之一:

- 它赢得了选举
  *超过半数同意* — 确保一个任期只有一个leader

  > 只有一票
  > 先到先得

  候选人一旦赢得选举，就成为领袖。然后，它向所有其他服务器发送心跳消息，以建立其权威并防止新的选举。

- 另一个服务器确立了自己的领导地位
  需要其term number至少跟自己一样

- 一段时间过去了，没有赢家。
  candidate太多，分票 — 需要采取措施 — **randomized election timeouts**

  > randomized election timeouts
  >
  > 投票时每个server的选举超时时间（从follower到candidate需要等待的时间）
  >
  > 是随机的，因此成为candidate是有先后的



## 5.3 Log replication

每个log还有一个记录对应的term number 和log index

![image-20240807093813709](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240807093813709.png)

*committed* entry

> leader向自己的state machine提交的log entry
> 需要大多数server已经复制该条目，如图六的log7
> followers直到log提交后，应用到其本地状态机

![image-20240807095612041](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240807095612041.png)

在Raft中，领导者通过强迫追随者的日志复制自己的日志来处理不一致。这意味着跟随者日志中的冲突条目将被来自领导者日志的条目覆盖。第5.4节将说明，如果加上另外一个限制（要求想成为candidate必须要知道所有的committed log），这样做是安全的。



Log Matching Property in Figure 3:

- If two entries in different logs have the same index and term, then they store the same command
-  If two entries in different logs have the same index and term, then the logs are identical in all preceding entries.



同时leader知道自己的最大的log index
leader为每个follower维护一个nextIndex，这是leader将发送给该follower的下一个日志条目的索引。

> 当领导者首次掌权时，它将所有nextIndex值初始化为日志中最后一个值之后的索引(图7中的11)。
> 如果跟随者的日志与领导者的日志不一致，则在下一个AppendEntries RPC中，AppendEntries一致性检查将失败。拒绝后，leader减少nextIndex并重试AppendEntries RPC。





## 5.4 Safety

该限制确保了任何给定任期的leader包含了在之前任期中提交的所有条目(图3中的leader完整性属性)。考虑到选举限制，我们可以制定更精确的提交规则。最后，我们给出了Leader完备性的证明草图，并展示了它如何导致复制状态机的正确行为。



### 5.4.1 Election restriction

这意味着日志条目只在一个方向上流动，从领导者到追随者，并且领导者永远不会覆盖其日志中的现有条目



Raft使用投票过程来阻止候选人赢得选举，除非其日志包含所有已提交的条目

> 如果follower的日志比candidate的新，拒绝投票
>
> Raft通过比较日志中最后一个entry的index和term来确定两个日志中的哪一个更最新。
>
> - term不同，term大的新。
> - term相同，index大的新。



### 5.4.2 Committing entries from previous terms

![image-20240807102938620](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240807102938620.png)



### 5.3 Safety argument

![image-20240807103947122](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240807103947122.png)

如何解决committed 过的log entry不会被覆盖？只靠term #好像不可靠？

<font color="red">还是TM没看懂</font>，好像又懂了<a id="疑问"></a>

> 比较最后一个entry，新的term还没有committed entry时会比较上一个entry ，则s5不会获得s3的选票
>
> > 最后一个entry的term相同，但s3的index多一个



已经提交的entry的index的位置无法被别的log entry覆盖



### 5.5 Follower and candidate crashes

如果跟随者或候选崩溃，那么发送到它的未来 RequestVote 和 AppendEntries RPC 将失败。Raft 通过无限期重试来处理这些故障；如果崩溃服务器重新启动，则 RPC 将成功完成。如果服务器在完成 RPC 但在响应之前崩溃，那么它会在重新启动后再次收到相同的 RPC。

### 5.6 Timing and availability



broadcastTime ≪ electionTimeout ≪ MTBF

在这个不等式中，广播时间是服务器向集群中的每个服务器并行发送rpc并获得响应的平均时间;选举超时是第5.2节中描述的选举超时;MTBF是单个服务器的故障之间的平均时间



广播时间和 MTBF 是底层系统的属性，而选举超时是我们必须选择的东西



# 7. Log compaction

如何解决log积累的问题，会有过时信息等

> snap shotting



状态机可以使用与快照相同的接口实现LSM树



图 12 显示了 Raft 中快照的基本思想。

![image-20240808112610803](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240808112610803.png)

覆盖committed entries还是state machine 已经应用的entry

快照还包含日志中的最新的配置，state machine state

一旦服务器完成snapshot，可能会删除最后一个索引之前的entries 并且 删除之前的 snapshot



尽管服务器通常独立获取快照，但领导者偶尔会向落后的追随者发送快照。

当leader将某个缓慢或者新的服务器要接受的entry快照，要将其丢弃时，直接发送快照



领导者使用一个名为InstallSnapshot的新RPC将快照发送到太落后的追随者;参见图13。

![image-20240808132044127](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240808132044127.png)



性能影响

- 什么时候进行快照？
  一个简单的策略是在日志以字节为单位达到固定大小时获取快照。

- 写入快照耗费大量性能
  （例如，Linux 上的 fork）可用于创建整个状态机的内存快照（我们的实现使用这种方法）



# 8. Client interaction

描述了客户端如何与Raft交互，包括客户端如何找到集群领导者以及Raft如何支持线性化语义



Clients of Raft send all of their requests to the leader.

当客户端首先启动时，它连接到随机选择的服务器。如果客户端的第一次选择不是领导者，服务器将拒绝客户端的请求并提供有关它听到的最新领导者的信息（AppendEntries 请求包括领导者的网络地址）。如果领导者崩溃，客户端请求将超时；然后，客户端再次尝试随机选择的服务器。



序列化语音信息
客户端为每个命令指定独特的序列号



读操作不需要在日志中，可能会读到过失的消息

Raft需要两个额外的预防措施来保证这一点，而不使用日志

> 只读操作可以在不将任何内容写入日志的情况下处理。然而，如果没有额外的措施，这将运行返回陈旧数据的风险，因为响应请求的领导者可能已经被它不知道的新领导者所取代。线性化读取不能返回陈旧的数据，
>
> 首先，领导者必须拥有提交条目的最新信息。领导者完整性属性保证领导者拥有所有提交的条目，但在其术语的开头，可能不知道这些条目是什么。要找出，它需要从其术语提交条目。Raft 通过让每个领导者在其术语的开头提交**空白的 no-op 条目来处理这一点**。<a href="#疑问">似乎与之前的疑问矛盾</a>
>
> [raft引入no-op解决了什么问题_raft 新leader 空op-CSDN博客](https://blog.csdn.net/zxpoiu/article/details/115464818)
>
> 
>
> 其次，领导者必须在处理只读请求之前检查它是否已被定义（如果选择了最近的领导者，则其信息可能会过时）。Raft 通过在响应只读请求之前，让**领导者与大多数集群交换心跳消息来处理**这一点。或者，leader 可以依靠心跳机制来提供租约 [9] 的一种形式，但这将依赖于安全性的时间（它假设时钟倾斜有界）。



# 9.Implementation and evaluation

