---
title: scala语言学习（五）、scala柯里化以及应用
permalink: scala-five
date: 2020-07-07 01:25:38
updated:  2020-07-07 01:25:39
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: 柯里化以及应用
---
## 概念
柯里化(currying, 以逻辑学家Haskell Brooks Curry的名字命名)指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数作为参数的函数。 在Scala中方法和函数有细微的差别，通常编译器会自动完成方法到函数的转换。

## Scala中柯里化的形式
Scala中柯里化方法的定义形式和普通方法类似，区别在于柯里化方法拥有多组参数列表，每组参数用圆括号括起来。

![](https://static.studytime.xin/article/20200707012325.png)

mysum方法拥有两组参数，分别是(x: Int)和(y: Int)。

mysum方法对应的柯里化函数类型是：
`Int => Int = >Int`
柯里化函数的类型声明是右结合的，即上面的类型等价于：
`Int => (Int = >Int)`
表明该函数若只接受一个Int参数，则返回一个Int => Int类型的函数，这也和柯里化的过程相吻合。

## 示例 

```scala
def getAddress(a: String): (String, String) => String = {
    (b: String,c: String) => a + "-" + b + "-" + c
}

scala> val f1 = getAddress("china")
f1: (String, String) => String = <function2>

scala> f1("beijing","tiananmen")
res5: String = china-beijing-tiananmen

//这里就可以用柯里化去定义方法
def getAddress(a: String)(b: String,c: String): String = { 
  		a + "-" + b + "-" + c 
}
//调用
scala> getAddress("china")("beijing", "tiananmen")
res0: String = china-beijing-tiananmen

scala> val func1 = getAddress("a") _
scala> func1("c", "d")
res117: String = a-c-d

//之前学习使用的下面这些操作就是使用到了柯里化
List(1,2,3,4).fold(0)(_ + _)
List(1,2,3,4).foldLeft(0)(_ + _)
List(1,2,3,4).foldRight(0)(_ + _)
```