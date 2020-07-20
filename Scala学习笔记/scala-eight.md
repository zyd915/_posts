---
title: scala语言学习（八）、异常处理机制
permalink: scala-eight
date: 2020-07-21 00:11:54
updated:  2020-07-21 00:11:56
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: Scala 的异常处理和其它语言比如 Java 类似，Scala 的方法可以通过抛出异常的方法的方式来终止相关代码的运行，而不必通过返回值。
---

## 异常处理
Scala 的异常处理和其它语言比如 Java 类似，Scala 的方法可以通过抛出异常的方法的方式来终止相关代码的运行，而不必通过返回值。

### 异常场景
```scala
package xin.studytime.App

object ExceptionDemo01 {
  def main(args: Array[String]): Unit = {
    val i = 10 / 0

    println("你好！")
  }
}

Exception in thread "main" java.lang.ArithmeticException: / by zero
	at xin.studytime.App.ExceptionDemo01$.main(ExceptionDemo01.scala:5)
	at xin.studytime.App.ExceptionDemo01.main(ExceptionDemo01.scala)
```

执行程序，可以看到scala抛出了异常，而且没有打印出来"你好"。说明程序出现错误后就终止了。那怎么解决该问题呢？

### 捕获异常
在Scala里，借用了模式匹配的思想来做异常的匹配。

scala异常格式：
```scala
  try {
      // 代码
  }
  catch {
      case ex:异常类型1 => // 代码
      case ex:异常类型2 => // 代码
  }
  finally {
      // 代码
  }
```

- try中的代码是我们编写的业务处理代码
- 在catch中表示当出现某个异常时，需要执行的代码
- 在finally中，是不管是否出现异常都会执行的代码

```scala
package xin.studytime.App

object ExceptionDemo01 {
  def main(args: Array[String]): Unit = {
    try {
      val i = 10 / 0

    } catch {
      case ex: Exception => println(ex.getMessage)
    } finally {
      println("我始终都会执行!")
    }
  }
}
```


### 抛出异常
我们也可以在一个方法中，抛出异常。语法格式和Java类似，使用throw new Exception。

```scala
def main(args: Array[String]): Unit = {
      throw new Exception("这是一个异常")
    }
  
  Exception in thread "main" java.lang.Exception: 这是一个异常
  	at ForDemo$.main(ForDemo.scala:3)
  	at ForDemo.main(ForDemo.scala)
```