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

注意这里并没有快照，index直接为log的索引， 没有偏移量，为了以后考虑，直接设为0也可以

## Start

使用 Raft 的服务（例如 k/v 服务器）想要就下一个要附加到 Raft 日志的命令启动协议。需要从这里获取**entry，即command**

如果此服务器不是领导者，则返回 false。否则启动协议并立即返回。

无法保证此命令一定会提交到 Raft 日志，因为领导者可能会失败或输掉选举。

即使 Raft 实例已被终止，此函数也应该正常返回。
第一个返回值是命令提交后将出现的索引。第二个返回值是当前任期。如果此服务器认为自己是领导者，则第三个返回值为 true。

- [x] 首先判断是否leader。是则继续，否则直接返回
- [x] 封装后加入log

> [!NOTE]
>
> 只有这个消息提交时，才返回index

```go
func (rf *Raft) Start(command interface{}) (index int, term int, isLeader bool) {
	rf.mu.Lock()
	defer rf.mu.Unlock()

	//首先判断leader
	if !rf.isLeader {
		return
	}
	isLeader = true

	//DPrintf("command:%v", command)

	////检测是否已经提交
	//if command == rf.log[rf.committedIndex].Command {
	//	index = rf.committedIndex
	//	return
	//}

	rf.matchIndex[rf.me] = rf.nextIndex[rf.me]
	//			1					1
	//DPrintf("matchIndex:%v", rf.matchIndex)

	index = rf.nextIndex[rf.me]
	term = rf.currentTerm
	//			1
	//DPrintf("NextIndex:%v", rf.nextIndex)

	rf.nextIndex[rf.me]++
	//        2

	//DPrintf("%v", command)

	entry := Entries{
		Command: command,
		Index:   index,
		Term:    rf.currentTerm,
	}

	//将新的entry加入log
	rf.log = append(rf.log, entry)
	//DPrintf("leaderLog:%v", rf.log)
	// Your code here (3B).

	return
}
```



## Make

applyCh 是测试人员或服务期望 Raft 发送 ApplyMsg 消息的通道。Make() 必须快速返回，因此它应该为任何长时间运行的工作启动 goroutines。

- [x] 新增apply的go routine<a id="apply"></a><a href="##rf.apply()">`rf.apply`</a>
- [x] 


