[The Go Memory Model - The Go Programming Language](https://go.dev/ref/mem#introduction)

Go内存模型指定了一些条件，在这些条件下，在一个运行例程中对变量的读取可以保证观察到在另一个运行例程中对同一变量的写入所产生的值。 

## 建议

避免数据竞争data race

1. channel
2. other synchronization primitives
   - sync
   - sync/atomic

## 内存模型

内存模型—程序执行—goroutine执行—内存操作

A *memory operation* is modeled by four details:

- its kind, indicating whether it is an ordinary data read, an ordinary data write, or a *synchronizing operation* such as an atomic data access, a mutex operation, or a channel operation,
- its location in the program,
- the memory location or variable being accessed, and
- the values read or written by the operation.





## go routine with performance

实际上更多的是易于 解释

1. closures
   可以使用其所在作用域的变量

2.condition variable
在持有锁的过程进行cond.broadcast等

3. channel
   看作同步通信机制 
   - 无缓存的channel是不能存储的，直到有接收者之前都阻塞
     接受和发送缺一都会死锁
   - 有缓存的可以存储一定的数据

 