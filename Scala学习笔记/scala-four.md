---
title: scala语言学习（四）、函数式编程
date: 2020-07-07 01:14:31
permalink: scala-four
date: 2020-07-07 01:14:31
updated:  2020-07-07 01:14:32
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: 函数式编程
---

## 函数式编程
我们将来使用Spark/Flink的大量业务代码都会使用到函数式编程。
下面的这些操作是学习的重点，先来感受下如何进行函数式编程以及它的强大

## 遍历 - foreach
### 方法描述
```scala
foreach(f: (A) ⇒ Unit): Unit
```

### 方法说明

| foreach | API           | 说明                                                         |
| ------- | ------------- | ------------------------------------------------------------ |
| 参数    | f: (A) ⇒ Unit | 接收一个函数对象作为参数<br />函数的输入参数为集合的元素<br />返回值为空 |
| 返回值  | Unit          | 空                                                           |

### 方法实操

```scala
scala> val list = List(1, 2, 3, 4)
list: List[Int] = List(1, 2, 3, 4)

//定义一个匿名函数传入到foreach方法中
scala> list.foreach((x: Int) => println(x))
1
2
3
4

//匿名函数的输入参数类型可以省略，由编译器自动推断
scala> list.foreach(x => println(x))
1
2
3
4

//当函数参数，只在函数体中出现一次，而且函数体没有嵌套调用时，可以使用下划线来简化函数定义
scala> list.foreach(println(_))
1
2
3
4

//最简写，直接给定println
scala> list.foreach(println)
1
2
3
4

//很神奇的语法，别害怕，盘它就可以了，后期通过scala语言开发spark、Flink程序非常简洁方便
```

## 映射 - map
集合的映射操作是将来在编写Spark/Flink用得最多的操作，是我们必须要掌握。

### 方法描述

```scala
def map[B](f: (A) ⇒ B): TraversableOnce[B]
```

### 方法说明

| map方法 | API                | 说明                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| 泛型    | [B]                | 指定map方法最终返回的集合泛型                                |
| 参数    | f: (A) ⇒ B         | 传入一个函数对象作为参数<br />该函数接收一个类型A（要转换的集合的元素类型）<br />返回值为类型B |
| 返回值  | TraversableOnce[B] | B类型的集合                                                  |

### 方法实操

```scala
//定义一个list集合，实现把内部每一个元素做乘以10，生成一个新的list集合
scala> val list = List(1, 2, 3, 4)
list: List[Int] = List(1, 2, 3, 4)

//定义一个匿名函数
scala> list.map((x: Int) => x * 10)
res21: List[Int] = List(10, 20, 30, 40)

//省略匿名函数参数类型
scala> list.map(x => x * 10)
res22: List[Int] = List(10, 20, 30, 40)

//最简写用下划线
scala> list.map(_ * 10)
res23: List[Int] = List(10, 20, 30, 40)
```

## 扁平化映射 - flatmap
映射扁平化也是将来用得非常多的操作，也是必须要掌握的。

### 方法描述

```scala
def flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): TraversableOnce[B]
```

### 方法说明

| flatmap方法 | API                            | 说明                                                         |
| ----------- | ------------------------------ | ------------------------------------------------------------ |
| 泛型        | [B]                            | 最终要转换的集合元素类型                                     |
| 参数        | f: (A) ⇒ GenTraversableOnce[B] | 传入一个函数对象作为参数<br/>函数的参数是集合的元素<br />函数的返回值是一个集合 |
| 返回值      | TraversableOnce[B]             | B类型的集合                                                  |

### 方法实操

```scala
//定义一个List集合,每一个元素中就是一行数据，有很多个单词
scala>  val list = List("hadoop hive spark flink", "hbase spark")
list: List[String] = List(hadoop hive spark flink, hbase spark)

//使用flatMap进行偏平化处理，获取得到所有的单词
scala> list.flatMap(x => x.split(" "))
res24: List[String] = List(hadoop, hive, spark, flink, hbase, spark)

//简写
scala> list.flatMap(_.split(" "))
res25: List[String] = List(hadoop, hive, spark, flink, hbase, spark)

// flatMap该方法其本质是先进行了map 然后又调用了flatten
scala> list.map(_.split(" ")).flatten
res26: List[String] = List(hadoop, hive, spark, flink, hbase, spark)
```

