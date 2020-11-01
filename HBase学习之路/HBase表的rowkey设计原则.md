---
title: Hbase学习之路（六）HBase表的设计原则
permalink: hbase-rowkey-design
date: 2020-06-14 19:33:22
updated: 2020-06-14 19:33:23
tags: 
    - bigdata
    - hbase
categories: [大数据,hbase]
toc: true
excerpt: HBase表的合理设计，对HBase高性能的使用是至关重要。
---

## HBase表的列簇设计
### 列簇设计的原则
在合理范围内能尽量少的减少列簇就尽量减少列簇。最优设计是：将所有相关性很强的 key-value 都放在同一个列簇下，这样既能做到查询效率 最高，也能保持尽可能少的访问不同的磁盘文件。以用户信息为例，可以将必须的基本信息存放在一个列族，而一些附加的额外信息可以放在 另一列族。

## HBase表的rowkey设计
### rowkey设计三原则
#### rowkey长度原则
rowkey是一个二进制码流，可以是任意字符串，最大长度64kb，实际应用中一般为10-100bytes，以byte[]形式保存，一般设计成定长。
* 建议尽可能短；但是也不能太短，否则rowkey前缀重复的概率增大
* 设计过长会降低memstore内存的利用率和HFile存储数据的效率。

#### rowkey散列原则
- 建议将rowkey的高位作为散列字段，这样将提高数据均衡分布在每个RegionServer，以实现负载均衡的几率。
- 如果没有散列字段，首字段直接是时间信息。所有的数据都会集中在一个RegionServer上，这样在数据检索的时候负载会集中在个别的RegionServer上，造成热点问题，会降低查询效率。	

### rowkey唯一原则
- 必须在设计上保证其唯一性，rowkey是按照字典顺序排序存储的
- 因此，设计rowkey的时候，要充分利用这个排序的特点，可以将经常读取的数据存储到一块，将最近可能会被访问的数据放到一块

## HBase表的热点

### 什么是热点
- 检索habse的记录首先要通过row key来定位数据行。
- 当大量的client访问hbase集群的一个或少数几个节点，造成少数region server的读/写请求过多、负载过大，而其他region server负载却很小，就造成了“热点”现象。

### 热点的解决方案

#### 预分区
预分区的目的让表的数据可以均衡的分散在集群中，而不是默认只有一个region分布在集群的一个节点上。

#### 加盐             
这里所说的加盐不是密码学中的加盐，而是在rowkey的前面增加随机数，具体就是给rowkey分配一个随机前缀以使得它和之前的rowkey的开头不同

#### 哈希
哈希会使同一行永远用一个前缀加盐。哈希也可以使负载分散到整个集群，但是读却是可以预测的。使用确定的哈希可以让客户端重构完整的rowkey，可以使用get操作准确获取某一个行数据。
```
rowkey=MD5(username).subString(0,10)+时间戳
```

#### 反转
- 反转固定长度或者数字格式的rowkey。这样可以使得rowkey中经常改变的部分（最没有意义的部分）放在前面。
- 这样可以有效的随机rowkey，但是牺牲了rowkey的有序性。

```
电信公司：
移动-----------> 136xxxx9301  ----->1039xxxx631
				136xxxx1234  
				136xxxx2341 
电信
联通

user表
rowkey    name    age   sex    address
		  lisi1    21     m       beijing
		  lisi2    22     m       beijing
		  lisi3    25     m       beijing
		  lisi4    30     m       beijing
		  lisi5    40     f       shanghai
		  lisi6    50     f       tianjin
	          
需求：后期想经常按照居住地和年龄进行查询？	
rowkey= address+age+随机数
        beijing21+随机数
        beijing22+随机数
        beijing25+随机数
        beijing30+随机数
   
rowkey= address+age+随机数
```