---
title: scala语言学习（三）、Scala的数组、Map、元组、集合
permalink: scala-three
date: 2020-07-07 00:47:11
updated:  2020-07-07 00:47:12
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: Scala的数组、Map、元组、集合
---


## 数组
scala中数组的概念是和Java类似，可以用数组来存放同类型的一组数据。
数组类型分为定长数组和变长数组两种。

### 定长数组
定长数组指的是数组的**长度**是**不允许改变**的，但数组的元素是可以改变的。

#### 语法
```
// 通过指定长度定义数组
val/var 变量名 = new Array[元素类型](数组长度)

// 用元素直接初始化数组
val/var 变量名 = Array(元素1, 元素2, 元素3...)
```

特殊说明：
- 在scala中，数组的泛型使用[]来指定
- 使用()来获取元素

```scala
scala>  val a = new Array[Int](10)
a: Array[Int] = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0)

scala>  a(0)
res91: Int = 0

scala> a(0) = 10

scala> a
res93: Array[Int] = Array(10, 0, 0, 0, 0, 0, 0, 0, 0, 0)

scala>  val b = Array("hadoop", "spark", "hive")
b: Array[String] = Array(hadoop, spark, hive)

scala> b(0)
res94: String = hadoop

scala> b.length
res95: Int = 3
```

### 变长数组
变长数组指的是数组的长度是可变的，可以往数组中添加、删除元素。

#### 创建变长数组，需要提前导入ArrayBuffer类
```scala
import scala.collection.mutable.ArrayBuffer
```

#### 语法
- 创建空的ArrayBuffer变长数组
```scala
val/var a = ArrayBuffer[元素类型]()
```

- 创建带有初始元素的ArrayBuffer

```scala
val/var a = ArrayBuffer(元素1，元素2，元素3....)
```

#### 演示

```scala
//导入ArrayBuffer类型
scala> import scala.collection.mutable.ArrayBuffer
import scala.collection.mutable.ArrayBuffer

//定义一个长度为0的整型变长数组
scala> val a = ArrayBuffer[Int]()
a: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()

//定义一个有初始元素的变长数组
scala> val b = ArrayBuffer("hadoop", "storm", "spark")
b: scala.collection.mutable.ArrayBuffer[String] = ArrayBuffer(hadoop, storm, spark)
```

#### 变长数组的增删改操作
- 使用`+=`添加元素
- 使用`-=`删除元素
- 使用`++=`追加一个数组到变长数组
- 示例
```scala
// 定义变长数组
scala> val a = ArrayBuffer("hadoop", "spark", "flink")
a: scala.collection.mutable.ArrayBuffer[String] = ArrayBuffer(hadoop, spark, flink)

// 追加一个元素
scala> a += "flume"
res10: a.type = ArrayBuffer(hadoop, spark, flink, flume)

// 删除一个元素
scala> a -= "hadoop"
res11: a.type = ArrayBuffer(spark, flink, flume)

// 追加一个数组
scala> a ++= Array("hive", "sqoop")
res12: a.type = ArrayBuffer(spark, flink, flume, hive, sqoop)
```

#### 遍历数组
可以使用以下两种方式来遍历数组：
- 使用==for表达式== 直接遍历数组中的元素
- 使用 ==索引== 获得数组中的元素

- 示例
```scala
scala> for(i <- a) println(i)
hadoop
hive
flume
spark

//0 to n    ——包含0，也包含n
scala> for(i <- 0 to a.length -1 ) println(a(i))
hadoop
hive
flume
spark

//0 until n ——生成一系列的数字，包含0，不包含n
scala> for(i <- 0 until a.length) println(a(i))
hadoop
hive
flume
spark
```

#### 数组常用操作
scala中的数组封装了丰富的计算操作，将来在对数据处理的时候，不需要我们自己再重新实现。
- 求和——sum方法
- 求最大值——max方法 
- 求最小值——min方法 
- 排序——sorted方法

