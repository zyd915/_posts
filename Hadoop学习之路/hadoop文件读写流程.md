---
title: HDFS 文件读写流程
date: 2020-11-10 16:49:57
updated:  2020-11-10 16:53:57
tags: 
    - 大数据
    - hadoop
categories: [大数据,hadoop]
toc: true
excerpt: HDFS是一个分布式文件系统，在HDFS上读写文件的过程与我们平时使用的单机文件系统非常不同。
---

## HDFS之文件写流程
HDFS是一个分布式文件系统，在HDFS上写文件的过程与我们平时使用的单机文件系统非常不同。

![image-20200509101409505](https://static.studytime.xin/article/2020/11/image20200509101409505.png)

![](https://static.studytime.xin/article/2020/11/16049799195153.jpg)

### 具体过程描述

1. Client调用DistributedFileSystem对象的create方法，创建一个文件输出流（FSDataOutputStream）对象。

2. 通过DistributedFileSystem对象与Hadoop集群的NameNode进行一次RPC远程调用，在HDFS的Namespace中创建一个文件条目，（Entry），并将操作记录在edits.log中

3. client调用DFSOutputStream.write()写数据方法，写第一个块的内容

4. DFSOutputStream通过RPC调用namenode的addBlock，向namenode申请一个空的数据块block

5. addBlock返回LocatedBlock对象，此对象中包含了第一个块要存储在哪三个datanode的信息，比如dn1、dn2、dn3

6. 客户端会根据位置信息，与datanode建立数据管道，一次在三个datanode，第一个datanode与第二个，第二个与第三个，依次。完成后，再依次返回通知客户端

7. 客户端向datanode写入数据，会先将数据写入一个chunk中，写满512字节后，会计算校验和checksum（4字节），然后再写入packet中，当packet写满后，将其写入dataqueue队列中国，沿着管道进行写入，同时也会保存一个packet，到ack queue队列中。

8. 当该packet依次写入最后一个datanode后，再发现返回且每个节点校验，到客户端

9. 客户端根据校验结果，成功，则删除ack queue的packet，失败，则将packet去除，再次放入dataqueu末尾，重新开始

10. 按照此流程，将其余block，一次通过管道，依次发送。当发送完成后，三个节点都收到完整的副本，dn节点会通知，namenode更新内存中block和datanode的关系

11. 关闭管道，如果仍然有block责继续，按照上述流程，重新获取新的blocl对应的datanode,创建管道，上传等。

12. 关闭输出流
13. 客户端调用namenode的complete，告知文件传输完成

### HDFS写流程另一种描述
客户端要向HDFS写数据，首先要跟namenode通信以确认可以写文件并获得接收文件block的datanode，然后，客户端按顺序将文件逐个block传递给相应datanode，并由接收到block的datanode负责向其他datanode复制block的副本

![](https://static.studytime.xin/article/2020/11/16049800541557.jpg)

1. 客户端与amenode进行一次rpc通信，请求上传文件，namenode检查目标文件是否已存在，父目录是否存在 
2. namenode返回是否可以上传 
3. client会先对文件进行切分，比如一个blok块128m，文件有300m就会被切分成3个块，一个128M、一个128M、一个44M请求第一个 block该传输到哪些datanode服务器上 
4. namenode返回datanode的服务器 
5. client请求一台datanode上传数据（本质上是一个RPC调用，建立pipeline），第一个datanode收到请求会继续调用第二个datanode，然后第二个调用第三个datanode，将整个pipeline建立完成，逐级返回客户端 
6. client开始往A上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位（一个packet为64kb），当然在写入的时候datanode会进行数据校验，它并不是通过一个packet进行一次校验而是以chunk为单位进行校验（512byte），第一台datanode收到一个packet就会传给第二台，第二台传给第三台；第一台每传一个packet会放入一个应答队列等待应答 
7. 当一个block传输完成之后，client再次请求namenode上传第二个block的服务器。

## HDFS之文件读流程
客户端将要读取的文件路径发送给namenode，namenode获取文件的元信息（主要是block的存放位置信息）返回给客户端，客户端根据返回的信息找到相应datanode逐个获取文件的block并在客户端本地进行数据追加合并从而获得整个文件

### HDFS读流程描述

![](https://static.studytime.xin/article/2020/11/16049812241756.jpg)

1. client端读取HDFS文件，client调用文件系统对象DistributedFileSystem的open方法

2. 返回FSDataInputStream对象（对DFSInputStream的包装）
3. 构造DFSInputStream对象时，调用namenode的getBlockLocations方法，获得file的开始若干block（如blk1, blk2, blk3, blk4）的存储datanode（以下简称dn）列表；针对每个block的dn列表，会根据网络拓扑做排序，离client近的排在前；
4. 调用DFSInputStream的read方法，先读取blk1的数据，与client最近的datanode建立连接，读取数据
5. 读取完后，关闭与dn建立的流
6. 读取下一个block，如blk2的数据
7. 这一批block读取完后，再读取下一批block的数据
8. 完成文件数据读取后，调用FSDataInputStream的close方法


### HDFS读流程另一种描述

1. 跟namenode通信查询元数据（block所在的datanode节点），找到文件块所在的datanode服务器 
2. 挑选一台datanode（就近原则，然后随机）服务器，请求建立socket流 
3. datanode开始发送数据（从磁盘里面读取数据放入流，以packet为单位来做校验） 
4. 客户端以packet为单位接收，先在本地缓存，然后写入目标文件，后面的block块就相当于是append到前面的block块最后合成最终需要的文件。
