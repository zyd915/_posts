---
title: scala语言学习（十一）、scala隐式转换和隐式参数
permalink: scala-eleven
date: 2020-07-21 00:12:28
updated: 2020-07-21 00:12:29
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala提供的隐式转换和隐式参数功能，是非常有特色的功能。是Java等编程语言所没有的功能。它可以允许你手动指定，将某种类型的对象转换成其他类型的对象或者是给一个类增加方法。通过这些功能，可以实现非常强大、特殊的功能。
---

scala提供的隐式转换和隐式参数功能，是非常有特色的功能。是Java等编程语言所没有的功能。它可以允许你手动指定，将某种类型的对象转换成其他类型的对象或者是给一个类增加方法。通过这些功能，可以实现非常强大、特殊的功能。

隐式转换其核心就是定义一个使用 `implicit` 关键字修饰的方法实现把一个原始类转换成目标类，进而可以调用目标类中的方法

### 隐式参数
所谓的隐式参数，指的是在函数或者方法中，定义一个用implicit修饰的参数，
此时Scala会尝试找到一个指定类型的用implicit修饰的参数，即隐式值，并注入参数。

所有的隐式转换和隐式参数必须定义在一个object中

### 让File类具备RichFile类中的read方法
```scala

package com.kaikeba.implic_demo

import java.io.File

import scala.io.Source

//todo:隐式转换案例一:让File类具备RichFile类中的read方法

object MyPredef{
  //定义一个隐式转换的方法，实现把File转换成RichFile
  implicit  def file2RichFile(file:File)=new RichFile(file)

}

class RichFile(val file:File){
     //读取数据文件的方法
    def read():String={
       Source.fromFile(file).mkString
    }
}

object RichFile{
  def main(args: Array[String]): Unit = {
     //1、构建一个File对象
          val file = new File("E:\\aa.txt")

     //2、手动导入隐式转换
      import MyPredef.file2RichFile

       val data: String = file.read
        println(data)
  }
}
```

### 超人变身

```scala
package com.kaikeba.implic_demo

//todo:隐式转换案例二:超人变身
class Man(val name:String)

class SuperMan(val name: String) {
  def heat=print("超人打怪兽")

}

object SuperMan{
  //隐式转换方法
  implicit def man2SuperMan(man:Man)=new SuperMan(man.name)

  def main(args: Array[String]) {
      val hero=new Man("hero")
      //Man具备了SuperMan的方法
      hero.heat
  }

}


```

### 一个类隐式转换成具有相同方法的多个类
```
package com.kaikeba.implic_demo

//todo:隐式转换案例三（一个类隐式转换成具有相同方法的多个类）

class C
class A(c:C) {
    def readBook(): Unit ={
      println("A说：好书好书...")
    }
}

class B(c:C){
  def readBook(): Unit ={
    println("B说：看不懂...")
  }
  def writeBook(): Unit ={
    println("B说：不会写...")
  }
}

object AB{

  //创建一个类转换为2个类的隐式转换
  implicit def C2A(c:C)=new A(c)
  implicit def C2B(c:C)=new B(c)
}

object B{
  def main(args: Array[String]) {
    //导包
    //1. import AB._ 会将AB类下的所有隐式转换导进来
    //2. import AB.C2A 只导入C类到A类的的隐式转换方法
    //3. import AB.C2B 只导入C类到B类的的隐式转换方法
    import AB._
    val c=new C

    //由于A类与B类中都有readBook()，只能导入其中一个，否则调用共同方法时代码报错
     //c.readBook()

    //C类可以执行B类中的writeBook()
    c.writeBook()

  }
}


```


### 员工领取薪水

```scala
package cn.itcast.implic_demo

//todo:隐式参数案例四:员工领取薪水

object Company{
  //在object中定义隐式值    注意：同一类型的隐式值只允许出现一次，否则会报错
  implicit  val xxx="zhangsan"
  implicit  val yyy=10000.00

  //implicit  val zzz="lisi"

}

class Boss {
  //定义一个用implicit修饰的参数 类型为String
  //注意参数匹配的类型   它需要的是String类型的隐式值
  def callName(implicit name:String):String={
    name+" is coming !"
  }

  //定义一个用implicit修饰的参数，类型为Double
  //注意参数匹配的类型    它需要的是Double类型的隐式值
  def getMoney(implicit money:Double):String={
    " 当月薪水："+money
  }


}

object Boss extends App{
  //使用import导入定义好的隐式值，注意：必须先加载否则会报错
  import Company.xxx
  import Company.yyy

  val boss =new Boss
  println(boss.callName+boss.getMoney)

}
```

