# Fault Tolerant Virtual Machinese



## Abstract

通过在另一个服务器上的备用虚拟机复制 主虚拟的的操作 来完成

## 1. INTRODUCTION

主备份方法

>  使用备份服务器接管主服务器

+

Statemachine Approach 状态机

> 从相同的状态启动，并保持请求一致



物理机实现状态机较为困难，但是 虚拟机 可以通过hypervisor实现所有信息的监控



容错的实现方式被称为 deterministic replay



对虚拟机的故障检测为 Fault-stop failures，

