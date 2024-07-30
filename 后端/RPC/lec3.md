# GFS

## 分布式

为什么困难？CAP[分布式之CAP原则详解_分布式cap-CSDN博客](https://blog.csdn.net/lixinkuan328/article/details/95535691)

- 为了获得几百台机器综合的**性能**
- Faults 总有机器会**出错**–**tolerance&Availability** 容错
- 可用性–数据复制**consistency** ->降低性能



strong consistency 像与一台服务器交互一样
weak consistency 允许返回错误的数据？

## GFS本身

1. Big,Fast
2. Global
3. Sharding
4. Automatic recovery



专为**大文件**读写设计
sequential access而不是 随机读写 
single master



Master Data

- filename->array of chunk handles(nv)
- handle-> list of chunk server(v) & version number(nv) & primary(v) & lease(v)      in RAM
- LOG,checkpoint  in DISK



## 步骤

### 1. READ





