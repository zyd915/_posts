---
title: scala语言学习（十）、scala泛型、scala上下界
permalink: scala-ten
date: 2020-07-21 00:12:15
updated:  2020-07-21 00:12:16
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala和Java一样，类和特质、方法都可以支持泛型。我们在学习集合的时候，一般都会涉及到泛型。在scala中，使用方括号来定义类型参数。
---
## 泛型
scala和Java一样，类和特质、方法都可以支持泛型。我们在学习集合的时候，一般都会涉及到泛型。在scala中，使用方括号来定义类型参数。

```scala
scala> val list1:List[String] = List("1", "2", "3")
list1: List[String] = List(1, 2, 3)
```

### 定义一个泛型方法

#### 不考虑泛型支持
```scala
 def getMiddle(arr:Array[Int]) = arr(arr.length / 2)

  def main(args: Array[String]): Unit = {
    val arr1 = Array(1,2,3,4,5)

    println(getMiddle(arr1))
  }

```

#### 考虑泛型支持
```scala

def getMiddle[A](arr:Array[A]) = arr(arr.length / 2)

  def main(args: Array[String]): Unit = {
    val arr1 = Array(1,2,3,4,5)
    val arr2 = Array("a", "b", "c", "d", "f")

    println(getMiddle[Int](arr1))
    println(getMiddle[String](arr2))

    // 简写方式
    println(getMiddle(arr1))
    println(getMiddle(arr2))
  }
```


### 定义一个泛型类
定义一个Pair类包含2个类型不固定的泛型.
```scala
// 类名后面的方括号，就表示这个类可以使用两个类型、分别是T和S
// 这个名字可以任意取
class Pair[T, S](val first: T, val second: S)

case class People(var name:String, val age:Int)

object Pair {
  def main(args: Array[String]): Unit = {

  val p1 = new Pair[String, Int]("张三", 10)
  val p2 = new Pair[String, String]("张三", "1988-02-19")
  val p3 = new Pair[People, People](People("张三", 20), People("李四", 30))
  }
}

```

## 上下界
在指定泛型类型时，有时需要界定泛型类型的范围，而不是接收任意类型==。比如，要求某个泛型类型，必须是某个类的子类，这样在程序中就可以放心的调用父类的方法，程序才能正常的使用与运行.

scala的上下边界特性允许泛型类型是某个类的子类，或者是某个类的父类，

- `U >: T`，这是类型下界的定义，也就是U必须是类型T的父类或者是自己本身。
- `U <: T`，这是类型上界的定义，也就是U必须是类型T的子类或者是自己本身。

```scala
// 类名后面的指定泛型的范围, 上界
class Pair1[T <: Person, S <:Person](val first: T, val second: S) {
  def chat(msg:String) = println(s"${first.name}对${second.name}说: $msg")
}

class Person(var name:String, val age:Int)

object Pair1 {
  def main(args: Array[String]): Unit = {

  val p3 = new Pair1[Person,Person](new Person("张三", 20), new Person("李四", 30))
  p3.chat("你好啊！")
  }
}
```

![](https://static.studytime.xin/article/20200719180537.png)

#### 案例
```scala
//要控制Person只能和Person、Policeman聊天，但是不能和Superman聊天。此时，还需要给泛型添加一个下界。

//上下界
class Pair[T <: Person, S >: Policeman <:Person](val first: T, val second: S) {
  def chat(msg:String) = println(s"${first.name}对${second.name}说: $msg")
}

class Person(var name:String, val age:Int)
class Policeman(name:String, age:Int) extends Person(name, age)
class Superman(name:String) extends Policeman(name, -1)

object Pair {
  def main(args: Array[String]): Unit = {
	// 编译错误：第二个参数必须是Person的子类（包括本身）、Policeman的父类（包括本身）
   val p3 = new Pair[Person,Superman](new Person("张三", 20), new Superman("李四"))
   p3.chat("你好啊！")
  }
}

```


## 协变、逆变、非变

![](https://static.studytime.xin/article/20200719180858.png)


### 协变
class Pair[+T]，这种情况是协变。类型B是A的子类型，Pair[B]可以认为是Pair[A]的子类型。这种情况，参数化类型的方向和类型的方向是一致的。
```
____            　　_____________ 
|     |             |             |
|  A  |             |  List[ A ]  |
|_____|             |_____________|
   ^                       ^ 
   |                       | 
 _____               _____________ 
|     |             |             |
|  B  |             |  List[ B ]  |
|_____|             |_____________|
```

### 逆变
class Pair[-T]，这种情况是逆变。类型B是A的子类型，Pair[A]反过来可以认为是Pair[B]的子类型。这种情况，参数化类型的方向和类型的方向是相反的。
```
____            　　_____________ 
|     |             |             |
|  A  |             |  List[ B ]  |
|_____|             |_____________|
   ^                       ^ 
   |                       | 
 _____               _____________ 
|     |             |             |
|  B  |             |  List[ A ]  |
|_____|             |_____________|
```

### 非变
class Pair[T]{}，这种情况就是非变（默认），类型B是A的子类型，Pair[A]和Pair[B]没有任何从属关系，这种情况和Java是一样的。

```
class Super
class Sub extends Super

//非变
class Temp1[A](title: String)
//协变
class Temp2[+A](title: String)
//逆变
class Temp3[-A](title: String)

object Covariance_demo {
  def main(args: Array[String]): Unit = {
    val a = new Sub()
    // 没有问题，Sub是Super的子类
    val b:Super = a

    // 非变
    val t1:Temp1[Sub] = new Temp1[Sub]("测试")
    // 报错！默认不允许转换
    // val t2:Temp1[Super] = t1

    // 协变
    val t3:Temp2[Sub] = new Temp2[Sub]("测试")
    val t4:Temp2[Super] = t3
    
    // 逆变
    val t5:Temp3[Super] = new Temp3[Super]("测试")
    val t6:Temp3[Sub] = t5
  }
}

```

### 总结
>C[+T]：如果A是B的子类，那么C[A]是C[B]的子类。
C[-T]：如果A是B的子类，那么C[B]是C[A]的子类。
C[T]： 无论A和B是什么关系，C[A]和C[B]没有从属关系。

函数的参数类型是逆变的，而函数的返回类型是协变的。

### 我们在定义Scala类的时候，是不是可以随便指定泛型类型为协变或者逆变呢？
答案是否定的。通过上面的例子可以看出，如果将Function1的参数类型定义为协变，或者返回类型定义为逆变，都会违反Liskov替换原则，因此，Scala规定，协变类型只能作为方法的返回类型，而逆变类型只能作为方法的参数类型。类比函数的行为，结合Liskov替换原则，就能发现这样的规定是非常合理的。