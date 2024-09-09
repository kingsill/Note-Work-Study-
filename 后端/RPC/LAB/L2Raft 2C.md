# 2C Task: Persistence



## Task

Complete the functions `persist()` and `readPersist()` in `raft.go` by adding code to save and restore persistent state. You will need to encode (or "serialize") the state as an array of bytes in order to pass it to the `Persister`. Use the `labgob` encoder; see the comments in `persist()` and `readPersist()`. `labgob` is like Go's `gob` encoder but prints error messages if you try to encode structures with lower-case field names. For now, pass `nil` as the second argument to `persister.Save()`. Insert calls to `persist()` at the points where your implementation changes persistent state. Once you've done this, and if the rest of your implementation is correct, you should pass all of the 3C tests.
完成功能 `persist()` 和 `readPersist()` 在 `raft.go` 通过添加代码来保存和恢复持久状态。您需要将状态编码（或“序列化”）为字节数组，以便将其传递给 `Persister` 。使用 `labgob` 编码器；请参阅中的评论 `persist()` 和 `readPersist()` 。 `labgob` 就像 Go 的 `gob` 编码器，但如果您尝试使用小写字段名称对结构进行编码，则会打印错误消息。现在，通过 `nil` 作为第二个参数 `persister.Save()` 。将呼叫插入到 `persist()` 在您的实现更改持久状态的点。完成此操作后，如果其余的实现正确，您应该通过所有 3C 测试。

## hint 提示

> [Hint]
>
> - The 3C tests are more demanding than those for 3A or 3B, and failures may be caused by problems in your code for 3A or 3B.
>   3C 测试比 3A 或 3B 测试要求更高，失败可能是由 3A 或 3B 代码中的问题引起的。



我们在实验中不将 state 写入到磁盘，而是保存到raft的persister对象中



传递的字段首字母大写



read和save的时机：

> 每次raft变化时save
> 当重启时read 



## 实现

### Persist

根据raft论文图2，需要持久化的字段为currentTerm、votedFor、log三个字段，那么这里就保存这三个

将这个函数放在改变raft的地方

```go
func (rf *Raft) persist() {
	// Your code here (3C).
	// Example:
	P := &NeedP{rf.currentTerm, rf.votedFor, rf.log}
	w := new(bytes.Buffer)
	e := labgob.NewEncoder(w)
	e.Encode(P)
	raftstate := w.Bytes()
	rf.persister.Save(raftstate, nil)
}

```



### ReadPersist

读取后赋值raft

```go
func (rf *Raft) readPersist(data []byte) {
	if data == nil || len(data) < 1 { // bootstrap without any state?
		return
	}
	// Your code here (3C).
	//Example:
	r := bytes.NewBuffer(data)
	d := labgob.NewDecoder(r)
	var P NeedP
	d.Decode(&P)

	rf.currentTerm = P.CurrentTerm
	rf.votedFor = P.VotedFor
	rf.log = P.Log
}
```

---

问题：

1.  committedIndex等怎么识别



appendEntries中，follower是不是没更新前不能apply，否则可能将不正确的信息apply

> leader： 1 1 3
>
> follower：1 2
>
> follower： 1  1 3
>
> 使用follower自己的rf.match来判定，0未更新，1更新



更新全部时，内容发不全

```

```

目前问题：没法确保leader和follower的committed是一致的

？？？？？？？？？？？？？？？？

如何确保committed的entries不被覆盖

leader先用  ，follower没用就断开 leader冲突

> committed大于lastapplied
