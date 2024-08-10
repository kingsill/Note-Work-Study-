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



## 各操作步骤

### 1. READ

1. **client** filename offset(range) -> **master**
2. **master** chunk handle & list of server  -> **client**
3. **client** -> **chunk server**

## 2. WRITE(append)

**No Primary**? **Master**

1. find the up-to-date replica

   > Master保存版本号在自己本地，防止有最新chunk的server离线

2. **pick a Primary**,others as Secondary

   > Primary 来接受client操作，同步到其他Secondary

3. Increament version number

4. TELL P,S.v# —–give Primary **LEASE**

   > 防止脑裂SPLIT BRAIN
   >
   > 租期内即使失效不重新指定PRIMARY

5. save these

 

**Primary**

1. pick a offset -> Secondaries (all replicas told to write at offset)

2. 数据缓冲

3. if all S “yes”, P-> “success” C  ->4
   else “no” ->重新请求追加，跳过4？？放弃本次

   > Primary一定要成功

4. 主节点开始写入







