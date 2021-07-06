---
title: hive 自定义函数浅谈（UDF、UDAF、UDTF）
permalink: hive-functions
date: 2020-07-23 23:48:01
updated: 2020-07-23 23:48:01
tags: 
    - hive
categories: [大数据,hive]
toc: true
excerpt: hive的内置函数满足不了所有的业务需求，hive提供很多的模块可以自定义功能，比如：自定义函数、serde、输入输出格式等。
---

## hive为什么需要自定义函数
hive的内置函数满足不了所有的业务需求，hive提供很多的模块可以自定义功能，比如：自定义函数、serde、输入输出格式等。

## 常见自定义函数有哪些

### UDF
用户自定义函数，user defined function。一对一的输入输出。（最常用的）。
### UDTF
用户自定义表生成函数。user defined table-generate function.一对多的输入输出。lateral view explode

### UDAF
用户自定义聚合函数。user defined aggregate function。多对一的输入输出 count sum max。

## 自定义函数实现

### UDF格式
先在工程下新建一个pom.xml,加入以下maven的依赖包 请查看`code/pom.xml`

定义UDF函数要注意下面几点:

1.  继承`org.apache.hadoop.hive.ql.exec.UDF`
2.  重写`evaluate`\(\)，这个方法不是由接口定义的,因为它可接受的参数的个数,数据类型都是不确定的。Hive会检查UDF,看能否找到和函数调用相匹配的evaluate\(\)方法

### 自定义函数案例

```java
public class FirstUDF extends UDF {
    public String evaluate(String str){
        String upper = null;
        //1、检查输入参数
        if (StringUtils.isEmpty(str)){
​
        } else {
            upper = str.toUpperCase();
        }
​
        return upper;
    }
​
    //调试自定义函数
    public static void main(String[] args){
        System.out.println(new firstUDF().evaluate("studytime"));
    }
}
```

## 函数加载方式

### 命令加载，这种加载只对本session有效

#### 将编写的udf的jar包上传到服务器上，并且将jar包添加到hive的class path中
#### 进入到hive客户端,执行下面命令 
`add jar /hivedata/udf.jar`
#### 创建一个临时函数名,要跟上面hive在同一个session里面：
`create temporary function toUP as 'com.qf.hive.FirstUDF';`
​
#### 检查函数是否创建成功
`show functions;`
​
#### 测试功能
`select toUp('abcdef');`
​
#### 删除函数 
`drop temporary function if exists tolow;`


### 启动参数加载,也是在本session有效，临时函数

#### 将编写的udf的jar包上传到服务器上
#### 创建配置文件
```
vi ./hive-init
add jar /hivedata/udf.jar;
create temporary function toup as 'com.qf.hive.FirstUDF';
```

#### 启动hive的时候带上初始化文件
```
 hive -i ./hive-init
 select toup('abcdef')
```

### 配置文件加载，通过配置文件方式这种只要用hive命令行启动都会加载函数

#### 将编写的udf的jar包上传到服务器上
#### 在hive的安装目录的bin目录下创建一个配置文件，文件名：.hiverc
```
vi ./bin/.hiverc
add jar /hivedata/udf.jar;
create temporary function toup as 'com.qf.hive.FirstUDF';
```
#### 启动hive
`show functions ;`

### UDTF格式，UDTF是一对多的输入输出,实现UDTF需要完成下面步骤

#### 继承`org.apache.hadoop.hive.ql.udf.generic.GenericUDF`，
#### 重写initlizer（）、getdisplay（）、evaluate()。

#### 执行流程如下
1. UDTF首先会调用initialize方法，此方法返回UDTF的返回行的信息（返回个数，类型）。
2. 初始化完成后，会调用process方法,真正的处理过程在process函数中，在process中，每一次forward()调用产生一行；如果产生多列可以将多个列的值放在一个数组中，然后将该数组传入到forward()函数。
3. 最后close()方法调用，对需要清理的方法进行清理。

### 自定义函数案例
把"k1:v1;k2:v2;k3:v3"类似的的字符串解析成每一行多行,每一行按照key:value格式输出

#### 代码
```java
package com.qf.hive;
​
 public class ParseMapUDTF extends GenericUDTF{
     @Override
     public void close() throws HiveException {
     }
​
     @Override
     public StructObjectInspector initialize(ObjectInspector[] args)
             throws UDFArgumentException {
         if (args.length != 1) {
             throw new UDFArgumentLengthException(" 只能传入一个参数");
         }
​
         ArrayList<String> fieldNameList = new ArrayList<String>();
         ArrayList<ObjectInspector> fieldOIs = new ArrayList<ObjectInspector>();
         fieldNameList.add("map");
         fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
         fieldNameList.add("key");
         fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
​
         return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNameList,fieldOIs);
     }
​
     @Override
     public void process(Object[] args) throws HiveException {
         String input = args[0].toString();
         String[] paramString = input.split(";");
         for(int i=0; i<paramString.length; i++) {
             try {
                 String[] result = paramString[i].split(":");
                 forward(result);
             } catch (Exception e) {
                 continue;
             }
         }
     }
 }
```

#### 打包加载
对上述命令源文件打包为udf.jar,拷贝到服务器的/hivedata/目录
在Hive客户端把udf.jar加入到hive中,如下:
```text
add jar /hivedata/udf.jar;
```

#### 创建临时函数
```
create temporary function parseMap as 'com.qf.hive.ParseMapUDTF'; # 创建一个临时函数parseMap

show functions ;
```

#### 测试临时函数
```
select parseMap("name:zhang;age:30;address:shenzhen")

#map  key  
name    zhang
age 30
address shenzhen
```


### UDAF格式
用户自定义聚合函数。user defined aggregate function。多对一的输入输出 count sum max。定义一个UDAF需要如下步骤:

1. UDF自定义函数必须是org.apache.hadoop.hive.ql.exec.UDAF的子类,并且包含一个或多个个嵌套的的实现了org.apache.hadoop.hive.ql.exec.UDAFEvaluator的静态类。
2. 函数类需要继承UDAF类，内部类Evaluator实UDAFEvaluator接口。
3. Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函

这几个函数作用如下:

#### 函数说明
init实现接口UDAFEvaluator的init函数
iterate每次对一个新值进行聚集计算都会调用,计算函数要根据计算的结果更新其内部状态terminatePartial无参数，其为iterate函数轮转结束后，返回轮转数据merge接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean。terminate返回最终的聚集函数结果。

#### 自定义函数案例
计算一组整数的最大值

#### 代码*

```java
package com.qf.hive;
​
public class MaxValueUDAF extends UDAF {
    public static class MaximumIntUDAFEvaluator implements UDAFEvaluator {
        private IntWritable result;
        public void init() {
            result = null;
        }
        public boolean iterate(IntWritable value) {
            if (value == null) {
                return true;
            }
            if (result == null) {
                result = new IntWritable( value.get() );
            } else {
                result.set( Math.max( result.get(), value.get() ) );
            }
            return true;
        }
        public IntWritable terminatePartial() {
            return result;
        }
        public boolean merge(IntWritable other) {
            return iterate( other );
        }
        public IntWritable terminate() {
            return result;
        }
    }
}
```

#### 打包加载
对上述命令源文件打包为udf.jar,拷贝到服务器的/hivedata/目录
在Hive客户端把udf.jar加入到hive中,如下:
```text
add jar /hivedata/udf.jar;
```

#### 创建临时函数
```
create temporary function maxInt as 'com.qf.hive.MaxValueUDAF';
​
# 查看函数是否加入
show functions ;
```

### 测试临时函数

```m
select maxInt(mgr) from emp

7902
```
