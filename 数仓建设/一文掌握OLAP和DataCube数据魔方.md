---
title: 一文掌握OLAP和DataCube数据魔方应用
permalink: datawarehouse-data-cube
date: 2021-07-06 00:56:45
updated: 2021-07-06 00:56:47
tags: 
    - 数仓
categories: 数仓建设
keywords: OLAP,OLAP是什么,DataCube,DataCube数据魔方,数据魔方,OLAP和OLTP的区别,数据切片和切块,数据上卷和钻取
toc: true
thumbnail: https://static.studytime.xin/article/nRxRicH5ISCN6uLcDadR_meitu_1.jpg
excerpt: OLAP( On-Line Analytical Processing)，联机分析处理过程。个人理解为主要场景针对大批量数据，实时性无要求，基于数仓多维模型，进行分析操作的系统中。Hadoop体系中MapReduce、Hive、Spark、Flink等都可以进行为OLAP实现。
---

OLAP（Online Analytical Process）指联机分析处理，字面意思理解可以是基于数据仓库多维度模型的基础上实现的面向分析的各类操作的集合， 而且能够弹性地提供上卷（Roll-up） 、 下钻（Drill-down） 等操作， 它是呈现集成性决策信息的方法，多用于决策支持系统、商务智能或数据仓库。操作主体一般是运营、销售市场、数据分析师等团队人员而不是用户。主要场景针对大批量数据，实时性无要求，基于数仓多维模型，进行分析操作的系统中。

OLAP的概念，在实际应用中存在广义和狭义两种不同的理解方式。 广义上的理解与字面上的意思相同，泛指一切不会对数据进行更新的分析处理。但更多的情况下OLAP被理解为其狭义上的含义，即与多维分析相关，基于立方体（Cube） 计算而进行的分析。

### 那么OLAP与OLTP区别是什么呢？

OLTP( On-Line Transaction Processing ) 联机事务处理过程，通常也可以成为面向交易的处理系统。个人理解为主要场景针对用户人机交互频繁，数据量小，操作快速响应的实时处理系统中。Mysql以及Oracle等数据库软件可以理解为OLTP的工业应用软件体现。

单次OLTP处理的数据量比较小，所涉及的表非常有限，一般仅一两张表。而OLAP是为了从大量的数据中找出某种规律性的东西，经常用到count()、sum()和avg()等聚合方法，用于了解现状并为将来的计划/决策提供数据支撑，所以对多张表的数据进行连接汇总非常普遍。

为了表示跟OLTP的数据库（database）在数据量和复杂度上的不同，一般称OLAP的操作对象为数据仓库（data warehouse），简称数仓。数据库仓库中的数据，往往来源于多个数据库，以及相应的业务日志。