- 示例
```scala
scala> val array = Array(1,3,4,2,5)
array: Array[Int] = Array(1, 3, 4, 2, 5)

//求和
scala> array.sum
res10: Int = 15

//求最大值
scala> array.max
res11: Int = 5

//求最小值
scala> array.min
res12: Int = 1

//升序 sorted原数组不变，生成一个新的排过序的数组
scala> array.sorted
res13: Array[Int] = Array(1, 2, 3, 4, 5)

//降序    reverse 反转
scala> array.sorted.reverse
res14: Array[Int] = Array(5, 4, 3, 2, 1)
```



## 元组
元组可以用来包含一组==不同类型==的值。例如：姓名，年龄，性别，出生年月。元组的元素是==不可变== 的。

### 定义元组
使用括号来定义元组

```scala
val/var 元组变量名称 = (元素1, 元素2, 元素3....)
```

使用箭头来定义元素（元组只有两个元素）

```scala
val/var 元组 = 元素1 -> 元素2
```

### 示例

```scala
// 可以直接使用括号来定义一个元组
scala> val a = (1, "张三", 20, "北京市") 
a: (Int, String, Int, String) = (1,张三,20,北京市)

//使用箭头来定义元素
scala> val b = 1 -> 2 
b: (Int, Int) = (1,2)
```

### 访问元组
- 使用 `_1、_2、_3....`来访问元组中的元素
- 元组的index从1开始，_1表示访问第一个元素，依次类推

- 示例
```scala
scala> val a = (1, "张三", 20, "北京市")
a: (Int, String, Int, String) = (1,张三,20,北京市)

//获取元组中的第一个元素
scala> a._1
res18: Int = 1

//获取元组中的第二个元素
scala> a._2
res19: String = 张三

//获取元组中的第三个元素
scala> a._3
res20: Int = 20

//获取元组中的第四个元素
scala> a._4
res21: String = 北京市

//不能修改元组中的值
scala> a._4 = "上海"
<console>:12: error: reassignment to val
       a._4="上海"
           ^
```

## 映射Map
Map可以称之为映射。它是由键值对组成的集合。scala当中的Map集合与java当中的Map类似，也是key，value对形式的。
在scala中，Map也分为不可变Map和可变 Map。

### 不可变Map

#### 定义语法

```scala
val/var map = Map(键->值, 键->值, 键->值...)    // 推荐这种写法，可读性更好 
val/var map = Map((键, 值), (键, 值), (键, 值), (键, 值)...)
```

#### 演示

```scala
scala> val map1 = Map("zhangsan"->30, "lisi"->40) 
map: scala.collection.immutable.Map[String,Int] = Map(zhangsan -> 30, lisi -> 40)

scala> val map2 = Map(("zhangsan", 30), ("lisi", 30)) 
map: scala.collection.immutable.Map[String,Int] = Map(zhangsan -> 30, lisi -> 30)

// 根据key获取value 
scala> map1("zhangsan") 
res10: Int = 30
```

### 可变Map
可变Map需要手动导入==import scala.collection.mutable.Map==, 定义语法与不可变Map一致。

#### 演示

```scala
//导包
scala> import scala.collection.mutable.Map
import scala.collection.mutable.Map

//定义可变的map
scala> val map3 = Map("zhangsan"->30, "lisi"->40)
map3: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 30)

//获取zhangsan这个key对应的value
scala> map3("zhangsan")
res26: Int = 30

//给zhangsan这个key重新赋值value
scala> map3("zhangsan") = 50

//显示map3
scala> map3
res28: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 50)
```

### Map基本操作
创建一个可变的map
```scala
//导包
scala> import scala.collection.mutable.Map
import scala.collection.mutable.Map

scala> val map = Map("zhangsan"->30, "lisi"->40) 
map: scala.collection.mutable.Map[String,Int] = Map(lisi -> 40, zhangsan -> 30)
```

- 按照key获取value

```scala
// 获取zhagnsan的年龄 
scala> map("zhangsan")
res10: Int = 30

// 获取wangwu的年龄，如果wangwu不存在，则返回-1 比较友好，避免遇到不存在的key而报错
scala> map.getOrElse("wangwu", -1) 
res11: Int = -1
```

- 修改key对应的value

```scala
scala> map("lisi") = 50
```

