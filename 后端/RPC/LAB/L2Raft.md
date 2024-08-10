需要实现
saving persistent state and reading it after a node fails and then restarts
log compaction / snapshotting (Section 7)

不需要
section6—change membership

[Students' Guide to Raft :: Jon Gjengset (thesquareplanet.com)](https://thesquareplanet.com/blog/students-guide-to-raft/)

[nil.csail.mit.edu/6.824/2020/labs/raft-locking.txt](http://nil.csail.mit.edu/6.824/2020/labs/raft-locking.txt)

[nil.csail.mit.edu/6.824/2020/labs/raft-structure.txt](http://nil.csail.mit.edu/6.824/2020/labs/raft-structure.txt)



## 2A task

- leader election
- heartbeats （AppendEntries RPCs with no entry）



### 遇到的困难

1.不知道怎么处理心跳的问题

> 网上大多使用**time.timer**进行计时，但是在lab的hint中已经写不推荐使用。（虽然这个方法也不会）
>
> 使用**time.sleep**该怎么解决，之前没用使用过，不知道如何**重置**

<a href="#心跳问题">解决</a>

### 解决困难

1. 心跳问题<a id="心跳问题"></a>

   > 虽然不推荐使用，这里还是学习以下time.timer怎么使用
   >
   > [wlwilliamx | Go Timer 详解以及 Reset 和 Stop 的详细用法](https://wlwilliamx.github.io/posts/go-timer-详解以及-reset-和-stop-的详细用法#reset-重置-timer)

   [Raft 算法选主详解与复现, 完成 MIT 6.824(6.5840) Lab2A - 掘金 (juejin.cn)](https://juejin.cn/post/7286133122211037244#heading-21)当然，在别人的尝试里面time.sleep的时间戳的做法会出错，这里还需要实验验证

   > 正经版，time.sleep相关的重置
   >
   > 

