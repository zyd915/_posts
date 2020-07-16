---
title: scala语言学习（七）、模式匹配和样例类
permalink: scala-seven
date: 2020-07-17 00:34:21
updated:  2020-07-17 00:34:21
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala有一个十分强大的模式匹配机制，可以应用到很多场合。而且scala还提供了样例类，对模式匹配进行了优化，可以快速进行匹配。
---

scala有一个十分强大的模式匹配机制，可以应用到很多场合。而且scala还提供了样例类，对模式匹配进行了优化，可以快速进行匹配。
- switch语句
- 类型查询
- 以及快速获取数据

<a name="8gsd3"></a>
## 模式匹配
<a name="AJGID"></a>
### 匹配字符串
```scala
package xin.studytime.App

import scala.util.Random

object CaseDemo01 extends App {
  val arr = Array("hadoop", "zookeeper", "spark", "storm")

  val name = arr(Random.nextInt(arr.length))
  println(name)

  name match {
    case "hadoop" => println("大数据分布式存储和计算框架。。。。。")
    case "zookeeper" => println("大数据分布式协调框架。。。。")
    case "spark" => println("大数据分布式内存计算框架。。。。")
    case _ => println("这是个啥。。。。")
  }
}

```
<a name="O9Zlp"></a>
### 匹配类型
```scala
package xin.studytime.App

import scala.util.Random

object CaseDemo02 extends App {
  val arr = Array("hello", 1, -2.0, CaseDemo02)

  val value = arr(Random.nextInt(arr.length))
  println(value)

  value match {
    case x: Int => println("Int=>" + x)
    case y: Double if (y > 0) => println("Double=>" + y)
    case z: String => println("String=>" + z)
    case _ => throw new Exception("not match exception")
  }
}

```


<a name="tiD9q"></a>
### 匹配数组
```scala
package xin.studytime.App

object CaseDemo03 extends App {
  val arr = Array(1, 3, 5)

  arr match {
    case Array(1, x, y) => println(x + "---" + y)
    case Array(1, _*) => println("1...")
    case Array(0) => println("0...")
    case _ => println("something else")
  }
}

```


<a name="DeUXx"></a>
### 匹配集合
```scala
package xin.studytime.App

object CaseDemo04 extends App {
  val list = List(0, 3, 6)
  list match {
    case 0 :: Nil => println("only 0")
    case 0 :: tail => println("0...")
    case x :: y :: z :: Nil => println(s"x:$x y:$y z:$z")
    case _ => println( "someting else")
  }
}

```


<a name="Aa1i0"></a>
### 匹配元组
```scala
package xin.studytime.App

object CaseDemo05 extends App {
  val tuple=(1,3,5)
  tuple match{
    case (1,x,y)    => println(s"1,$x,$y")
    case (2,x,y)    => println(s"$x,$y")
    case _          => println("others...")
  }
}

```
<a name="xdLqS"></a>
### 
<a name="KX3yM"></a>
## 样例类
样例类是一种特殊类，它可以用来快速定义一个用于保存数据的类（类似于Java POJO类），而且它会自动生成apply方法，允许我们快速地创建样例类实例对象。在并发编程和spark、flink这些框架也都会经常使用它。<br />

<a name="pXQBq"></a>
### 语法结构
```scala
case class 样例类名(成员变量名1:类型1, 成员变量名2:类型2 ...)
```
```scala
package xin.studytime.App

case class CasePerson(name: String, age: Int)

case class CaseStudent(var name: String, var age: Int)

object CaseClassDemo {
  def main(args: Array[String]): Unit = {
    // 1. 使用new创建实例
    val zhagnsan = new CasePerson("张三", 20)
    println(zhagnsan)

    // 2. 使用类名直接创建实例
    val lisi = CasePerson("李四", 21)
    println(lisi)

    // 3. 样例类默认的成员变量都是val的，除非手动指定变量为var类型
    //lisi.age = 22  // 编译错误！age默认为val类型

    val xiaohong = CaseStudent("小红", 23)
    xiaohong.age = 24
    println(xiaohong)
  }
}

```
<br />

<a name="qmmf3"></a>
## 样例对象
使用case object可以创建样例对象。样例对象是单例的，而且它**没有主构造器**。样例对象是可序列化的。格式：<br />

<a name="4NhCT"></a>
### 语法结构
```scala
case object 样例对象名
```
   