- 添加key-value键值对

```scala
scala> map += ("wangwu" ->35)
res12: map.type = Map(lisi -> 50, zhangsan -> 30, wangwu -> 35)
```

- 删除key-value键值对

```scala
scala> map -= "wangwu"
res13: map.type = Map(lisi -> 50, zhangsan -> 30)
```

- 获取所有的key和所有的value

```scala
//获取所有的key
scala> map.keys
res36: Iterable[String] = Set(lisi, zhangsan)

//获取所有的key
scala> map.keySet
res37: scala.collection.Set[String] = Set(lisi, zhangsan)

//获取所有的value
scala> map.values
res38: Iterable[Int] = HashMap(50, 30)
```

- 遍历map

```scala
//第一种遍历
scala> for(k <- map.keys) println(k + " -> " + map(k))
lisi -> 50
zhangsan -> 30

scala> for(kv <- map3) println(kv._1 + "->" + kv._2)

//第二种遍历
scala> for((k, v) <- map) println(k + " -> " + v)
lisi -> 50
zhangsan -> 30
```


## Set集合
Set是代表没有重复元素的集合。
Set具备以下性质：
- 1、元素不重复 
- 2、不保证插入顺序

scala中的set集合也分为两种，一种是不可变集合，另一种是可变集合。

### 不可变Set集合
#### 语法
```scala
//创建一个空的不可变集
val/var 变量名 = Set[类型]()

//给定元素来创建一个不可变集
val/var 变量名 = Set[类型](元素1, 元素2, 元素3...)

```

#### 演示
```scala
// 创建set集合 
scala> val a = Set(1, 1, 2, 3, 4, 5) 
a: scala.collection.immutable.Set[Int] = Set(5, 1, 2, 3, 4)

// 获取集合的大小 
scala> a.size 
res0: Int = 5

// 遍历集合
scala> for(i <- a) println(i)

//添加元素生成新的集合
scala> a + 6
res1: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 3, 4)

// 删除一个元素 
scala> a - 1 
res2: scala.collection.immutable.Set[Int] = Set(5, 2, 3, 4)

// 删除set集合中存在的元素 
scala> a -- Set(2,3) 
res3: scala.collection.immutable.Set[Int] = Set(5, 1, 4)

// 拼接两个集合 
scala> a ++ Set(6,7,8) 
res4: scala.collection.immutable.Set[Int] = Set(5, 1, 6, 2, 7, 3, 8, 4)

//求2个Set集合的交集
scala> a & Set(3,4,5,6)
res5: scala.collection.immutable.Set[Int] = Set(5, 3, 4)

//注意：这里对不可变的set集合进行添加删除等操作，对于该集合来说是没有发生任何变化，这里是生成了新的集合，新的集合相比于原来的集合来说发生了变化。
scala> a
```

### 可变Set集合
要使用可变集，必须要手动导入： ==import scala.collection.mutable.Set==

#### 演示

```scala
//导包
scala> import scala.collection.mutable.Set
import scala.collection.mutable.Set

//定义可变的set集合
scala> val set = Set(1, 2, 3, 4, 5)
set: scala.collection.mutable.Set[Int] = Set(1, 5, 2, 3, 4)

//添加单个元素
scala> set += 6
res10: set.type = Set(1, 5, 2, 6, 3, 4)

//添加多个元素
scala> set += (6, 7, 8, 9)
res11: set.type = Set(9, 1, 5, 2, 6, 3, 7, 4, 8)

//添加一个set集合中的元素
scala> set ++= Set(10, 11)
res12: set.type = Set(9, 1, 5, 2, 6, 3, 10, 7, 4, 11, 8)

//删除一个元素
scala> set -= 11
res13: set.type = Set(9, 1, 5, 2, 6, 3, 10, 7, 4, 8)

//删除多个元素
scala> set -= (9, 10)
res15: set.type = Set(1, 5, 2, 6, 3, 7, 4, 8)

//删除一个set子集
scala> set --= Set(7, 8)
res19: set.type = Set(1,5, 2, 6, 3, 4)

scala> set.remove(1)
res17: Boolean = true

scala> set
res18: scala.collection.mutable.Set[Int] = Set(5, 2, 6, 3, 4)
```

