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

1. 不知道怎么处理心跳的问题

> 网上大多使用**time.timer**进行计时，但是在lab的hint中已经写不推荐使用。（虽然这个方法也不会）
>
> 使用**time.sleep**该怎么解决，之前没用使用过，不知道如何**重置**

<a href="#心跳问题">解决</a>

2. 为啥RequestVote RPC中Results包含term，有什么作用？

   > 在 `RequestVoteReply` 中，`term` 参数虽然是由 `follower` 返回给 `candidate` 的，但它仍然非常重要。原因如下：
   >
   > ### 1. **检测并纠正过期的候选者**
   >
   > 当一个 `candidate` 请求选票时，它可能处于一个较旧的 `term`。如果 `follower` 处于一个更新的 `term`（即 `follower` 的 `term` 大于 `candidate` 的 `term`），`follower` 会拒绝投票，并在 `RequestVoteReply` 中返回它的当前 `term`。
   >
   > - 当 `candidate` 收到这个带有较新 `term` 的 `RequestVoteReply` 时，它会意识到自己落后了，并更新自己的 `term` 为这个更大的值。根据 Raft 协议，一旦 `candidate` 发现有服务器的 `term` 比它自己大，它会立即放弃竞选，回退到 `follower` 状态。
   >
   > ### 2. **避免网络分区引发的过期领导问题**
   > 在分布式系统中，网络分区或其他原因可能导致某些服务器的 `term` 落后。通过让 `follower` 返回它的当前 `term`，可以防止过期的 `candidate` 继续尝试获取选票，避免产生多个领导者的情况，确保集群的正确性和一致性。
   >
   > ### 3. **同步集群状态**
   > 通过这种机制，所有服务器能够逐渐收敛到一致的 `term`，从而避免了因 `term` 不一致引发的潜在问题。在 Raft 中，`term` 是确保系统在面对网络分区或故障时仍然保持一致性和正确性的重要部分。
   >
   > ### 小结
   > 虽然 `candidate` 在发送 `RequestVote` 请求时有自己的 `term`，但 `follower` 返回的 `term` 提供了一种机制，使 `candidate` 能够发现并更新它的 `term`，从而避免产生多个领导者并确保集群的状态一致性。

### 解决困难

1. 心跳问题<a id="心跳问题"></a>

   > 虽然不推荐使用，这里还是学习以下time.timer怎么使用
   >
   > [wlwilliamx | Go Timer 详解以及 Reset 和 Stop 的详细用法](https://wlwilliamx.github.io/posts/go-timer-详解以及-reset-和-stop-的详细用法#reset-重置-timer)

   [Raft 算法选主详解与复现, 完成 MIT 6.824(6.5840) Lab2A - 掘金 (juejin.cn)](https://juejin.cn/post/7286133122211037244#heading-21)当然，在别人的尝试里面time.sleep的时间戳的做法会出错，这里还需要实验验证

   > 正经版，time.sleep相关的重置
   >
   > [Golang time.Sleep()函数及示例|极客教程 (geek-docs.com)](https://geek-docs.com/go-tutorials/go-examples/g_time-sleep-function-in-golang-with-examples.html)

查找资料过程中，还有time.after相关，可以直连channel，但是助教不推荐channel。不太明白



到底哪些函数需要锁？



在for 中使用闭包函数的传入问题，在使用过程中一直对同一follower发送投票请求

> 如下例：
> ```go
> func main() {
>     for i := 0; i < 3; i++ {
>         //i:=i
>         go func() {
>             fmt.Println(i)
>         }()
>     }
>     time.Sleep(time.Second)
> }
> 
> ```
>
> 他的输出实际为333，当闭包函数执行时，闭包中i已经被更新到3了
> 所以应该使用i：=i，如注释，以传递副本保证值的正确



### 重新整理问题

candidate和follower有什么本质区别？

心跳超时
大家都一样



 选举超时 
每次都初始化



子函数中创建的 goroutine **不会**随着子函数的结束而结束。goroutine 是独立执行的，并且在创建后，它的生命周期与创建它的函数无关。