```go
func Make(peers []*labrpc.ClientEnd, me int, persister *Persister,
	applyCh chan ApplyMsg) *Raft {

	nextIndex := make([]int, len(peers))
	for i, _ := range nextIndex {
		nextIndex[i] = 1
	}
	matchIndex := make([]int, len(peers))
	log := make([]Entries, 0)

	rf := &Raft{
		peers:          peers,
		persister:      persister,
		me:             me,
		votedFor:       -1,
		electedTimeOut: time.Millisecond * 400, //设定心跳超时时间,开始选举
		lastRevTime:    time.Now(),             //初始化为设置当前时间
		currentTerm:    0,
		isLeader:       false,
		nextIndex:      nextIndex,
		matchIndex:     matchIndex,
		committedIndex: 0,
		log:            log,
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

- [x] ~~需要检查自己的身份? 每个peer都要将commited的上传~~
  ~~leader需要确认，follower需要吗？~~
  不对，leader确认有commited之后，告知follower，自己再应用
  只有leader需要确认，follower只要发现committed大于lastApplied就可以应用
  
  > 需要
  
- [x] 从matchindex中，如果大多数比自己的commitedIndex大，则将一个放入applyCh



如何检测新的commited entry？

> matchIndex？



rf从哪里获取entry？

> start函数



```go
func (rf *Raft) apply(applyCh chan ApplyMsg) {
	for rf.killed() == false {

		//leader角度
		for rf.isLeader && rf.killed() == false {
			rf.mu.Lock()
			//me := rf.me
			committedIndex := rf.committedIndex
			matchIndex := rf.matchIndex
			th := len(rf.peers) / 2
			lastApplied := rf.lastApplied
			rf.mu.Unlock()

			counts := 0
			//DPrintf("id:%v,committedIndex:%v", me, committedIndex)
			for _, v := range matchIndex {
				if committedIndex != -1 && v > committedIndex {
					counts++
				}
			}

			applyMsg := ApplyMsg{
				CommandValid: true,
				//Command:      rf.log[committedIndex].Command,
				CommandIndex: lastApplied + 1,
			}
			if counts > th {
				rf.mu.Lock()
				rf.committedIndex++
				DPrintf("leader,id:%v,log:%v,committedIndex:%v", rf.me, rf.log, rf.committedIndex)
				initIndex := rf.log[0].Index
				applyMsg.Command = rf.log[committedIndex+1-initIndex].Command
				//DPrintf("id:%v,commandIndex:%v", rf.me, committedIndex+1)

				//DPrintf("state:%v,一条消息过半认同", rf.isLeader)
				applyCh <- applyMsg

				rf.lastApplied++
				rf.mu.Unlock()
			}
			time.Sleep(30 * time.Millisecond)
		}
		//follower角度
		for !rf.isLeader && rf.killed() == false {
			rf.mu.Lock()
			//DPrintf("follower:%v try apply", rf.me)
			committedIndex := rf.committedIndex
			lastApplied := rf.lastApplied
			log := rf.log

			if committedIndex > lastApplied {
				//1					0

				DPrintf("follower,id:%v,log:%v,committedIndex:%v", rf.me, rf.log, rf.committedIndex)
				iniIndex := 1
				//DPrintf("log:%v", log)
				if len(log) != 0 {
					iniIndex = rf.log[0].Index
				}
				//DPrintf("id:%v,commandIndex:%v", rf.me, committedIndex)

				//DPrintf("commitedIndex:%v", committedIndex)
				applyMsg := ApplyMsg{
					CommandValid: true,
					Command:      log[lastApplied+1-iniIndex].Command,
					CommandIndex: lastApplied + 1,
				}

				//DPrintf("state:%v,leader确认commited，apply", rf.isLeader)
				applyCh <- applyMsg

				rf.lastApplied++

			}
			rf.mu.Unlock()
			time.Sleep(30 * time.Millisecond)
		}
		time.Sleep(20 * time.Millisecond)
	}
}
```

## HeartBeats

- [x] 需要收集append的结果

- [x] 没有成功的append需要针对性地在下一次重发

  > 超时的数据不变，下次重复发送

- [x] 不能卡在没有结果上，定时下次重传，修改<a href="##sendAppendEntries">sendAppendEntries</a>

- [x] 根据不同的peer匹配不同的index等

  > 都从rf的nextIndex参数传，发现对不上，则减小一位

- [x] 给args传递正确的数据，并根据reply调整下次传递的参数

  > ```go
  > type AppendEntriesArgs struct {
  >     Term         int
  >     LeaderId     int
  >     PrevLogIndex int
  >     PrevLogTerm  int
  >     Entries      []Entries
  >     LeaderCommit int
  > }
  > ```
  >
  > PrevLogIndex怎么获取？
  > matchIndex-1
  >
  > 
  >
  > entries怎么设置？
  > commitedIndex和nextIndex进行对比
  >
  > 
  >
  > raft中的nextIndex[]和matchIndex[]分别代表什么？
  >
  > > `nextIndex[]` 记录了每个从节点下一个应该发送的日志条目的索引。换句话说，领导者知道对于每个从节点，应该从日志的哪个条目开始发送以进行同步。
  > > `matchIndex[]` 记录了每个从节点已经复制了的日志的最高索引值。这有助于领导者知道哪些日志条目已经被大多数从节点复制，从而可以安全地提交这些日志

```go
func (rf *Raft) HeartBeats() {
	for rf.killed() == false {
		for rf.isLeader == true && rf.killed() == false {
			rf.mu.Lock()
			//nums := len(rf.peers)
			//resultCh := make(chan AppendEntriesResult, nums)
			peers := rf.peers
			me := rf.me
			DPrintf("me:%v", me)
			currentTerm := rf.currentTerm
			matchIndex := rf.matchIndex //0
			nextIndex := rf.nextIndex   //2

			//DPrintf("matchIndex:%v", matchIndex)

			//commitedIndex := rf.committedIndex
			entries := rf.log
			leaderCommit := rf.committedIndex
			rf.lastRevTime = time.Now()
			iniIndex := 0
			if len(entries) != 0 {
				iniIndex = entries[0].Index //1
			}
			rf.mu.Unlock()

			//DPrintf("entries:%v", entries)

			for i, _ := range peers {
				if i != me {
					go func(i int) {
						args := &AppendEntriesArgs{
							Term:         currentTerm,
							LeaderId:     me,
							PrevLogIndex: nextIndex[i] - 1, //1
							LeaderCommit: leaderCommit,
						}
						//DPrintf("matchIndex:%v", matchIndex)
						//DPrintf("nextIndex:%v", nextIndex)
						prevLogTerm := 0

						//如果有新消息，log一定存在 全是leader信息
						//if matchIndex[me] > commitedIndex && len(entries) != 0 {
						if nextIndex[i] < matchIndex[me]+1 && len(entries) != 0 { //有过start的leader

							//考虑消息累计，leader有多个还没发给follower
							//如何判断是别人的全场第一条：
							//nextIndex是否为1，已经判断过leader是否有内容
							if nextIndex[i] == 1 { //第一条
								args.Entries = entries
							} else if nextIndex[i] == 0 { //全覆盖
								args.Entries = entries
								prevLogTerm = -1
							} else { //不止一条的话传follower没有的
								//这里要使用nextIndex了
								DPrintf("id:%v,nextIndex:%v,initIndex:%v", i, nextIndex[i], iniIndex)
								args.Entries = entries[nextIndex[i]-iniIndex:] //2-1

								//如果新的leader，match清零怎么办
								prevLogTerm = entries[nextIndex[i]-iniIndex-1].Term
							}

							//if len(entries) == 1 { //只有一条的话肯定就是这一条
							//	//leader只有一条，则全场第一条
							//	args.Entries = entries
							//	//prevLogTerm = entries[0].Term
							//	//prevLogTerm = 0
							//
							//} else { //不止一条的话传follower没有的
							//	//这里要使用nextIndex了
							//	args.Entries = entries[nextIndex[i]-iniIndex:] //2-1
							//
							//	//如果新的leader，match清零怎么办
							//	prevLogTerm = entries[nextIndex[i]-iniIndex-1].Term
							//}
						}

						//即使没有新消息，也应该根据follower的nextIndex和自己的matchIndex来发送follower没有的日志
						//if nextIndex[i]<matchIndex[me]-1 {
						//
						//}
						if matchIndex[me] != 0 { //没有start过的新leader

						}
						//新继任的leader也应该同步

						args.PrevLogTerm = prevLogTerm
						reply := &AppendEntriesReply{}

						//DPrintf("prev:(%v,%v)", args.PrevLogIndex, args.PrevLogTerm)
						//DPrintf("args entry:%v", args.Entries)
						//ok := rf.sendAppendEntries(i, args, reply)
						rf.sendAppendEntries(i, args, reply)

						// 将 reply 的 success 状态发送到 channel 中
						//resultCh <- AppendEntriesResult{
						//	TimeOut: ok,
						//	Success: reply.Success,
						//	Number:  reply.Term,
						//}
					}(i)

				}
			}
			// 收集所有协程的结果
			//stop := false
			//
			//for j := 0; j < nums-1; j++ {
			//	result := <-resultCh
			//	if !result.Success && !result.TimeOut { //说明自己的term小于被请求对象
			//		stop = true
			//		//DPrintf("%v恢复follower,", rf.me)
			//		rf.mu.Lock()
			//		rf.isLeader = false
			//		rf.currentTerm = max(result.Number, rf.currentTerm)
			//		rf.lastRevTime = time.Now()
			//		rf.mu.Unlock()
			//		break
			//	}
			//}

			//if stop {
			//	break
			//}
			time.Sleep(40 * time.Millisecond)
		}
		time.Sleep(10 * time.Millisecond)
	}
}
```



## sendAppendEntries

- [x] 等待10毫秒，没有结果的话返回false

- [x] 对返回值进行处理，更新matchIndex和nextIndex


```go
func (rf *Raft) sendAppendEntries(server int, args *AppendEntriesArgs, reply *AppendEntriesReply) bool {
	/*	DPrintf("%v给%v发送AppendEntries", rf.me, server)

		tim := make(chan bool)

		go func() {
			time.Sleep(10 * time.Millisecond)
			tim <- false
		}()
		tim <- rf.peers[server].Call("Raft.AppendEntries", args, reply)

		return <-tim*/
	//DPrintf("%v给%v发送AppendEntries", rf.me, server)

	rf.mu.Lock()
	defer rf.mu.Unlock()

	// 创建一个结果通道
	resultChan := make(chan bool)

	// 启动一个Goroutine进行RPC调用
	go func() {
		ok := rf.peers[server].Call("Raft.AppendEntries", args, reply)
		resultChan <- ok
	}()

	select {
	case result := <-resultChan:
		DPrintf("%vsendAppendEntries有结果", server)
		//成功的话更新matchindex表
		if reply.Success {
			if len(args.Entries) != 0 {
				rf.matchIndex[server] = args.Entries[len(args.Entries)-1].Index
				rf.nextIndex[server] = args.Entries[len(args.Entries)-1].Index + 1
			}
		} else {
			if rf.currentTerm < reply.Term { //leader过时导致失败
				rf.isLeader = false

				DPrintf("id%v被发现超时，term由%v到%v", rf.me, rf.currentTerm, reply.Term)
				rf.currentTerm = reply.Term
				rf.log = rf.log[:rf.committedIndex]
			} else { //其他失败
				DPrintf("id:%v,减nextIndex", server)
				rf.nextIndex[server]--
			}
		}

		return result // RPC调用成功，返回结果
	case <-time.After(10 * time.Millisecond):
		DPrintf("%vsendAppendEntries超时", server)
		return false // 超时，返回false
	}
}
```

## AppendEntries

需要的igure2的指导：

- [x] ### 1. **接收请求**

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

- [x] ### 2. **步骤A: 任期检查**

- 如果收到的`term`比当前节点的任期号小，说明这是一个旧的`AppendEntries`请求，因此拒绝这个请求并返回`false`。

- [x] ### 3. **步骤B: 日志一致性检查**

- 检查本地日志在`prevLogIndex`位置处的条目是否与`prevLogTerm`匹配：
  - 如果不匹配，说明当前Follower的日志与Leader不一致，则拒绝该请求并返回`false`。
  - 如果匹配，则进入下一步。

- [x] ### 4. **步骤C: 日志复制**

- 删除本地日志中从`prevLogIndex+1`位置开始的所有条目，然后将`entries[]`中的新日志条目附加到本地日志中。
- 这样可以确保Follower与Leader的日志保持一致。

- [x] ### 5. **步骤D: 更新commitIndex**

- 如果`leaderCommit`大于当前Follower的`commitIndex`，那么Follower将其`commitIndex`更新为`leaderCommit`和新日志条目最后一个索引值中的较小值。
- 这一步确保了Follower的提交进度与Leader保持一致。

- [x] ### 6. **返回结果**

- 如果以上所有检查和操作都成功执行，Follower返回`true`，表示该`AppendEntries RPC`请求处理成功。

---

```go
func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
	rf.mu.Lock()
	defer rf.mu.Unlock()
	reply.Success = true
	rf.lastRevTime = time.Now()

	//判断是否leader过时
	//if args.Term < rf.currentTerm && args.LeaderCommit < rf.committedIndex {
	if args.Term < rf.currentTerm {
		//DPrintf("Args Term:%v", args.Term)
		//DPrintf("currentTerm:%v", rf.currentTerm)
		DPrintf("ID:%v,leader过时消息护着index小，follower当前term：%v，candidate term:%v", args.LeaderId, rf.currentTerm, args.Term)
		reply.Success = false
		reply.Term = rf.currentTerm
		return
	}

	//entry不为空，说明是正式的leader发送的有消息的心跳，否则为上位宣称
	//或者二者刚刚同步，本次心跳没有新消息
	if len(args.Entries) != 0 {
		DPrintf("收到有效内容")

		//正常消息，非第一条
		if len(rf.log) != 0 && args.PrevLogTerm != -1 {

			iniIndex := rf.log[0].Index //1

			//推算prevLogIndex对应的entry的实际log位置
			order := args.PrevLogIndex - iniIndex
			//0			1			1

			//检查prevLogIndex
			//if order < 0 || order >= len(rf.log) || rf.log[order].Term != args.PrevLogTerm {
			if order < 0 || order >= len(rf.log) || rf.log[order].Term != args.PrevLogTerm { //不匹配问题，不是要减next的地方
				reply.Success = false
				return
			}

			//修剪log
			rf.log = append(rf.log[:order+1], args.Entries...)
			DPrintf("received1,id:%v", rf.me)

		} else { //全场第一条消息

			////prevLogIndex为0，则合理，否错错误
			//if args.PrevLogIndex != 0 {
			//	reply.Success = false
			//	return
			//}
			//直接保留所有log
			rf.log = args.Entries
			DPrintf("received2,id:%v", rf.me)
		}

		//修改commitedIndex
		//如果没错，log已经有内容
		if args.LeaderCommit > rf.committedIndex {
			rf.committedIndex = min(args.LeaderCommit, rf.log[len(rf.log)-1].Index)
			DPrintf("follower,1committedIndex:%v", rf.committedIndex)
		}

	} else if args.LeaderCommit != -1 { //正常领导者的心跳
		if len(rf.log) != 0 {
			if args.LeaderCommit > rf.committedIndex && args.Term == rf.log[len(rf.log)-1].Term {
				rf.committedIndex = min(args.LeaderCommit, rf.log[len(rf.log)-1].Index)
				DPrintf("follower,2committedIndex:%v", rf.committedIndex)
			}
		} else if args.LeaderCommit > rf.committedIndex {
			rf.committedIndex = 0
			DPrintf("follower,3committedIndex:%v", rf.committedIndex)
		}
	}

	//同步term
	rf.currentTerm = args.Term
	rf.isLeader = false
	//重置心跳时间和是否投票
	rf.lastRevTime = time.Now()
	rf.votedFor = -1
	//DPrintf("%v成为follower", rf.me)
}
```



## 其他函数等，源码链接

这里不再进行赘述，放下[github源代码地址]([kingsill/RPC (github.com)](https://github.com/kingsill/RPC)),需要者自取即可

博主这里是自己搞得，可能比别人写的垃圾很多，轻喷

## index问题等一系列东西，做的过程中简单记录，大家来看应该无意义，附上其他人比较好的记录

### 别人帖子

> [MIT 6.824 Lab 2: Raft 实验 | Ray's Blog (rayzhang.top)](https://blog.rayzhang.top/2022/11/09/mit-6.824-lab2-raft/index.html#调试技巧)

leader：
 nextIndex[]		用于记录送给各follower的index		初始为leader的最新的log的index+1
 matchIndex[]		每个follower的最新的log的index		初始为0

all				
 lastApplied		给状态机的最后一条指令的index				初始0 ，单调增
 committedIndex	已知的最高的committed的index				初始0



1. leader收到第一条消息，用自己的nextIndex作为entey的index，在把自己的nextIndex+1
   同时把自己的matchIndex置为nextIndex，新消息检查

2. leader在心跳中检查是否有新消息，通过自己的matchIndex和committedIndex进行对比，由于只在start中进行自己的nextIndex修改，所以只要自己的matchIndex大于commitedIndex即有新消息

3. 在args填充信息后发送

   > leader刚上任时和普通心跳没有新消息的区别|
   >
   > > entries都为空
   > >
   > > 刚上任的宣称消息中的leadercommited改为-1
   > >
   > > 普通心跳包正常
   >
   > 没有消息的心跳包的任务；
   >
   > 1. 重置超时时间
   > 2. 同步commited信息

4. AppendEntries处理

   > 即使arg没有entry，是否要检查prevIndex等

5. leader的heartBeats（sendAppendEntries）再进行处理

   > - 超时
   >   直接结束，不进行处理
   > - 未超时
   >   - 成功
   >     - 有新消息
   >       matchIndex更新
   >       nextIndex更新
   >     - 无新消息
   >       不更新
   >   - 失败
   >     NextIndex-1



apply协程

- follower
  1. command



新的leader，match清零，prevLogTerm怎么算？
prevLogIndex 也是应该由 nextIndex来算 ，next是下一条日志的index，所以prev就是next-1



没有新消息的时候是否也能更新nextIndnex



如何确保follower Apply的log是更新过的呢？

> leader确认后会更新commit

> 从leader回到follower需要采取什么措施
>
> > 分区导致问题，单一分区的leader收到消息后堆积在自己的日志内
> >
> > 删除不是committed的部分？



---

received1 修剪log

received2 覆盖log

### testBackUp

创建5个服务器



![image-20240826133845285](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240826133845285.png)

![image-20240826133830310](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-20240826133830310.png)

append 失败可能性：

1. 过时 term小 或者 committedIndex小

2.  上位宣称：这个不会失败

   > 只需要更新term，自身isleader否，更新心跳，投票重置

3. leader有，follower没有
   next - -

4. 冲突，相同index，不同term，不删除，next - -



当leader发现某一个next为0后，直接发送自己的全部log

---

当前问题

例

| 4                           | 0                          | 1                       |
| --------------------------- | -------------------------- | ----------------------- |
| 第一次的leader，被断开      | 旧follower，一直被关       | 一直跟随，被断开        |
| term小，1                   | 断开后自增 term，term大，7 | term，2                 |
| 第一个10个随机数，index，11 | 但是没有消息，index小，1   | index收第二次随机数，11 |
| committed  1                | committed  1               | committed  11           |



如何处理不一致问题，4和1的消息不一致，4的未commit，1的commit大

​		

nextIndex大于follower所有的

```
leader,id:4,log:[{4678004126722157505 1 1}],committedIndex:1
2024/08/29 16:25:18 follower,id:0,log:[{4678004126722157505 1 1}],committedIndex:1
2024/08/29 16:25:18 follower,id:1,log:[{4678004126722157505 1 1}],committedIndex:1
2024/08/29 16:25:18 follower,id:3,log:[{4678004126722157505 1 1}],committedIndex:1
2024/08/29 16:25:18 follower,id:2,log:[{4678004126722157505 1 1}],committedIndex:1

