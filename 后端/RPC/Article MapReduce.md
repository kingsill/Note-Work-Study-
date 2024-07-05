# MapReduce

1. introduction 绪论

​	**map** **reduce** 两个操作实现 通用的简单**接口**，实现大规模计算的并行和分布

2. Programming Model 程序模式

​	输入和输出都为 key/value 形式

2.1 example 例

​	输入---map---产生key/value---MapReduce---中间值分组---reduce---组合相同键的值，只产生0或一个输出值

> ​	一个key对应多个value

3. Implementation 实现
3.1 执行概述

![image-20240703164223099](C:\Users\wang2\AppData\Roaming\Typora\typora-user-images\image-20240703164223099.png)

3.2 数据结构

对每一个map或reduce任务，存储idle，in-progress，completed标志，和工作机器（非工作）标识