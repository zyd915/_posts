---
title: scala语言学习（六）、面向对象编程类、对象、继承、trait特质
permalink: scala-six
date: 2020-07-13 01:06:37
updated:  2020-07-13 01:06:37
tags:
    - 大数据
    - scala
categories: scala
toc: true
excerpt: scala是支持面向对象的，也有类和对象的概念。定义一个Customer类，并添加成员变量/成员方法。添加一个main方法，并创建Customer类的对象，并给对象赋值，打印对象中的成员，调用成员方法
---

## scala面向对象编程之类
### 类的定义
scala是支持面向对象的，也有类和对象的概念。定义一个Customer类，并添加成员变量/成员方法。添加一个main方法，并创建Customer类的对象，并给对象赋值，打印对象中的成员，调用成员方法

```scala
class Customer {
  var name:String = _
  var sex:String = _
  val registerDate:Date = new Date

  def sayHi(msg:String) = {
    println(msg)
  }
}

object Main {
  def main(args: Array[String]): Unit = {
    val customer = new Customer
    //给对象的成员变量赋值
    customer.name = "张三"
    customer.sex = "男"

    println(s"姓名: ${customer.name}, 性别：${customer.sex}, 注册时间: ${customer.registerDate}")
    //对象调用方法  
    customer.sayHi("你好!")
  }
}
```

### 特说说明：
1. `var name:String = _`表示使用默认值进行初始化， 例如：String类型默认值是null，Int类型默认值是0，Boolean类型默认值是false...
2. val变量不能使用_来进行初始化，因为val是不可变的，所以必须手动指定一个默认值
3. main方法必须要放在一个scala的object（单例对象）中才能执行


## 类的构造器

### 主构造器
主构造器是指在类名的后面跟上一系列参数，例如
```scala
class 类名(var/val 参数名:类型 = 默认值, var/val 参数名:类型 = 默认值){
  // 构造代码块
}
```

### 辅助构造器
在类中使用this来定义，例如
```scala
def this(参数名:类型, 参数名:类型) {
  ...
}
```

### 演示
```scala
class Student(val name: String, val age: Int) {
    
  val address: String = "beijing" 
  // 定义一个参数的辅助构造器
  def this(name: String) {
    // 辅助构造器的第一行必须调用主构造器或其他辅助构造器或者super父类的构造器
    this(name, 20)
  }

  def this(age: Int) {
    this("某某某", age)
  }
}
```

## scala面向对象编程之对象

### scala中的object
scala中是没有Java中的静态成员的。如果将来我们需要用到static变量、static方法，就要用到scala中的单例对象object

### 定义object
定义单例对象和定义类很像，就是把class换成object

### 演示
定义一个工具类，用来格式化日期时间

```scala
object DateUtils {

  // 在object中定义的成员变量，相当于Java中定义一个静态变量
  // 定义一个SimpleDateFormat日期时间格式化对象
  val simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm")

  // 构造代码
  println("构造代码")
    
  // 相当于Java中定义一个静态方法
  def format(date:Date) = simpleDateFormat.format(date)

  // main是一个静态方法，所以必须要写在object中
  def main(args: Array[String]): Unit = {
      
    println { DateUtils.format(new Date()) };
  }
}
```

### 说明
1. 使用object 单例对象名定义一个单例对象，可以用object作为工具类或者存放常量
2. 在单例对象中定义的变量，类似于Java中的static成员变量
3. 在单例对象中定义的方法，类似于Java中的static方法
4. object单例对象的构造代码可以直接写在花括号中
5. 调用单例对象的方法，直接使用单例对象名.方法名，访问单例对象的成员变量也是使用单例对象名.变量名
6. 单例对象只能有一个无参的主构造器，不能添加其他参数


### scala中的伴生对象
在同一个scala文件，有一个class和object具有同样的名字，那么就称这个object是class的伴生对象，class是object的伴生类,伴生类和伴生对象的最大特点是，可以相互访问；

```scala
class ClassObject {
  val id = 1
  private var name = "itcast"
  def printName(): Unit ={
    //在Dog类中可以访问伴生对象Dog的私有属性
    println(ClassObject.CONSTANT + name )
  }


}

object ClassObject{
  //伴生对象中的私有属性
  private val CONSTANT = "汪汪汪 : "
  def main(args: Array[String]) {
    val p = new ClassObject
    //访问私有的字段name
    p.name = "123"
    p.printName()
  }
}
```

