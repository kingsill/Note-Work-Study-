# 2C Task: Persistence



## Task

Complete the functions `persist()` and `readPersist()` in `raft.go` by adding code to save and restore persistent state. You will need to encode (or "serialize") the state as an array of bytes in order to pass it to the `Persister`. Use the `labgob` encoder; see the comments in `persist()` and `readPersist()`. `labgob` is like Go's `gob` encoder but prints error messages if you try to encode structures with lower-case field names. For now, pass `nil` as the second argument to `persister.Save()`. Insert calls to `persist()` at the points where your implementation changes persistent state. Once you've done this, and if the rest of your implementation is correct, you should pass all of the 3C tests.
完成功能 `persist()` 和 `readPersist()` 在 `raft.go` 通过添加代码来保存和恢复持久状态。您需要将状态编码（或“序列化”）为字节数组，以便将其传递给 `Persister` 。使用 `labgob` 编码器；请参阅中的评论 `persist()` 和 `readPersist()` 。 `labgob` 就像 Go 的 `gob` 编码器，但如果您尝试使用小写字段名称对结构进行编码，则会打印错误消息。现在，通过 `nil` 作为第二个参数 `persister.Save()` 。将呼叫插入到 `persist()` 在您的实现更改持久状态的点。完成此操作后，如果其余的实现正确，您应该通过所有 3C 测试。

## hint 提示

> [Hint]
>
> - The 3C tests are more demanding than those for 3A or 3B, and failures may be caused by problems in your code for 3A or 3B.
>   3C 测试比 3A 或 3B 测试要求更高，失败可能是由 3A 或 3B 代码中的问题引起的。

