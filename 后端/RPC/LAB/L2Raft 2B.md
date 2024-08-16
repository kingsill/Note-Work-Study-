# Raft 2B Task

**LOG**

>  Implement the leader and follower code to **append new log entries**

---

- 您的第一个目标应该是传递 `TestBasicAgree3B（）。` 首先实现 `Start（），`然后编写代码 通过 `AppendEntries` RPC 发送和接收新的日志条目， 如图 2 所示。发送每个新提交的条目 在每个对等体上的 `applyCh` 上。
- 您将需要实施选举 限制（论文第 5.4.1 节）。
- 您的代码可能具有重复检查某些事件的循环。 不要有这些循环 从那时起，连续执行而不停顿 会减慢您的实现速度，以至于无法通过测试。 使用 Go [的条件变量](https://golang.org/pkg/sync/#Cond)， 或插入`时间。睡眠（10 * 次。毫秒）`在每次循环迭代中。
- 为未来的实验室提供帮助，并编写（或重写）代码 这很清楚。如需想法，请重新访问我们的 [“指南”页面](https://pdos.csail.mit.edu/6.824/labs/guidance.html)，其中包含有关如何执行以下操作的提示 开发和调试代码。
- 如果测试失败，请查看 `test_test.go` 和 `config.go` 了解正在测试的内容。`config.go` 也 演示了测试人员如何使用 Raft API。

---

您需要一个单独的长期运行的 goroutine，在 applyCh 上按顺序发送已提交的日志条目。它必须是单独的，因为在 applyCh 上发送可能会阻塞；而且它必须是一个单独的 goroutine，否则可能难以确保按日志顺序发送日志条目。 推进 commitIndex 的代码需要启动 apply goroutine；为此，使用条件变量（Go 的 sync.Cond）可能是最简单的方法。

# 需要修改的函数

## Start

使用 Raft 的服务（例如 k/v 服务器）想要就下一个要附加到 Raft 日志的命令启动协议。需要从这里获取**entry，即command**

如果此服务器不是领导者，则返回 false。否则启动协议并立即返回。

无法保证此命令一定会提交到 Raft 日志，因为领导者可能会失败或输掉选举。

即使 Raft 实例已被终止，此函数也应该正常返回。
第一个返回值是命令提交后将出现的索引。第二个返回值是当前任期。如果此服务器认为自己是领导者，则第三个返回值为 true。

1. 首先判断是否leader。是则继续，否则直接返回
2. 封装后加入log

```go
func (rf *Raft) Start(command interface{}) (int, int, bool) {
	index := -1
	term := -1
	isLeader := true

	// Your code here (3B).

	return index, term, isLeader
}
```

```go
func (rf *Raft) Start(command interface{}) (index int, term int, isLeader bool) {
	rf.mu.Lock()
	defer rf.mu.Unlock()
	index = len(rf.log) + 1
	isLeader = rf.isLeader
	term = rf.currentTerm

	if !isLeader {
		return
	}

	entry := Entries{
		Command: command.(string),
		Index:   index,
		Term:    term,
	}

	//将新的entry加入log
	rf.log = append(rf.log, entry)
	// Your code here (3B).

	return
}
```



## Make

applyCh 是测试人员或服务期望 Raft 发送 ApplyMsg 消息的通道。Make() 必须快速返回，因此它应该为任何长时间运行的工作启动 goroutines。

- [ ] 新增apply的go routine<a id="apply"></a><a href="##rf.apply()">`rf.apply`</a>
- [ ] 

```go
func Make(peers []*labrpc.ClientEnd, me int, persister *Persister,
	applyCh chan ApplyMsg) *Raft {

	rf := &Raft{
		peers: peers,
		persister:persister,
		me: me,
		votedFor: -1,
		electedTimeOut: time.Millisecond * 300,//设定心跳超时时间,开始选举
		lastRevTime: time.Now(),	//初始化为设置当前时间
		currentTerm: 0,
		isLeader: false,
	}
	
	// Your initialization code here (3A, 3B, 3C).

	// initialize from state persisted before a crash
	rf.readPersist(persister.ReadRaftState())

	// start ticker goroutine to start elections

	go rf.ticker() //RequestVote RPC

	go rf.HeartBeats()

	return rf
}

```

```go
func Make(peers []*labrpc.ClientEnd, me int, persister *Persister,
	applyCh chan ApplyMsg) *Raft {

	rf := &Raft{
		peers:          peers,
		persister:      persister,
		me:             me,
		votedFor:       -1,
		electedTimeOut: time.Millisecond * 300, //设定心跳超时时间,开始选举
		lastRevTime:    time.Now(),             //初始化为设置当前时间
		currentTerm:    0,
		isLeader:       false,
	}

	// Your initialization code here (3A, 3B, 3C).

	// initialize from state persisted before a crash
	rf.readPersist(persister.ReadRaftState())

	// start ticker goroutine to start elections

	go rf.ticker() //RequestVote RPC

	go rf.HeartBeats()

	go rf.apply(applyCh)

	return rf
}
```



## applyMsg结构体

当每个 Raft 对等体意识到连续的日志条目被提交时，对等体应该通过传递给 Make() 的 applyCh 向同一服务器上的服务（或测试器）发送 ApplyMsg。将 CommandValid 设置为 true 以指示 ApplyMsg 包含新提交的日志条目。

所以说当有新的commited entry时，将其打包为applyMsg再发送到applyCh

因为follower也要对其服务进行提交，所以也要有这个长线服务，应该在<a href="#apply">make中新建一个go routine来完成这件事</a>

## rf.apply()

1. 不需要检查自己的身份
2. 当有新的commited entry时启动，将其打包为applyMsg再发送到applyCh



如何检测新的commited entry？

> 



rf从哪里获取entry？

> start函数



```go
```

## HeartBeats

- [ ] 需要收集append的结果
- [ ] 没有成功的append需要针对性地在下一次重发
- [ ] 不能卡在没有结果上，定时下次重传，修改<a href="##sendAppendEntries">sendAppendEntries</a>
- [ ] 根据不同的peer匹配不同的index等
- [ ] 给args传递正确的数据，并根据reply调整下次传递的参数

```go
```



## sendAppendEntries

- [x] 等待10毫秒，没有结果的话返回false



```go
func (rf *Raft) sendAppendEntries(server int, args *AppendEntriesArgs, reply *AppendEntriesReply) bool {
    DPrintf("%v给%v发送AppendEntries", rf.me, server)

    // 创建一个结果通道
    resultChan := make(chan bool)

    // 启动一个Goroutine进行RPC调用
    go func() {
       ok := rf.peers[server].Call("Raft.AppendEntries", args, reply)
       resultChan <- ok
    }()

    select {
    case result := <-resultChan:
       return result // RPC调用成功，返回结果
    case <-time.After(10 * time.Millisecond):
       return false // 超时，返回false
    }
}
```

## AppendEntries

需要的igure2的指导：

### 1. **接收请求**

- Follower节点收到来自leader的

  ```
  AppendEntries RPC
  ```

  请求。该请求中包含了以下信息：

  - `term`：Leader的任期号。
  - `leaderId`：Leader的ID。
  - `prevLogIndex`：新的日志条目紧接在此之前的日志条目的索引。
  - `prevLogTerm`：`prevLogIndex`条目的任期号。
  - `entries[]`：需要被存储的日志条目（可能为空，用于心跳）。
  - `leaderCommit`：Leader的已知已提交日志的索引。

### 2. **步骤A: 任期检查**

- 如果收到的`term`比当前节点的任期号小，说明这是一个旧的`AppendEntries`请求，因此拒绝这个请求并返回`false`。

### 3. **步骤B: 日志一致性检查**

- 检查本地日志在`prevLogIndex`位置处的条目是否与`prevLogTerm`匹配：
  - 如果不匹配，说明当前Follower的日志与Leader不一致，则拒绝该请求并返回`false`。
  - 如果匹配，则进入下一步。

### 4. **步骤C: 日志复制**

- 删除本地日志中从`prevLogIndex+1`位置开始的所有条目，然后将`entries[]`中的新日志条目附加到本地日志中。
- 这样可以确保Follower与Leader的日志保持一致。

### 5. **步骤D: 更新commitIndex**

- 如果`leaderCommit`大于当前Follower的`commitIndex`，那么Follower将其`commitIndex`更新为`leaderCommit`和新日志条目最后一个索引值中的较小值。
- 这一步确保了Follower的提交进度与Leader保持一致。

### 6. **返回结果**

- 如果以上所有检查和操作都成功执行，Follower返回`true`，表示该`AppendEntries RPC`请求处理成功。

---

```go
func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
	rf.mu.Lock()
	defer rf.mu.Unlock()
	reply.Success = true

	//判断是否leader过时
	if args.Term < rf.currentTerm {
		DPrintf("ID:%v,leader过时", args.LeaderId)
		reply.Success = false
		reply.Term = rf.currentTerm
		return
	}

	//log中第一个entry的Index
	iniIndex := rf.log[0].Index
	//推算prevLogIndex对应的entry的实际log位置
	order := args.PrevLogIndex - iniIndex
	//entry不为空，说明是正式的leader发送的有消息的心跳，否则为上位宣称
	if args.Entries != nil {
		//检查prevLogIndex
		if order < 0 || rf.log[order].Term != args.PrevLogTerm {
			reply.Success = false
			return
		}
	}

	//修剪log
	rf.log = append(rf.log[:order+1], args.Entries...)

	//修改commitedIndex
	if args.LeaderCommit > rf.committedIndex {
		rf.committedIndex = min(args.LeaderCommit, rf.log[len(rf.log)-1].Index)
	}

	//同步term
	rf.currentTerm = args.Term
	rf.isLeader = false
	//重置心跳时间和是否投票
	rf.lastRevTime = time.Now()
	rf.votedFor = -1
	DPrintf("%v成为follower", rf.me)
}

```