## 过滤 - filter
过滤符合一定条件的元素

### 方法描述

```scala
def filter(p: (A) ⇒ Boolean): TraversableOnce[A]
```

### 方法说明

| filter方法 | API                | 说明                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| 参数       | p: (A) ⇒ Boolean   | 传入一个函数对象作为参数<br />函数的参数是集合中的元素<br />此函数返回布尔类型，满足条件返回true, 不满足返回false |
| 返回值     | TraversableOnce[A] | 列表                                                         |

### 方法实操

```scala
//定义一个list集合
scala> val list = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
list: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

//过滤出集合中大于5的元素
scala> list.filter(x => x > 5)
res27: List[Int] = List(6, 7, 8, 9, 10)

//把集合中大于5的元素取出来乘以10生成一个新的list集合
scala> list.filter(_ > 5).map(_ * 10)
res29: List[Int] = List(60, 70, 80, 90, 100)

//通过这个案例，应该是可以感受到scala比java的强大了...
```

## 排序 - sort
在scala集合中，可以使用以下几种方式来进行排序
- sorted默认排序 
- sortBy指定字段排序 
- sortWith自定义排序

### sorted默认排序

```scala
//定义一个List集合
scala> val list = List(5, 1, 2, 4, 3)
list: List[Int] = List(5, 1, 2, 4, 3)

//默认就是升序
scala> list.sorted
res30: List[Int] = List(1, 2, 3, 4, 5)
```

### sortBy指定字段排序
根据传入的函数转换后，再进行排序

#### 方法描述
  
```scala
def sortBy[B](f: (A) ⇒ B): List[A]
```
  
### 方法说明

| sortBy方法 | API        | 说明                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| 泛型       | [B]        | 按照什么类型来进行排序                                       |
| 参数       | f: (A) ⇒ B | 传入函数对象作为参数<br />函数接收一个集合类型的元素为参数<br />返回B类型的元素进行排序 |
| 返回值     | List[A]    | 返回排序后的列表                                             |

#### 方法实操
  
```scala
//定义一个List集合
scala> val list = List("1 hadoop", "2 spark", "3 flink")
list: List[String] = List(1 hadoop, 2 spark, 3 flink)

//按照单词的首字母进行排序
scala> list.sortBy(x => x.split(" ")(1))
res33: List[String] = List(3 flink, 1 hadoop, 2 spark)
```
  
### sortWith自定义排序
自定义排序，根据一个函数来进行自定义排序

#### 方法描述

```scala
def sortWith(lt: (A, A) ⇒ Boolean): List[A]
```

#### 方法说明

| sortWith方法 | API                  | 说明                                                         |
| ------------ | -------------------- | ------------------------------------------------------------ |
| 参数         | lt: (A, A) ⇒ Boolean | 传入一个比较大小的函数对象作为参数<br />函数接收两个集合类型的元素作为参数<br />返回两个元素大小，小于返回true，大于返回false |
| 返回值       | List[A]              | 返回排序后的列表                                             |

#### 方法实操

```scala
scala> val list = List(2, 3, 1, 6, 4, 5)
a: List[Int] = List(2, 3, 1, 6, 4, 5)

//降序
scala> list.sortWith((x, y) => x > y)
res35: List[Int] = List(6, 5, 4, 3, 2, 1)

//简写
scala> list.sortWith(_ > _)

//升序
scala> list.sortWith((x, y) => x < y)
res36: List[Int] = List(1, 2, 3, 4, 5, 6)

//升序 简写
scala> list.sortWith(_ < _)
```

## 分组 - groupBy
我们如果要将数据按照分组来进行统计分析，就需要使用到分组方法
groupBy表示按照函数将列表分成不同的组

### 方法描述

```scala
def groupBy[K](f: (A) ⇒ K): Map[K, List[A]]
```

### 方法说明

| groupBy方法 | API             | 说明                                                         |
| ----------- | --------------- | ------------------------------------------------------------ |
| 泛型        | [K]             | 分组字段的类型                                               |
| 参数        | f: (A) ⇒ K      | 传入一个函数对象作为参数<br />函数接收集合元素作为参数<br />返回一个K类型的key，这个key会用来进行分组，相同的key放在一组中 |
| 返回值      | Map[K, List[A]] | 返回一个映射，K为分组字段，List为这个分组字段对应的一组数据  |

