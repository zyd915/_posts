---
title: 浅谈数仓二、OLAP和DataCube数据魔方
date: 2020-06-07 00:56:45
updated: 2020-06-07 00:56:47
tags: 
    - 数仓
categories: data
toc: true
thumbnail: https://static.studytime.xin/article/nRxRicH5ISCN6uLcDadR_meitu_1.jpg
excerpt: OLAP( On-Line Analytical Processing)，联机分析处理过程。个人理解为主要场景针对大批量数据，实时性无要求，基于数仓多维模型，进行分析操作的系统中。Hadoop体系中MapReduce、Hive、Spark、Flink等都可以进行为OLAP实现。
---

OLTP( On-Line Transaction Processing ) 联机事务处理过程，通常也可以成为面向交易的处理系统。个人理解为主要场景针对用户人机交互频繁，数据量小，操作快速响应的实时处理系统中。Mysql以及Oracle等数据库软件可以理解为OLTP的工业应用软件体现。

OLAP( On-Line Analytical Processing)，联机分析处理过程。个人理解为主要场景针对大批量数据，实时性无要求，基于数仓多维模型，进行分析操作的系统中。Hadoop体系中MapReduce、Hive、Spark、Flink等都可以进行为OLAP实现。

### OLAP和OLTP区别
![ea607040374e31a83b6fed9241f1e66b.png](evernotecid://7E7E8A12-7BA4-4081-A93E-6E3CE2ABD0B6/appyinxiangcom/18253885/ENResource/p435)

### OLAP的特点
- 从用户的思考⻆度出发，仿照用户思考模式预先为构建多维的数据模型。
- 用户可以快速查询分析各个维度数据
- 能动态的在各个维度之间切换或者进行多维度综合分析，具有极大的分析灵活性。

### OLAP和数仓的关系
OLAP和数仓的关系是互补的。
一般以数据仓库作为基础，即从数据仓库中抽取详细数据的一个子集并经过必要的聚集存储到OLAP存储中供数据分析工具读取。

在规范化数据仓库中OLAP工具和数据仓库的关系大致是这样的：

![](https://static.studytime.xin/article/20200606193107.png)

这种情况下，OLAP不允许访问中心数据库。一方面中心数据库是采取规范化建模的，而OLAP只支持对维度建模数据的分析；另一方面规范化数据仓库的中心数据库本身就不允许上层开发人员访问。而在维度建模数据仓库中，OLAP/BI工具和数据仓库的关系则是这样的：
![](https://static.studytime.xin/article/20200606193144.png)

在维度建模数据仓库中，OLAP不但可以从数据仓库中直接取数进行分析，还能对架构在其上的数据集市群做同样工作。

### DataCube数据魔方
很多年前，当我们要手工从一堆数据中提取信息时，我们会分析一堆数据报告。通常这些数据报告采用二维表示，是行与列组成的二维表格。
但在真实世界里我们分析数据的角度很可能有多个，数据立方体可以理解为就是维度扩展后的二维表格。
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

