# GFS

## 绪论

可拓展分布式文件系统

- 容错
- 高聚合性能

需要实现的关键点：

1. 错误恢复

2. 对文件操作的审视 I/O操作的定义
   文件大小较大

3. 对文件的操作通常都是读取，或者是（写入）插入在文件的末尾

4. 应用程序和文件系统API一同设计

   ## 概述

   ### 假设

   - 文件读取

     - 大型流式读取

     - 小型随机读取

   - 写入

   - 系统具有原子性

   - 高持续带宽 大于 低延迟

### 接口

create

delete 

open 

close

read 

write



snapshot

record

###  结构

![image-20240724141750052](Article GFS.assets/image-20240724141750052.png)



- <font color="red">master</font>	一个
  记录文件树,存储 元数据

  > namespcae
  >
  > access control information
  >
  > mapping from files to chunks
  >
  > current locations of chunks

  系统范围内活动

  通过heartBeat消息收集chunkserver状态并分配任务

- <font color="red">chunkserver</font> 多个
  保存文件
  写入读取依靠 **句柄**chunk handle **范围** byte range



**文件chunk的标识 ****chunk handle** 是创建时在 主服务器分配的，

文中默认保存 3 个副本



client与master和chunkserver都有沟通，三者直接联系 ，没有中间媒介

## 2.4 单个 master

不参与读写，master不是瓶颈

客户端client 向 master 询问 应该请求哪个 chunkserver

### 2.5 chunk Size 块大小

文中为64MB

**Big Size的优势**

- 减少了client和master交互的次数
- client在一个大的chunk上可能进行更多操作，再减少交互
- 节省了master的元数据存储，方便存储到内存上

**big Size 的劣势**

- 出现热点问题hot spots

### 2.6 Metadata 

存储三种主要信息

1. 文件、块名称 namespace
2. 文件到块的映射 mapping
3. 块副本的位置 locations



所有元数据都保存在主内存中



前两种类型(名称空间和文件到块映射)也通过将更改记录到存储在主服务器本地磁盘上并复制到远程机器上的操作日志中来保持持久化。

主服务器不持久化存储块位置信息。相反，它会在主启动时以及每当有chunkserver加入集群时询问每个chunkserver的块。

> 这样可以在集群增加或减少chunkserver时不用和master保持一致，master直接查询即可

#### operation log

它不仅是元数据的唯一持久记录，而且还用作定义并发操作顺序的逻辑时间线。

在元数据更改持久化之前，不能使更改对客户机可见。

在日志记录刷新到磁盘后才相应client



定时或者在一定操作量后**自动保存检查点**，方便回放操作

checkpoints存储形式B-tree

### 2.7 Consistency Model

#### Guarantees

文件命名空间namespaces为原子操作

![image-20240725164322011](Article GFS.assets/image-20240725164322011.png)

文件区域状态有consistent和inconsistent

并发成功的突变使区域未定义，但保持一致:所有客户端都看到相同的数据，但它可能不反映任何一个突变所写的内容。

相同顺序执行操作
使用块版本号来检测是否错过突变

## System Interactions

三种操作

- data mutations 数据突变
- atomic record append 原子记录追加
- snapshot 快照

### 3.1 Leases and Mutation Order

突变是一种改变数据块内容或元数据的操作，例如写操作或追加操作。在所有的chunk副本运行

> 我们使用leases租约来维护副本间一致的mutations顺序

主服务器将块租期授予其中一个副本，我们称之为primary

