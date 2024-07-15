# MapReduce

每个 `worker `进程需完成以下工作：
向 master 进程请求 task，从若干文件中读取输入数据，执行 task，并将 task 的输出写入到若干文件中。

`master `进程除了为 worker 进程分配 task 外，还需要检查在一定时间内（本实验中为 10 秒）每个 worker 进程是否完成了相应的 task，如果未完成的话则将该 task 转交给其他 worker 进程。



## worker

worker函数包含map和redicef两个功能
需要知道自己执行哪个功能



worker 请求获取任务 **GetTask**

> 任务设置为结构体，其中一个字段为任务类型
>
> > type Task struct{Type：0 1 }0为map，1为reduce  *const枚举*
>
> 一个为编号

如何获取任务？

GetTask请求coordinator的assignTask方法，传入为自己的信息（？？？），获得任务信息



> 自己当前的状态 空闲 忙碌



**`call（rpcname,args,reply）bool`的rpcname**对应**coordinator.go**中coordinator的相关方法

> ???没理解这个什么用
> GetTask中调用call来从coordinator获取任务信息？
>
> > 那么rpc.go来干什么



使用ihash(key) % NReduce为Map发出的每个KeyValue选择reduce任务号。随机选择序号
n个文件，生成m个不同文件                     n X m个 文件？？

> 保存多少个文件
>
> map得到[](key,val)
>
> 对结果ohashkey，写入到文件序号1-10
>
> 根据序号分配reduce任务
>
> 将结果写入到同一文件

### map

```go
mapf func(filename string, content string) []KeyValue
```

filename是传入的文件名
content为传入的文件的内容——传入前需读取内容

传出intermediate[] 产生文件名 mr-x-y

### reduce

```go
reducef func(key string, values []string) string)
```

这里的key对应ihash生成的任务号？？

## coordinator

分配任务
需要创建几个map worker—根据几个文件
几个 reduce worker—根据设置，这里为10

如何查询worker状态



## rpc

`coordinator`函数用来生成唯一id

 