## 列表 List
List是scala中最重要的、也是最常用的数据结构。

List具备以下性质：
- 1、可以保存重复的值 
- 2、有先后顺序

在scala中，也有两种列表，一种是不可变列表、另一种是可变列表

### 不可变列表
不可变列表就是列表的元素、长度都是不可变的

#### 语法
使用 List(元素1, 元素2, 元素3, ...) 来创建一个不可变列表，语法格式

```scala
val/var 变量名 = List(元素1, 元素2, 元素3...)

//使用 Nil 创建一个不可变的空列表
val/var 变量名 = Nil

//使用 :: 方法创建一个不可变列表
val/var 变量名 = 元素1 :: 元素2 :: Nil
```

#### 演示

```scala
//创建一个不可变列表，存放以下几个元素（1,2,3,4）
scala> val list1 = List(1, 2, 3, 4)
list1: List[Int] = List(1, 2, 3, 4)

//使用Nil创建一个不可变的空列表
scala> val list2 = Nil
list2: scala.collection.immutable.Nil.type = List()

//使用 :: 方法创建列表，包含1、2、3三个元素
scala> val list3= 1::2::3::Nil
list3: List[Int] = List(1, 2, 3)
```

### 可变列表
- 可变列表就是列表的元素、长度都是可变的。
- 要使用可变列表，先要导入 ==import scala.collection.mutable.ListBuffer==

#### 语法
使用ListBuffer[元素类型]() 创建空的可变列表，语法结构

```scala
val/var 变量名 = ListBuffer[Int]()
```

- 使用ListBuffer(元素1, 元素2, 元素3...)创建可变列表，语法结构

```scala
val/var 变量名 = ListBuffer(元素1，元素2，元素3...)
```

#### 演示

```scala
//导包
scala> import scala.collection.mutable.ListBuffer
import scala.collection.mutable.ListBuffer

//定义一个空的可变列表
scala> val a = ListBuffer[Int]()
a: scala.collection.mutable.ListBuffer[Int] = ListBuffer()

//定义一个有初始元素的可变列表
scala> val b = ListBuffer(1, 2, 3, 4)
b: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2, 3, 4)
```

### 列表操作

```scala
//导包
scala> import scala.collection.mutable.ListBuffer
import scala.collection.mutable.ListBuffer

//定义一个可变的列表
scala> val list = ListBuffer(1, 2, 3, 4)
list: scala.collection.mutable.ListBuffer[Int] = ListBuffer(1, 2, 3, 4)

//获取第一个元素
scala> list(0)
res4: Int = 1
//获取第一个元素
scala> list.head
res5: Int = 1

//获取除了第一个元素外其他元素组成的列表
scala> list.tail
res6: scala.collection.mutable.ListBuffer[Int] = ListBuffer(2, 3, 4)

//添加单个元素
scala> list += 5
res7: list.type = ListBuffer(1, 2, 3, 4, 5)

//添加一个不可变的列表
scala> list ++= List(6, 7)
res8: list.type = ListBuffer(1, 2, 3, 4, 5, 6, 7)

//添加一个可变的列表
scala> list ++= ListBuffer(8, 9)
res9: list.type = ListBuffer(1, 2, 3, 4, 5, 6, 7, 8, 9)

//删除单个元素
scala> list -= 9
res10: list.type = ListBuffer(1, 2, 3, 4, 5, 6, 7, 8)

//删除一个不可变的列表存在的元素
scala> list --= List(7,8)
res11: list.type = ListBuffer(1, 2, 3, 4, 5, 6)

//删除一个可变的列表存在的元素
scala> list --= ListBuffer(5,6)
res12: list.type = ListBuffer(1, 2, 3, 4)

//toList根据可变的列表生成一个不可变列表，原列表不变
scala> list.toList
res13: List[Int] = List(1, 2, 3, 4)

//toArray根据可变的列表生成一个新的不可变数组，原列表不变
scala> list.toArray
res14: Array[Int] = Array(1, 2, 3, 4)
```