![](https://static.studytime.xin/article/20200607010742.png)

### OLAP有哪些显著特点

- 从用户的思考⻆度出发，仿照用户思考模式预先为构建多维的数据模型。

- 用户可以快速查询分析各个维度数据

- 能动态的在各个维度之间切换或者进行多维度综合分析，具有极大的分析灵活性。

### OLAP和数仓的关系

OLAP和数仓的关系是互补的，一般以数据仓库作为基础，经数据仓库进行加工处理生成宽表后，将数据导入到OLAP存储中供数据分析工具读取。

在规范化数据仓库中OLAP工具和数据仓库的关系大致是这样的：

![](https://static.studytime.xin/article/20200606193107.png)

这种情况下，OLAP不允许访问中心数据库。一方面中心数据库是采取规范化建模的，而OLAP只支持对维度建模数据的分析；另一方面规范化数据仓库的中心数据库本身就不允许上层开发人员访问。而在维度建模数据仓库中，OLAP/BI工具和数据仓库的关系则是这样的：

![](https://static.studytime.xin/article/20200606193144.png)

在维度建模数据仓库中，OLAP不但可以从数据仓库中直接取数进行分析，还能对架构在其上的数据集市群做同样工作。

### OLAP的架构模式和类型分别是那些？    

1. MOLAP(Multidimensional Online Analytical Processing)    

MOLAP架构会生成一个新的多维数据集，也可以说是构建了一个实际数据立方体。其架构如下图所示： 


![](https://static.studytime.xin//studytime/image/articles/ufINyn.jpg)

在该立方体中，每一格对应一个直接地址，且常用的查询已被预先计算好。因此每次的查询都是非常快速的，但是由于立方体的更新比较慢，所以是否使用这种架构得具体问题具体分析。    

2. ROLAP(Relational Online Analytical Processing)    

ROLAP架构并不会生成实际的多维数据集，而是使用星形模式以及多个关系表对数据立方体进行模拟。其架构如下图所示： 

![](https://static.studytime.xin//studytime/image/articles/FYdroJ.jpg)

显然，这种架构下的查询没有MOLAP快速。因为ROLAP中，所有的查询都是被转换为SQL语句执行的。而这些SQL语句的执行会涉及到多个表之间的JOIN操作，没有MOLAP速度快。

3. HOLAP(Hybrid Online Analytical Processing)

这种架构综合参考MOLAP和ROLAP而采用一种混合解决方案，将某些需要特别提速的查询放到MOLAP引擎，其他查询则调用ROLAP引擎。

### DataCube数据魔方（OLAP基本操作）

早期当我们要手工从一堆数据中提取信息时，我们会分析一堆数据报告。通常这些数据报告采用二维表示，是行与列组成的二维表格。但在真实世界里我们分析数据的角度很可能有多个，数据立方体可以理解为就是维度扩展后的二维表格。

下图展示了一个多维数据抽象的数据立方体：

![](https://static.studytime.xin/article/20200606191204.png)

尽管这个例子是三维的，但更多时候数据立方体是N维的。对于大多数纯OLAP使用者来讲，数据分析的对象就是这个逻辑概念上的数据立方体，其具体实现不用深究。对于这些OLAP工具的使用者来讲，基本用法是首先配置好维表、事实表，然后在每次查询的时候告诉OLAP需要展示的维度和事实字段和操作类型即可。

下面介绍数据魔方中(OLAP基本操作)最常见的五大操作：

- 切片

- 切块

- 旋转

- 上卷

- 钻取

#### 切片和切块

在数据立方体的某一维度上选定一个维成员的操作叫切片，而对两个或多个维执行选择则叫做切块。

![](https://static.studytime.xin/article/20200606192208.png)

```

# 切片

SELECT Locates.地区, Products.分类, SUM(数量)
FROM Sales, Dates, Products, Locates
WHERE Dates.季度 = 2
AND Sales.Date_key = Dates.Date_key
AND Sales.Locate_key = Locates.Locate_key
AND Sales.Product_key = Products.Product_key
GROUP BY Locates.地区, Products.分类

# 切块
SELECT Locates.地区, Products.分类, SUM(数量)
FROM Sales, Dates, Products, Locates
WHERE (Dates.季度 = 2 OR Dates.季度 = 3) AND (Locates.地区 = '江苏' OR Locates.地区 = '上海')
AND Sales.Date_key = Dates.Date_key
AND Sales.Locate_key = Locates.Locate_key
AND Sales.Product_key = Products.Product_key
GROUP BY Dates.季度, Locates.地区, Products.分类

```

#### 旋转(Pivot)

旋转就是指改变报表或页面的展示方向。对于使用者来说，就是个视图操作，而从SQL模拟语句的角度来说，就是改变SELECT后面字段的顺序而已。

![](https://static.studytime.xin/article/20200606192310.png)

#### 上卷和钻取(Rol-up and Drill-down)

上卷可以理解为"无视"某些维度；钻取则是指将某些维度进行细分。

![](https://static.studytime.xin/article/20200606192502.png)

```

# 上卷
SELECT Locates.地区, Products.分类, SUM(数量)
FROM Sales, Products, Locates
WHERE Sales.Locate_key = Locates.Locate_key
AND Sales.Product_key = Products.Product_key
GROUP BY Locates.地区, Products.分类

# 钻取
SELECT Locates.地区, Dates.季度, Products.分类, SUM(数量)
FROM Sales, Dates, Products, Locates
WHERE Sales.Date_key = Dates.Date_key
AND Sales.Locate_key = Locates.Locate_key
AND Sales.Product_key = Products.Product_key
GROUP BY Dates.季度.月份, Locates.地区, Products.分类

```

### BI（Business Intelligence）

商务智能， 指用现代数据仓库技术、 在线分析技术、 数据挖掘和数据展现技术进行数据分析以实现商业价值大致上来讲，BI就是利用各种技术来辅助于商业决策，它需要以数据仓库的数据为基础，通过OLAP系统来做分析，必要时还需要一些数据挖掘的方法来挖掘更深层次的价值。

#### 那么两者又有什么关系呢？

数据仓库建设好以后，用户可以编写SQL语句对其进行访问并对其中数据进行分析。但每次查询都要编写SQL语句的话，未免太麻烦，而且对维度建模数据进行分析的SQL代码套路比较固定。于是，便有了OLAP工具，它专用于维度建模数据的分析。而BI工具则是能够将OLAP的结果以图表的方式展现出来，它和OLAP此时就出现在了一起。  

  