2024/08/29 16:25:18 断开id：1,2,3连接
2024/08/29 16:25:18 开始给leader4，发送10个随机数

2024/08/29 16:25:21 每一个都断开连接
2024/08/29 16:25:21 恢复连接：4 0 1
2024/08/29 16:25:21 发送10随机数

2024/08/29 16:25:20 leader,id:2,log:[{4678004126722157505 1 1} {8938097423806100375 2 2} {723196532827361542 2 3} {18442494331360046 2 4} {5649708825675347494 2 5} {5043821671661526099 2 6} {984076030453990503 2 7} {2555064003070238563 2 8} {4577031850588410304 2 9} {474870608042202811 2 10} {3407621702695960717 2 11}],committedIndex:11
2024/08/29 16:25:20 follower,id:3,log:[{4678004126722157505 1 1} {8938097423806100375 2 2} {723196532827361542 2 3} {18442494331360046 2 4} {5649708825675347494 2 5} {5043821671661526099 2 6} {984076030453990503 2 7} {2555064003070238563 2 8} {4577031850588410304 2 9} {474870608042202811 2 10} {3407621702695960717 2 11}],committedIndex:11
2024/08/29 16:25:20 follower,id:1,log:[{4678004126722157505 1 1} {8938097423806100375 2 2} {723196532827361542 2 3} {18442494331360046 2 4} {5649708825675347494 2 5} {5043821671661526099 2 6} {984076030453990503 2 7} {2555064003070238563 2 8} {4577031850588410304 2 9} {474870608042202811 2 10} {3407621702695960717 2 11}],committedIndex:11

 follower,id:0,log:[{4678004126722157505 1 1} {2858403243448308923 1 2} {5374456348256256886 1 3} {2411776181554754243 1 4} {2594279163070091109 1 5} {1446728681006973319 1 6} {4312261448858846746 1 7} {4585759772685965362 1 8} {6390925677967595987 1 9} {1637295433917407814 1 10} {8198173872564129616 1 11}],committedIndex:11

```



上传的未committed
