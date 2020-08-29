---
title: scala语言学习（二）、scala的基本使用
permalink: scala-base-two
date: 2020-07-05 00:17:20
updated:  2020-07-05 00:17:22
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala是运行在 JVM 上的多范式编程语言，同时支持面向对象和面向函数编程。早期scala刚出现的时候，并没有怎么引起重视，随着Spark和Kafka这样基于scala的大数据框架的兴起，scala逐步进入大数据开发者的眼帘。scala的主要优势是它的表达性。
---

## 变量声明

### 语法格式
```
val/var 变量名称:变量类型 = 初始值
```

- `val`定义的是**不可重新赋值**的变量(值不可修改)
- `var`定义的是**可重新赋值**的变量(值可以修改)
- scala中声明变量是变量名称在前，变量类型在后，跟java是正好相反
- scala的语句最后不需要添加分号

### 演示
```
#使用val声明变量,相当于java中的final修饰,不能在指向其他的数据了
val  a:Int = 10
#使用var声明变量,后期可以被修改重新赋值
var  b:Int = 20	 
b=100
#scala中的变量的类型可以显式的声明,也可以不声明,如果不显式的声明这会根据变量的值来推断出来变量的类型(scala支持类型推断)
val c = 20
```

### 惰性变量
* Scala中使用关键字lazy来定义惰性变量，实现延迟加载(懒加载)。 
* 惰性变量只能是不可变变量，并且只有在调用惰性变量时，才会去实例化这个变量。
* 语法格式

```
lazy val 变量名 = 表达式
```

## 数据类型
Scala 与 Java有着相同的数据类型，下表列出了 Scala 支持的数据类型：

| 数据类型 | 描述 |
| --- | --- |
| Byte | 8位有符号补码整数。数值区间为 -128 到 127 |
| Short | 16位有符号补码整数。数值区间为 -32768 到 32767 |
| Int | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647 |
| Long | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
| Float | 32位IEEE754单精度浮点数 |
| Double | 64位IEEE754单精度浮点数 |
| Char | 6位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF |
| String | 字符序列 |
| Boolean | true或false |
| Unit | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
| Null | null 或空引用 |	
| Nothing | Nothing类型在Scala的类层级的最低端；它是任何其他类型的子类型。 |	
| Any | Any是所有其他类的超类 |	
| AnyRef | AnyRef类是Scala里所有引用类(reference class)的基类 |	
	
### scala类型与Java的区别
1. scala中所有的类型都使用大写字母开头
2. 整形使用Int而不是Integer
3. scala中定义变量可以不写类型，让scala编译器自动推断

