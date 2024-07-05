# LABVIEW

## 组成

1. 前面版
2. 程序框图
3. 连线板/图标

##  技巧

- ctrl+B 删除所有断线
- ctrl+H 显示即时帮助
- ctrl+E 显示程序框图

## 奈奎斯特定律

- 频率

  想要精确复现原始信号的频率：必须以至少原始信号最高频率两倍的采样率进行采样


- 波形

  想要精确复现原始信号的波形，必须以至少原始信号最高频率5-10倍采样率进行采样


# S7 1200

## 通信



三种通信方式：[Labview 三种方式通讯西门子PLC_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1JP411F78R/?spm_id_from=333.788&vd_source=d3dafb5faaa2391d25c0cffb421d2fa0)

[S7-1200与Labview以太网通讯-SIMATICS7-1200系列-找答案-西门子中国 (siemens.com.cn)](https://www.ad.siemens.com.cn/service/answer/solved_181535_1072.html)

- **OPC**

  OPC是开放的，1200的协议不开放，PLC数据读进OPC，VC或LABVIEW读写OPC的数据。PC上使用SIMATIC NET 或KEPSERVER

  [使用OPC将LabVIEW连接到任何PLC - NI](https://knowledge.ni.com/KnowledgeArticleDetails?id=kA03q000000x0MPCAY&l=en-SG)

  [1200、1500用labview做上位机-SIMATIC S7-1500系列-找答案-西门子中国 (siemens.com.cn)](https://www.ad.siemens.com.cn/service/answer/solved_249360_1077.html)

- **s7 NET**

​	[SP7 Toolkit for LabVIEW Download - NI](https://www.ni.com/en/support/downloads/tools-network/download.sp7-toolkit-for-labview.html#379042)

- **HSL communication**

  [HslCommunication - 胡工科技官网 (hsltechnology.cn)](http://www.hsltechnology.cn/Doc/HslCommunication)官网 使用教程 

  [【工控老马】labview 调用HslCommunication.dll 教程-CSDN博客](https://blog.csdn.net/ksthen/article/details/122598903)

  基于modbus

  [HslCommunication 命名空间](http://api.hslcommunication.cn/html/c136d3de-eab7-9b0f-4bdf-d891297c8018.htm)



## 注意事项

- 数据类型 [basic_type (siemens.com.cn)](https://www.ad.siemens.com.cn/productportal/Prods/S7-1200_PLC_EASY_PLUS/07-Program/02-basic/01-Data_Type/01-basic.html)
  - UDint 无符号双整数 32位
  - real 单精度浮点数 32
- 文件保存格式 
  - 

## 修改想法

1. 如果程序直接读取每一位的数据，可以直接进行所有一块传输修改，不用按照循环读取进行修改，单个修改单独进行即可



直接使用

每一秒记录一次 100s -14kb 不采样 -57kb

​	每秒0.14kb	每分钟 8.4kb	每小时 504kb 两小时 1m 一天12m 一年3g

修改表头 132s 3kb 

​	每秒 0.023kb	每分钟1.36 kb	每小时81.8kb  一天2m 一年700m 