```scala
case class SendMessage(text:String)
   
   // 消息如果没有任何参数，就可以定义为样例对象
   case object startTask
   case object PauseTask
   case object StopTask
```


<a name="BWWOo"></a>
## 样例类和样例对象结合模式使用


```scala
package xin.studytime.App

import scala.util.Random

case class SubmitTask(id: String, name: String)

case class HeartBeat(time: Long)

case object CheckTimeOutTask

object CaseDemo06 extends App {
  val arr = Array(CheckTimeOutTask,
    HeartBeat(10000),
    SubmitTask("0001", "task-0001"))

  arr(Random.nextInt(arr.length)) match {
    case SubmitTask(id, name) => println(s"id=$id, name=$name")
    case HeartBeat(time) => println(s"time=$time")
    case CheckTimeOutTask => println("检查超时")
  }
}

```
<a name="E2f75"></a>
## Option类型
在Scala中Option类型用样例类来表示可能存在或也可能不存在的值。<br />Option[T] 是一个类型为 T 的可选值的容器： 如果值存在， Option[T] 就是一个 Some[T] ，如果不存在， Option[T] 就是对象 None 。
<a name="q8qnJ"></a>
### Option类型有2个子类
<a name="BVbFA"></a>
#### Some，Some包装了某个值
![](https://static.studytime.xin/article/20200716233850.png)
<a name="fic3i"></a>
#### None，None表示没有值
![](https://static.studytime.xin/article/20200716235201.png)<br />Option 类型的值通常作为 Scala 集合类型（List，Map 等）操作的返回类型。 比如 Map 的 get 方法：
```scala
val capitals = Map("France"->"Paris", "Japan"->"Tokyo","China"->"Beijing")
capitals.get("France")
res0: Option[String] = Some(Paris)

capitals.get("North Pole")
res1: Option[String] = None
```
在上面的代码中，capitals 一个是一个 Key 的类型是 String，Value 的类型是 String 的 hash map，但不一样的是他的 get() 返回的是一个叫 Option[String] 的类别。<br />Scala 使用 Option[String] 来告诉你：「我会想办法回传一个 String，但也可能没有 String 给你」。<br />capitals 里并没有 North Pole 这笔数据，get() 方法返回 None。<br />Option 有两个子类别，一个是 Some，一个是 None，当他回传 Some 的时候，代表这个函式成功地给了你一个 String，而你可以透过 get() 这个函式拿到那个 String，如果他返回的是 None，则代表没有字符串可以给你。<br />

<a name="W1TET"></a>
#### 通过模式匹配来输出匹配值
```scala
object Test {
   def main(args: Array[String]) {
      val sites = Map("studytime" -> "www.studytime.xin", "google" -> "www.google.com")
      
      println("show(sites.get( \"studytime\")) : " +  
                                          show(sites.get( "studytime")) )
      println("show(sites.get( \"baidu\")) : " +  
                                          show(sites.get( "baidu")) )
     
     //更好的方式
     val v2 = sites.getOrElse("baidu", 0)
     println(v2)
   }
   
   def show(x: Option[String]) = x match {
      case Some(s) => s
      case None => "?"
   }
}
```
<a name="NX540"></a>
#### getOrElse() 方法
你可以使用 getOrElse() 方法来获取元组中存在的元素或者使用其默认的值
```scala
object Test {
   def main(args: Array[String]) {
      val a:Option[Int] = Some(5)
      val b:Option[Int] = None 
      
      println("a.getOrElse(0): " + a.getOrElse(0) )
      println("b.getOrElse(10): " + b.getOrElse(10) )
   }
}

结果为：
$ scalac Test.scala 
$ scala Test
a.getOrElse(0): 5
b.getOrElse(10): 10
```
<br />
<a name="ruQb2"></a>

## 偏函数
被包在花括号内没有match的一组case语句是一个偏函数,可以理解为：偏函数是一个参数和一个返回值的函数。
它是PartialFunction[A, B]的一个实例，A代表输入参数类型，B代表返回结果类型。
```scala
package xin.studytime.App

object TestPartialFunction {
  val func1: PartialFunction[Int, String] = {
    case 1 => "一"
    case 2 => "二"
    case 3 => "三"
    case _ => "其他"
  }

  def main(args: Array[String]): Unit = {
    println(func1(1))

    val list = List(1, 2, 3, 4, 5, 6)
    //使用偏函数操作
    val result = list.filter {
      case x if x > 3 => true
      case _ => false
    }
    println(result)
  }
}

```