### scala类型层次结构
![](https://static.studytime.xin/article/20200705001207.png)


| 类型    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| Any     | **所有类型**的父类，,它有两个子类AnyRef与AnyVal              |
| AnyVal  | **所有数值类型**的父类                                       |
| AnyRef  | 所有对象类型（引用类型）的父类                               |
| Unit    | 表示空，Unit是AnyVal的子类，它只有一个的实例（），它类似于Java中的void，但scala要比Java更加面向对象 |
| Null    | Null是AnyRef的子类，也就是说它是所有引用类型的子类。它的实例是null,   可以将null赋值给任何对象类型 |
| Nothing | 所有类型的**子类**不能直接创建该类型实例，某个方法抛出异常时，返回的就是Nothing类型，因为Nothing是所有类的子类，那么它可以赋值为任何类型 |
	
## 条件表达式
条件表达式就是if表达式，if表达式可以根据给定的条件是否满足，根据条件的结果（真或假）决定执行对应的操作。scala条件表达式的语法和Java一样。
```
//定义变量x
scala> val x =1
x: Int = 1

//if表达式
scala> val y =if(x>0) 1 else -1
y: Int = 1

//支持混合类型表达式
scala> val z=if(x>1) 1 else "error"
z: Any = error

//缺失else 相当于 if(x>2) 1 else ()
scala> val m=if(x>2) 1
m: AnyVal = ()

//scala中有个Unit类，用作不返回任何结果的方法的结果类型,相当于Java中的void，Unit只有一个实例值，写成()
scala> val n=if(x>2) 1 else ()
n: AnyVal = ()

//if(xx) else if(xx) else 
scala> val k=if(x<0) -1 else if (x==0) 0 else 1
k: Int = 1
```

## 块表达式
定义变量时用 {} 包含一系列表达式，其中块的最后一个表达式的值就是块的值。

```
val x=0 
val result={
  val y=x+10
  val z=y+"-hello"  
  val m=z+"-kaikeba"
    "over"
}
//result的值就是块表达式的结果    
//后期一个方法的返回值不需要加上return,把要返回的结果放在方法的最后一行就可以了 
```

## 循环
在scala中，可以使用for和while，但一般推荐使用for表达式，因为for表达式语法更简洁

### for循环
#### 语法结构
```
for (i <- 表达式/数组/集合){
  //表达式
}
```
#### 演示
简单的for循环
```
//简单的for循环
scala> val nums= 1 to 10
nums: scala.collection.immutable.Range.Inclusive = Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> for(i <- nums) println(i)
1
2
3
4
5
6
7
8
9
10
```

#### 双重for循环
```
//双重for循环
scala>  for(i <- 1 to 3; j <- 1 to 3) println(i*10+j)
11
12
13
21
22
23
31
32
33

//双重for循环打印99乘法表
for(i <- 1 to 9; j <- 1 to i){
  print(i+"*"+j+"="+i*j+"\t")
   if(i==j){
     println()
  }    
} 

1*1=1
2*1=2   2*2=4
3*1=3   3*2=6   3*3=9
4*1=4   4*2=8   4*3=12  4*4=16
5*1=5   5*2=10  5*3=15  5*4=20  5*5=25
6*1=6   6*2=12  6*3=18  6*4=24  6*5=30  6*6=36
7*1=7   7*2=14  7*3=21  7*4=28  7*5=35  7*6=42  7*7=49
8*1=8   8*2=16  8*3=24  8*4=32  8*5=40  8*6=48  8*7=56  8*8=64
9*1=9   9*2=18  9*3=27  9*4=36  9*5=45  9*6=54  9*7=63  9*8=72  9*9=81
```

#### 守卫
在for表达式中可以添加if判断语句，这个if判断就称为守卫
```scala
//语法结构
for(i <- 表达式/数组/集合 if 表达式) {
  // 表达式
}

scala> for(i <- 1 to 10 if i >5) println(i)
6
7
8
9
10 
```

#### for推导式
在for循环体中，可以使用yield表达式构建出一个集合，我们把使用yield的for表达式称之为推导式
```scala
# for推导式：for表达式中以yield开始，该for表达式会构建出一个集合

val v = for(i <- 1 to 5) yield i * 10
```

### while循环
scala中while循环和Java中是一致的

#### 语法结构
```scala
while(返回值为布尔类型的表达式){
    //表达式
}

scala> var x = 10
x: Int = 10

scala> while(x >5){
     | println(x)
     | x -= 1
     | }
10
9
8
7
6
```

## 方法

### 语法

```scala
def methodName (参数名:参数类型, 参数名:参数类型) : [return type] = {
    // 方法体：一系列的代码
}
```

说明:
- 参数列表的参数类型不能省略
- 返回值类型可以省略，由scala编译器自动推断
- 返回值可以不写return，默认就是{}块表达式的值

```scala
scala> def add(a:Int,b:Int) = a+b
add: (a: Int, b: Int)Int

scala> add(1,2)
res8: Int = 3

scala>
```

注意:
- 如果定义递归方法，不能省略返回值类型


```scala
# 定义递归方法（求阶乘）
* 10 * 9 * 8 * 7 * 6 * ... * 1

scala> def m1(x:Int)={
   | if(x==1) 1
   | else x * m1(x-1)
   | }
<console>:14: error: recursive method m1 needs result type
     else x * m1(x-1)
              ^

scala> def m1(x:Int):Int={
   | if(x==1) 1
   | else x * m1(x-1)
   | }
m1: (x: Int)Int

scala> m1(10)
res9: Int = 3628800
```

### 方法的参数

#### 默认参数
在定义方法时可以给参数定义一个默认值。
```scala
# 1. 定义一个计算两个值相加的方法，这两个值默认为0
# 2. 调用该方法

scala> def add(x:Int = 0, y:Int = 0) = x + y
add: (x: Int, y: Int)Int

scala> add(10)
res14: Int = 10

scala> add(10,20)
res15: Int = 30
```

#### 带名参数
在调用方法时，可以指定参数的名称来进行调用。

```scala
scala> def add(x:Int = 0, y:Int = 0) = x + y
add: (x: Int, y: Int)Int

scala> add(x=1)
res16: Int = 1
```

#### 变长参数
如果方法的参数是不固定的，可以定义一个方法的参数是变长参数。

```scala
def 方法名(参数名:参数类型*):返回值类型 = {
方法体
}
//在参数类型后面加一个*号，表示参数可以是0个或者多个
```

```scala
scala> def add(num:Int*) = num.sum
add: (num: Int*)Int

scala> add(1,2,3,4,5)
res17: Int = 15
```

## 函数

scala支持函数式编程，将来编写Spark/Flink程序中，会大量使用到函数

### 语法
```scala
val 函数变量名 = (参数名:参数类型, 参数名:参数类型....) => 函数体
```

注意:
- 函数是一个对象（变量）
- 类似于方法，函数也有输入参数和返回值
- 函数定义不需要使用def定义
- 无需指定返回值类型

```scala
scala> val add = (x:Int, y:Int) => x + y
add: (Int, Int) => Int = <function2>

scala> add(1,2)
res3: Int = 3


//一个函数没有赋予一个变量，则称为匿名函数，
//后期再实际开发代码的时候，基本上都是使用匿名函数
(x:Int,y:Int)=>x+y
```

## 方法和函数的区别
- 方法是隶属于类或者对象的，在运行时，它是加载到JVM的方法区中
- 可以将函数对象赋值给一个变量，在运行时，它是加载到JVM的堆内存中
- 函数是一个对象，继承自FunctionN，函数对象有apply，curried，toString，tupled这些方法，而方法则没有

## 10.4 方法转换为函数
有时候需要将方法转换为函数，作为变量传递，就需要将方法转换为函数
使用`_`即可将方法转换为函数

```
scala> def add(x:Int,y:Int)=x+y
add: (x: Int, y: Int)Int

scala> val a = add _
a: (Int, Int) => Int = <function2>
```