# MapReduce

每个 `worker `进程需完成以下工作：
向 master 进程请求 task，从若干文件中读取输入数据，执行 task，并将 task 的输出写入到若干文件中。

`master `进程除了为 worker 进程分配 task 外，还需要检查在一定时间内（本实验中为 10 秒）每个 worker 进程是否完成了相应的 task，如果未完成的话则将该 task 转交给其他 worker 进程。



## worker

worker函数包含map和redicef两个功能
需要知道自己执行哪个功能

### map

### reduce