### 方法实操

```scala
scala> val a = List("张三" -> "男", "李四" -> "女", "王五" -> "男")
a: List[(String, String)] = List((张三,男), (李四,女), (王五,男))

// 按照性别分组
scala> a.groupBy((kv: (String, String)) => {kv._2})
//简写
scala> a.groupBy(_._2)
res0: scala.collection.immutable.Map[String,List[(String, String)]] = Map(男 -> List((张三,男), (王五,男)),
女 -> List((李四,女)))

// 将分组后的映射转换为性别/人数元组列表
scala> res0.map(x => x._1 -> x._2.size)
res3: scala.collection.immutable.Map[String,Int] = Map(男 -> 2, 女 -> 1)
```

## 聚合 - reduce
reduce表示将列表，传入一个函数进行聚合计算

### 方法描述

```scala
def reduce[A1 >: A](op: (A1, A1) ⇒ A1): A1
```

### 方法说明

| reduce方法 | API               | 说明                                                         |
| ---------- | ----------------- | ------------------------------------------------------------ |
| 泛型       | [A1 >: A]         | （下界）A1必须是集合元素类型的子类                           |
| 参数       | op: (A1, A1) ⇒ A1 | 传入函数对象，用来不断进行聚合操作<br />第一个A1类型参数为：当前聚合后的变量<br />第二个A1类型参数为：当前要进行聚合的元素 |
| 返回值     | A1                | 列表最终聚合为一个元素                                       |

### 方法实操

```scala
scala> val a = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
a: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> a.reduce((x, y) => x + y)
res5: Int = 55

// 第一个下划线表示第一个参数，就是历史的聚合数据结果
// 第二个下划线表示第二个参数，就是当前要聚合的数据元素
scala> a.reduce(_ + _)
res53: Int = 55

// 与reduce一样，从左往右计算
scala> a.reduceLeft(_ + _)
res0: Int = 55

// 从右往左聚合计算
scala> a.reduceRight(_ + _)
res1: Int = 55
```

## 折叠 - fold
fold与reduce很像，但是多了一个指定初始值参数

### 方法描述

```scala
def fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
```

### 方法说明

| reduce方法 | API               | 说明                                                         |
| ---------- | ----------------- | ------------------------------------------------------------ |
| 泛型       | [A1 >: A]         | （下界）A1必须是集合元素类型的子类                           |
| 参数1      | z: A1             | 初始值                                                       |
| 参数2      | op: (A1, A1) ⇒ A1 | 传入函数对象，用来不断进行折叠操作<br />第一个A1类型参数为：当前折叠后的变量<br />第二个A1类型参数为：当前要进行折叠的元素 |
| 返回值     | A1                | 列表最终折叠为一个元素                                       |

### 方法实操

```scala
//定义一个List集合
scala> val a = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
a: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

//求和
scala> a.sum
res41: Int = 55

//给定一个初始值，，折叠求和
scala> a.fold(0)(_ + _)
res42: Int = 55

scala> a.fold(10)(_ + _)
res43: Int = 65

//从左往右
scala> a.foldLeft(10)(_ + _)
res44: Int = 65

//从右往左
scala> a.foldRight(10)(_ + _)
res45: Int = 65

//fold和foldLet效果一致，表示从左往右计算
//foldRight表示从右往左计算
```

## 高阶函数
使用函数值作为参数，或者返回值为函数值的“函数”和“方法”，均称之为“高阶函数”。

### 函数值作为参数
```scala
//定义一个数组
scala> val array = Array(1, 2, 3, 4, 5)
array: Array[Int] = Array(1, 2, 3, 4, 5)

//定义一个函数
scala> val func = (x: Int) => x * 10
func: Int => Int = <function1>

//函数作为参数传递到方法中
scala> array.map(func)
res0: Array[Int] = Array(10, 20, 30, 40, 50)
```

### 匿名函数
```scala
//定义一个数组
scala> val array = Array(1, 2, 3, 4, 5)
array: Array[Int] = Array(1, 2, 3, 4, 5)

//定义一个没有名称的函数----匿名函数
scala> array.map(x => x * 10)
res1: Array[Int] = Array(10, 20, 30, 40, 50)
