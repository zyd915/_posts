---
title: scala语言学习（九）、scala提取器(Extractor)
permalink: scala-nine
date: 2020-07-21 00:11:58
updated:  2020-07-21 00:11:58
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: 提取器是从传递给它的对象中提取出构造该对象的参数。(回想样例类进行模式匹配提取参数)
---

## 提取器(Extractor)
提取器是从传递给它的对象中提取出构造该对象的参数。(回想样例类进行模式匹配提取参数)

scala 提取器是一个带有unapply方法的对象，unapply方法算是apply方法的反向操作，unapply接受一个对象，然后从对象中提取值，提取的值通常是用来构造该对象的值。

![](https://static.studytime.xin/article/20200719175442.png)

![](https://static.studytime.xin/article/20200719175455.png)

```scala
class Student {
  var name:String = _   // 姓名
  var age:Int = _       // 年龄
  
  // 实现一个辅助构造器
  def this(name:String, age:Int) = {
    this()
    
    this.name = name
    this.age = age
  }
}

object Student {
  def apply(name:String, age:Int): Student = new Student(name, age)

  // 实现一个解构器
  def unapply(arg: Student): Option[(String, Int)] = Some(arg.name, arg.age))
}

object extractor_DEMO {
  def main(args: Array[String]): Unit = {
    val zhangsan = Student("张三", 20)

    zhangsan match {
      case Student(name, age) => println(s"姓名：$name 年龄：$age")
      case _ => println("未匹配")
    }
  }
}
```

