---
title: hadoop集群配置LZO压缩以及支持Hive
permalink: hadoop-install-lzo
thumbnail: https://static.studytime.xin/article/20200628001054.jpg
date: 2020-06-28 01:10:33
updated: 2020-06-28 01:10:35
tags: 
    - 大数据
    - hadoop
    - hive
categories: [大数据,hadoop]
toc: true
excerpt: hadoop集群配置LZO压缩,在集群上运行jar包生成loz文件,以及支持Hive.
---

### 下载以及编译lzo源码包

[LZO源码包地址](https://github.com/twitter/hadoop-lzo/tree/release-0.4.20)

#### 将下载的源码包使用maven进行编译
使用 `mvn  clean package`，编译之后将生成adoop-lzo-0.4.20.jar的jar包

特殊说明：也可以不进行编译，直接使用作者编译好的lzo jar包。链接: https://pan.baidu.com/s/13IjKDEokh_dqkGoDY1bIFA  密码: fbtf

### 将编译之后的jar包上传到node01节点，且同步至其他机器
```
# 上传
cd /opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common
rz
...

# 同步至其他机器
scp hadoop-lzo-0.4.20.jar  node02:$PWD
scp hadoop-lzo-0.4.20.jar  node03:$PWD
```

### 修改core-site.xml，配置压缩方式
```
cd /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop
vim core-site.xml

<property>
        <name>io.compression.codecs</name>
        <value>
                org.apache.hadoop.io.compress.GzipCodec,
                org.apache.hadoop.io.compress.DefaultCodec,
                org.apache.hadoop.io.compress.BZip2Codec,
                org.apache.hadoop.io.compress.SnappyCodec,
                com.hadoop.compression.lzo.LzoCodec,
                com.hadoop.compression.lzo.LzopCodec
        </value>
</property>

<property>
    <name>io.compression.codec.lzo.class</name>
    <value>com.hadoop.compression.lzo.LzoCodec</value>
</property>
```

### 同步core-site.xml配置到其他机器
```
cd /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop
scp core-site.xml  node02:$PWD
scp core-site.xml  node03:$PWD
```

### 配置hadoop-env.sh(可选)
```
export LD_LIBRARY_PATH=/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common/
```

### 配置mapred-site.xml(可选)
```
<!--启用map中间文件压缩-->
<property> 
    <name>mapreduce.map.output.compress</name> 
    <value>true</value> 
</property> 
<!--启用map中间压缩类-->
<property>
   <name>mapred.map.output.compression.codec</name>
   <value>com.hadoop.compression.lzo.LzopCodec</value>
</property>
<!--启用mapreduce文件压缩-->
<property>
    <name>mapreduce.output.fileoutputformat.compress</name>
    <value>true</value>
</property> 
<!--启用mapreduce压缩类-->
<property>
   <name>mapreduce.output.fileoutputformat.compress.codec</name>
   <value>com.hadoop.compression.lzo.LzopCodec</value>
</property>
<!--配置Jar包-->
<property>
    <name>mapred.child.env</name>
    <value>LD_LIBRARY_PATH=/opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common/</value>
</property>
```

### 同步hadoop-env.sh以及mapred-site.xml配置到其他机器(可选)
```
cd /opt/module/hadoop-2.6.0-cdh5.14.2/etc/hadoop
scp hadoop-env.sh  node02:$PWD
scp hadoop-env.sh  node03:$PWD

scp mapred-site.xml  node02:$PWD
scp mapred-site.xml  node03:$PWD
```

### 重新启动hdfs集群
```
stop-dfs.sh
start-dfs.sh
```

### 配置hive支持LZO压缩
直接将hadoop-lzo-0.20.jar这个jar包拷贝到hive的lib目录下即可。

我的hive服务器安装在node03节点上，所以在node03执行以下命令，将jar包拷贝到hive的lib目录下即可。
```
cp /opt/module/hadoop-2.6.0-cdh5.14.2/share/hadoop/common/hadoop-lzo-0.4.20.jar  /opt/module/hive-1.1.0-cdh5.14.2/lib/
```