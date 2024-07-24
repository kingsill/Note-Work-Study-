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