#### 说明
>(1). 伴生类和伴生对象的名字必须是一样的
(2). 伴生类和伴生对象需要在一个scala源文件中
(3). 伴生类和伴生对象可以互相访问private的属性


### scala中object的apply方法
我们之前使用过这种方式来创建一个Array对象。
```scala
// 创建一个Array对象
val a = Array(1,2,3,4)
```

这种写法非常简便，不需要再写一个new，然后敲一个空格，再写类名。如何直接使用类名来创建对象呢？查看scala源代码发现：
![](https://static.studytime.xin/article/20200713004435.png)

答案就是： 实现伴生对象的apply方法，伴生对象的apply方法用来快速地创建一个伴生类的对象。

```scala
class Person(var name:String, var age:Int) {

  override def toString = s"Person($name, $age)"
}

object Person {
  // 实现apply方法
  // 返回的是伴生类的对象
  def apply(name:String, age:Int): Person = new Person(name, age)

  // apply方法支持重载
  def apply(name:String):Person = new Person(name, 20)

  def apply(age:Int):Person = new Person("某某某", age)

  def apply():Person = new Person("某某某", 20)
}

object Main2 {
  def main(args: Array[String]): Unit = {
    val p1 = Person("张三", 20)
    val p2 = Person("李四")
    val p3 = Person(100)
    val p4 = Person()

    println(p1)
    println(p2)
    println(p3)
    println(p4)
  }
}
```

#### 说明
1. 当遇到类名(参数1, 参数2...)会自动调用apply方法，在apply方法中来创建对象
2. 定义apply时，如果参数列表是空，也不能省略括号()，否则引用的是伴生对象


### scala中object的main方法
scala和Java一样，如果要运行一个程序，必须有一个main方法。而在Java中main方法是静态的，而在scala中没有静态方法。在scala中，这个main方法必须放在一个object。

```scala
object Main1{
  def main(args:Array[String]) = {
    println("hello, scala")
  }
}
```

也可以继承自App Trait（特质），然后将需要编写在main方法中的代码，写在object的构造方法体内。其本质是调用了Trait这个特质中的main方法。
```scala
object Main2 extends App {
  println("hello, scala")
}
```

## scala面向对象编程之继承

### 继承extends
scala和Java一样，使用`extends`关键字来实现继承。可以在子类中定义父类中没有的字段和方法，或者重写父类的方法。

#### 示例1：实现简单继承

```
class Person1 {
  var name = "super"

  def getName = this.name
}

class Student1 extends Person1

object Main1 {
  def main(args: Array[String]): Unit = {
    val p1 = new Person1()
    val p2 = new Student1()

    p2.name = "张三"

    println(p2.getName)
  }
}
```

#### 示例2：单例对象实现继承

```scala
class Person2 {
  var name = "super"

  def getName = this.name
}

object Student2 extends Person2

object Main2 {
  def main(args: Array[String]): Unit = {
    println(Student2.getName)
  }
}
```

### override和super

* 如果子类要覆盖父类中的一个非抽象方法，必须要使用override关键字
* 可以使用override关键字来重写一个val字段
* 可以使用super关键字来访问父类的成员

#### 示例1：class继承class

```scala
class Person3 {
  val name = "super"

  def getName = name
}

class Student3 extends Person3 {
  // 重写val字段
  override val name: String = "child"

  // 重写getName方法
  override def getName: String = "hello, " + super.getName
}

object Main3 {
  def main(args: Array[String]): Unit = {
    println(new Student3().getName)
  }
}
```

### isInstanceOf和asInstanceOf
我们经常要在代码中进行类型的判断和类型的转换。在Java中，我们可以使用instanceof关键字、以及(类型)object来实现。
scala中对象提供`isInstanceOf`和`asInstanceOf`方法。
- isInstanceOf判断对象是否为指定类的对象
- asInstanceOf将对象转换为指定类型


|                        | Java             | Scala               |
| ---------------------- | ---------------- | ------------------- |
| 判断对象是否是C类型    | obj instanceof C | obj.isInstanceof[C] |
| 将对象强转成C类型      | (C ) obj         | obj.asInstanceof[C] |
| 获取类型为T的class对象 | C.class          | classOf[C]          |

```scala
class Person4
class Student4 extends Person4

object Main4 {
  def main(args: Array[String]): Unit = {
    val s1:Person4 = new Student4

    // 判断s1是否为Student4类型
    if(s1.isInstanceOf[Student4]) {
      // 将s1转换为Student3类型
      val s2 =  s1.asInstanceOf[Student4]
      println(s2)
    }

  }
}
```

### getClass和classOf
isInstanceOf 只能判断出对象是否为指定类以及其子类的对象，而不能精确的判断出，对象就是指定类的对象。如果要求精确地判断出对象就是指定类的对象，那么就只能使用getClass和 classOf 。

* 对象.getClass可以精确获取对象的类型
* classOf[x]可以精确获取类型
* 使用==操作符就可以直接比较

```scala
class Person5
class Student5 extends Person5

object Student5{
  def main(args: Array[String]) {
    val p:Person5=new Student5
    //判断p是否为Person5类的实例
    println(p.isInstanceOf[Person5])//true

    //判断p的类型是否为Person5类
    println(p.getClass == classOf[Person5])//false

    //判断p的类型是否为Student5类
    println(p.getClass == classOf[Student5])//true
  }
}
```

### 访问修饰符
Java中的访问控制，同样适用于scala，可以在成员前面添加private/protected关键字来控制成员的可见性。但在scala中，没有public关键字，任何没有被标为private或protected的成员都是公共的==。

#### private[this]修饰符
被修饰的成员只能在当前类中被访问。或者可以理解为：`只能通过this.来访问`（在当前类中访问成员会自动添加this。
```scala
  class Person6 {
    // 只有在当前对象中能够访问
    private[this] var name = "super"

    def getName = this.name	// 正确！

    def sayHelloTo(p:Person6) = {
      println("hello" + p.name)     // 报错!无法访问
    }
  }

  object Person6 {
    def showName(p:Person6) = println(p.name)  // 报错!无法访问
  }
```

#### protected[this]修饰符
被修饰的成员只能在当前类和当前子类中被访问==。也可以理解为：当前类通过this.访问或者子类this.访问

```scala
  class Person7 {
    // 只有在当前类以及继承该类的当前对象中能够访问
    protected[this] var name = "super"

    def getName = {
      // 正确！
      this.name
    }

    def sayHelloTo1(p:Person7) = {
      // 编译错误！无法访问
      println(p.name)
    }
  }

  object Person7 {
    def sayHelloTo3(p:Person7) = {
      // 编译错误！无法访问
      println(p.name)
    }
  }

  class Student7 extends Person7 {
    def showName = {
      // 正确！
      println(name)
    }

    def sayHelloTo2(p:Person7) = {
      // 编译错误！无法访问
      println(p.name)
    }
  }
```


### 调用父类的constructor
实例化子类对象，必须要调用父类的构造器，在scala中，只能在子类的`主构造器`中调用父类的构造器
```scala
class Person8(var name:String){
    println("name:"+name)
}

// 直接在父类的类名后面调用父类构造器
class Student8(name:String, var clazz:String) extends Person8(name)

object Main8 {
  def main(args: Array[String]): Unit = {
    val s1 = new Student8("张三", "三年二班")
    println(s"${s1.name} - ${s1.clazz}")
  }
}
```

### 抽象类

* 如果类的某个成员在当前类中的定义是不包含完整的，它就是一个抽象类**
* 不完整定义有两种情况：
  * 1.方法没有方法体
  * 2.变量没有初始化
* 没有方法体的方法称为**抽象方法**，没有初始化的变量称为**抽象字段**。定义抽象类和Java一样，在类前面加上**abstract**关键字就可以了

```scala
abstract class Person9(val name:String) {
  //抽象方法
  def sayHello:String
  def sayBye:String
  //抽象字段  
  val address:String  
}
class Student9(name:String) extends Person9(name){
  //重写抽象方法
  def sayHello: String = "Hello,"+name
  def sayBye: String ="Bye,"+name
  //重写抽象字段
  override val address:String ="beijing "
}
object Main9{
  def main(args: Array[String]) {
    val s = new Student9("tom")
    println(s.sayHello)
    println(s.sayBye)
    println(s.address)
  }
}
```

### 匿名内部类
匿名内部类是没有名称的子类，直接用来创建实例对象。Spark的源代码中有大量使用到匿名内部类。
```scala
abstract class Person10 {
  //抽象方法  
  def sayHello:Unit
}

object Main10 {
  def main(args: Array[String]): Unit = {
    // 直接用new来创建一个匿名内部类对象
    val p1 = new Person10 {
      override def sayHello: Unit = println("我是一个匿名内部类")
    }
    p1.sayHello
  }
}
```



### 21. scala面向对象编程之trait特质

* 特质是scala中代码复用的基础单元
* 它可以将方法和字段定义封装起来，然后添加到类中
* 与类继承不一样的是，类继承要求每个类都只能继承`一个`超类，而一个类可以添加`任意数量`的特质。
* 特质的定义和抽象类的定义很像，但它是使用`trait`关键字



#### 21.1 作为接口使用
* 使用`extends`来继承trait（scala不论是类还是特质，都是使用extends关键字）
* 如果要继承多个trait，则使用`with`关键字

##### 继承单个trait
```scala
  trait Logger1 {
    // 抽象方法
    def log(msg:String)
  }
  
  class ConsoleLogger1 extends Logger1 {
    override def log(msg: String): Unit = println(msg)
  }
  
  object LoggerTrait1 {
    def main(args: Array[String]): Unit = {
      val logger = new ConsoleLogger1
      logger.log("控制台日志: 这是一条Log")
    }
  }
```


##### 示例二：继承多个trait

```
  trait Logger2 {
    // 抽象方法
    def log(msg:String)
  }
  
  trait MessageSender {
    def send(msg:String)
  }
  
  class ConsoleLogger2 extends Logger2 with MessageSender {
    
    override def log(msg: String): Unit = println(msg)
  
    override def send(msg: String): Unit = println(s"发送消息:${msg}")
  }
  
  object LoggerTrait2 {
    def main(args: Array[String]): Unit = {
      val logger = new ConsoleLogger2
      logger.log("控制台日志: 这是一条Log")
      logger.send("你好!")
    }
  }
```

#### 21.2 定义具体的方法
和类一样，trait中还可以定义具体的方法。
```scala
  trait LoggerDetail {
    // 在trait中定义具体方法
    def log(msg:String) = println(msg)
  }
  
  class PersonService extends LoggerDetail {
    def add() = log("添加用户")
  }
  
  object MethodInTrait {
    def main(args: Array[String]): Unit = {
      val personService = new PersonService
      personService.add()
    }
  }
```

#### 21.3 定义具体方法和抽象方法
在trait中，可以混合使用具体方法和抽象方法,使用具体方法依赖于抽象方法，而抽象方法可以放到继承trait的子类中实现，这种设计方式也称为**模板模式**
```scala
  trait Logger3 {
    // 抽象方法
    def log(msg:String)
    // 具体方法（该方法依赖于抽象方法log
    def info(msg:String) = log("INFO:" + msg)
    def warn(msg:String) = log("WARN:" + msg)
    def error(msg:String) = log("ERROR:" + msg)
  }
  
  class ConsoleLogger3 extends Logger3 {
    override def log(msg: String): Unit = println(msg)
  }
  
  object LoggerTrait3 {
    def main(args: Array[String]): Unit = {
      val logger3 = new ConsoleLogger3
  
      logger3.info("这是一条普通信息")
      logger3.warn("这是一条警告信息")
      logger3.error("这是一条错误信息")
    }
  }
```

### 定义具体字段和抽象字段
在trait中可以定义具体字段和抽象字段,继承trait的子类自动拥有trait中定义的字段,字段直接被添加到子类中.
```scala
  trait LoggerEx {
    // 具体字段
    val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm")
    val INFO = "信息:" + sdf.format(new Date)
    // 抽象字段
    val TYPE:String
  
    // 抽象方法
    def log(msg:String)
  }
  
  class ConsoleLoggerEx extends LoggerEx {
    // 实现抽象字段
    override val TYPE: String = "控制台"
    // 实现抽象方法
    override def log(msg:String): Unit = print(s"$TYPE$INFO $msg")
  }
  
  object FieldInTrait {
    def main(args: Array[String]): Unit = {
      val logger = new ConsoleLoggerEx
  
      logger.log("这是一条消息")
    }
  }
  
```

### 实例对象混入trait
trait还可以混入到`实例对象`中，给对象实例添加额外的行为, 只有混入了trait的对象才具有trait中的方法，其他的类对象不具有trait中的行为,使用with将trait混入到实例对象中.
```scala
  trait LoggerMix {
    def log(msg:String) = println(msg)
  }
  
  class UserService
  
  object FixedInClass {
    def main(args: Array[String]): Unit = {
      // 使用with关键字直接将特质混入到对象中
      val userService = new UserService with LoggerMix
  
      userService.log("你好")
    }
  }
```