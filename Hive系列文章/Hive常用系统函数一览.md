---
title: Hive系统函数一览 (建议收藏)
permalink: hive-knowledge-function 
date: 2021-07-30 15:47:13 
updated: 2021-07-30 15:47:43 
tags:
  - 大数据
  - hive 
categories: [大数据,hive]
toc: true 
thumbnail: https://static.studytime.xin//studytime/image/articles/sIZ2hq.jpg
keywords: hive常用函数, hive with as, Hive union, hive日期函数, hive拼接函数, hive系统函数
excerpt: Hive 提供了较完整的 SQL 功能，HQL 与 SQL 基本上一致，旨在让会 SQL 而不懂 MapReduce 编程的用户可以调取 Hadoop 中的数据，进行数据处理和分析。这里记录了个人日常数据分析过程中 Hive SQL 需要的查询函数，方便手头随时查询，定期更新补充。
---

Hive 提供了较完整的 SQL 功能，HQL 与 SQL 基本上一致，旨在让会 SQL 而不懂 MapReduce 编程的用户可以调取 Hadoop 中的数据，进行数据处理和分析。
这里记录了个人日常数据分析过程中 Hive SQL 需要的查询函数，方便手头随时查询，定期更新补充。

[特殊说明：本文档整理内容为作者常用部分，不代表hive只有这些，感兴趣也可以查看Hive函数官方文档https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF ](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF )

## 在 Beeline 或 CLI 中，可以使用以下命令显示Hive最新文档信息
```hql
-- 查看Hive系统函数 
show functions;
-- 查看Hive系统函数的用法
desc function upper;
-- 查看Hive系统函数的详细用法
desc function extended upper;
```


## Hive关系运算符
| 运算符 | 类型 | 功能说明 |
| --- | --- | --- |
| A = B | 所有类型 | 如果A与B相等,返回TRUE,否则返回FALSE |
| A == B | 无 | 效的语法，SQL使用”=”，不使用”==” |
| A <> B | 所有类型 | 如果A不等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL” |
| A < B | 所有类型 | 如果A小于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL” |
| A <= B | 所有类型 | 如果A小于等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL” |
| A > B | 所有类型 | 如果A大于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL” |
| A >= B | 所有类型 | 如果A大于等于B返回TRUE,否则返回FALSE。如果A或B值为”NULL”，结果返回”NULL” |
| A IS NULL | 所有类型 | 如果A值为”NULL”，返回TRUE,否则返回FALSE |
| A IS NOT NULL  | 所有类型 | 如果A值不为”NULL”，返回TRUE,否则返回FALSE |
| A [NOT] LIKE B | 所有类型 | 如果A或B值为”NULL”，结果返回”NULL”。字符串A与B通过sql进行匹配，如果相符返回TRUE，不符返回FALSE。B字符串中 的”_”代表任一字符，”%”则代表多个任意字符. 可以通过使用 NOT 关键字来反转。 |
| A [NOT] BETWEEN B AND C | 所有类型 | 如果A或B或C值为”NULL”，结果返回”NULL”。如果 A 大于或等于 B 且 A 小于或等于 C，则为 TRUE，否则为 FALSE。可以通过使用 NOT 关键字来反转。 （从 0.9.0 版开始。） |

```hql
-- 1、等值比较 = 
hive> select 1 from tableName where 1=1;
1

-- 2、不等值比较: <>
hive> select 1 from tableName where 1 <> 2;
1

-- 3、小于比较: <
hive> select 1 from tableName where 1 < 2;
1

-- 4、小于等于比较: <=
hive> select 1 from tableName where 1 < = 1;
1

-- 5、大于比较: >
hive> select 1 from tableName where 2 > 1;
1

--  6、大于等于比较: >=
hive> select 1 from tableName where 1 >= 1;
1

-- 7、空值判断: IS NULL
hive> select 1 from tableName where null is null;
1

-- 8、非空判断: IS NOT NULL
hive> select 1 from tableName where 1 is not null;
1

-- 9、LIKE比较: LIKE
hive> select 1 from tableName where 'football' like 'foot%';
1

hive> select 1 from tableName where 'football' like 'foot____';
1
```


## Hive算术运算符

| 运算符 | 类型 | 功能说明 |
| --- | --- | --- |
| A + B  | 所有类型 | 返回A与B相加的结果,结果的数值类型等于A的类型和B的类型的最小父类型;int+int一般结果为int类型，而int+double一般结果为double类型 |
| A – B   | 所有类型 | 返回A与B相减的结果。结果的数值类型等于A的类型和B的类型的最小父类型 |
| A * B  | 所有类型 | 返回A与B相乘的结果。结果的数值类型等于A的类型和B的类型的最小父类型 |
| A / B  | 所有类型 | 返回A除以B的结果。结果的数值类型为double |
| A % B  | 所有类型 | 返回A除以B的余数 |
| A & B | 所有类型 | 回A和B按位进行与操作的结果 |
| A|B | 所有类型 | 返回A和B按位进行或操作的结果  |
| A ^ B	  | 所有类型 | 返回A和B按位进行异或操作的结果 |
| ~A	  | 所有类型 | 返回A按位取反操作的结果，结果的数值类型等于A的类型 |

```hql
-- 1、Hive加法操作: +
hive> select 1 + 9 from tableName;
10

-- 2、Hive减法操作: -
hive> select 10 – 5 from tableName;
5

-- 3、Hive乘法操作: *
hive> select 40 * 5 from tableName;
200

-- 4、Hive除法操作: /
hive> select 40 / 5 from tableName;
8.0 

注意：hive中最高精度的数据类型是double,只精确到小数点后16位，在做除法运算的时候要特别注意

hive> select ceil(28.0/6.999999999999999999999) from tableName limit 1;  
4
hive> select ceil(28.0/6.99999999999999) from tableName limit 1;           
5

-- 5、Hive取余操作 : %
hive> select 41 % 5 from tableName;
1
hive> select 8.4 % 4 from tableName;
0.40000000000000036

注意：精度在hive中是个很大的问题，类似这样的操作最好通过round指定精度
hive> select round(8.4 % 4 , 2) from tableName;
0.4

-- 6、Hive位与操作: &
hive> select 4 & 8 from tableName;
0
hive> select 6 & 4 from tableName;
4

-- 7、Hive位或操作: |
hive> select 4 | 8 from tableName;
12
hive> select 6 | 8 from tableName;
14 

-- 8、Hive位异或操作: ^
hive> select 4 ^ 8 from tableName;
12
hive> select 6 ^ 4 from tableName;
2

-- 9．Hive位取反操作: ~
hive> select ~6 from tableName;
-7
hive> select ~4 from tableName;
-5
```


## Hive算术运算符

| 运算符 | 类型 | 功能说明 |
| --- | --- | --- |
| A AND B  | boolean | 如果A和B均为TRUE，则为TRUE；否则为FALSE。如果A为NULL或B为NULL，则为NULL |
| A OR B  | boolean | 如果A为TRUE，或者B为TRUE，或者A和B均为TRUE，则为TRUE；否则为FALSE |
| NOT A  | boolean | 如果A为FALSE，或者A为NULL，则为TRUE；否则为FALSE |
| A && B | boolean | 等同 A AND B |
| A | B  | boolean | 等同 A OR B |
| !A  | boolean | 等同 NOT A |
| A IN (val1, val2, ...)  | boolean | 如果 A 在列表中，则为 TRUE。从 Hive 0.13 开始，IN 语句支持子查询 |
| A NOT IN (val1, val2, ...)  | boolean | 如果 A 不在列表中，则为 TRUE。从 Hive 0.13 开始， NOT IN 语句支持子查询。 |
| A [NOT] EXISTS (subquery)  | boolean | 如果 A 存在子查询结果中，则为 TRUE。从 Hive 0.13 开始支持 |

```hql
-- 1、逻辑与操作: AND
hive> select 1 from tableName where 1=1 and 2=2;
1

-- 2、逻辑或操作: OR
hive> select 1 from tableName where 1=2 or 2=2;
1

-- 3、逻辑非操作: NOT
hive> select 1 from tableName where not 1=2;
1

-- 4、逻辑非操作: NOT
hive> select 1 from tableName where not 1=2;
1

-- 5、in查询操作: in
hive> select 1 from tableName where not 1 in (1, 2, 3);
1

-- 6、not in查询操作: not in
hive> select 1 from tableName where not 1 not in ( 2, 3);
1

-- 7、EXISTS 查询操作: [not] EXISTS
hive> select 1 from tableName where  1 EXISTS (select 1, 2, 3);
1
```

## Hive数值函数

| 运算符 | 类型 | 功能说明 |
| --- | --- | --- |
| round(double a) | BIGINT | Hive四舍五入取整 |
| round(double a, int d) | DOUBLE | Hive指定保留小数点四舍五入 |
| floor(double a) | BIGINT | 对给定数据进行向下取最接近的整数 |
| ceil(double a), ceiling(double a) | BIGINT | 参数向上取最接近的整数 |
| rand(), rand(int seed) | double | 返回一个0到1范围内的随机数。如果指定种子seed，则会等到一个稳定的随机数序列 |
| abs(double a) | double | 取绝对值 |
| positive(int a) positive(double a) | double | 取绝对值 |
| negative(int a) negative(double a) | double | 返回A的相反数 |

```hql
-- 1、取整函数: round
hive> select round(3.1415926) from tableName;
3
hive> select round(3.5) from tableName;
4

-- 2、指定精度取整函数: round
hive> select round(3.1415926,4) from tableName;
3.1416

-- 3、向下取整函数: floor
hive> select floor(3.1415926) from tableName;
3
hive> select floor(25) from tableName;
25

-- 4、向上取整函数: ceil
hive> select ceil(3.1415926) from tableName;
4
hive> select ceil(46) from tableName;
46

-- 5、向上取整函数: ceiling
hive> select ceiling(3.1415926) from tableName;
4
hive> select ceiling(46) from tableName;
46

-- 6、取随机数函数: rand
hive> select rand() from tableName;
0.5577432776034763
hive> select rand() from tableName;
0.6638336467363424
hive> select rand(100) from tableName;
0.7220096548596434
hive> select rand(100) from tableName;
0.7220096548596434

-- 7、绝对值函数: abs
hive> select abs(-3.9) from tableName;
3.9
hive> select abs(10.9) from tableName;
10.9

-- 8、取反函数：negative
hive> select negative(-5) from tableName;
5
hive> select negative(8) from tableName;
-8
```

## Hive日期函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
| from_unixtime(bigint unixtime[, string format]) | string |  转化UNIX时间戳（从1970-01-01 00:00:00 UTC到指定时间的秒数）到当前时区的时间 |
| unix_timestamp() | bigint | 获得当前时区的UNIX时间戳 |
| unix_timestamp(string date) | bigint | 转换格式为"yyyy-MM-dd HH:mm:ss"的日期到UNIX时间戳 |
| unix_timestamp(string date, string pattern) | bigint | 转换指定pattern格式的日期到UNIX时间戳 |
| to_date(string timestamp) | string | 返回时间中的年月日 |
| year(string date) | int | 返回指定时间的年份 |
| month(string date) | int | 返回指定时间的月份 |
| day(string date) | int | 返回指定时间的天 |
| hour(string date) | int | 返回指定时间的小时 |
| minute(string date) | int | 返回指定时间的分钟 |
| second(string date) | int | 返回指定时间的秒 |
| weekofyear(string date) | int | 返回指定日期所在一年中的星期号，范围为0到53 |
| datediff(string enddate, string startdate) | int | 两个时间参数的日期之差 |
| date_add(string startdate, int days) | int | 返回开始日期startdate增加days天后的日期。 |
| date_sub(string startdate, int days) | int | 返回开始日期startdate减少days天后的日期 |
| last_day(STRING date)	 | int | 指定日期字符所在月份的最后一天 |
| TRUNC（date[,fmt]）		 | int | 截断返回指定格式单位的日期，支持的格式有MONTH/MON/MM, YEAR/YYYY/YY.|
| date_format(time, string pattern)		 | int | 对时间日期进行格式化 |
| current_date()		 | string | 当前日期，yyyy-MM-dd |
| current_time()		 | string | 当前时间，HH:mm:ss|
| current_timestamp()		 | string | 返回当前UTC时间(GMT+0)的时间戳，小于北京时间8小时，就是日期时间yyyy-MM-dd HH:mm:ss返回当前UTC时间(GMT+0)的时间戳，小于北京时间8小时，就是日期时间yyyy-MM-dd HH:mm:ss|
| MONTHS_BETWEEN (x, y)	| string | 用于计算x和y之间有几个月。如果x在日历中比y早，那么MONTHS_BETWEEN()就返回一个负数 |


```hql
-- 1、Hive UNIX时间戳转日期函数: from_unixtime 
hive> select from_unixtime(1323308943,'yyyyMMdd') from tableName;
20111208

-- 2、Hive获取当前UNIX时间戳函数: unix_timestamp 
hive> select unix_timestamp() from tableName;
1323309615

-- 3、Hive日期转UNIX时间戳函数: unix_timestamp 
hive> select unix_timestamp('2011-12-07 13:01:03') from tableName;
1323234063

-- 4、Hive指定格式日期转UNIX时间戳函数: unix_timestamp 
hive> select unix_timestamp('20111207 13:01:03','yyyyMMdd HH:mm:ss') from tableName;
1323234063 

-- 5、Hive日期时间转日期函数: to_date
 hive> select to_date('2011-12-08 10:03:01') from tableName;
2011-12-08

-- 6、Hive日期转年函数: year
hive> select year('2011-12-08 10:03:01') from tableName;
2011
hive> select year('2012-12-08') from tableName;
2012

-- 7、Hive日期转月函数: month 
hive> select month('2011-12-08 10:03:01') from tableName;
12
hive> select month('2011-08-08') from tableName;
8 

-- 8、Hive日期转天函数: day
hive> select day('2011-12-08 10:03:01') from tableName;
8
hive> select day('2011-12-24') from tableName;
24 

-- 9、Hive日期转小时函数: hour 
hive> select hour('2011-12-08 10:03:01') from tableName;
10 

-- 10、Hive日期转分钟函数: minute
hive> select minute('2011-12-08 10:03:01') from tableName;
3
-- 11、Hive日期转秒函数: second
hive> select second('2011-12-08 10:03:01') from tableName;
1  

-- 12、Hive日期转周函数: weekofyear
hive> select weekofyear('2011-12-08 10:03:01') from tableName;
49

-- 13、Hive日期比较函数: datediff
hive> select datediff('2012-12-08','2012-05-09') from tableName;
213

-- 14、Hive日期增加函数: date_add 
hive> select date_add('2012-12-08',10) from tableName;
2012-12-18

-- 15、Hive日期减少函数: date_sub 
hive> select date_sub('2012-12-08',10) from tableName;
2012-11-28

-- 16、Hive指定日期的当月的最后一天：last_day
SELECT last_day('2021-02-22')
2021-02-28

-- 17、Hive截断返回指定格式单位的日期: trunc
select trunc('2015-03-17', 'MM')  from tableName;
2015-03-01 

-- 18、Hive获取日期之间的月份差：MONTHS_BETWEEN
SELECT MONTHS_BETWEEN('2008-05-05', '2008-04-05') FROM dual
-1
```



## Hive条件函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
| if(boolean testCondition, T valueTrue, T valueFalseOrNull) | T | 如果testCondition 为true就返回valueTrue,否则返回valueFalseOrNull   |
| isnull( a ) | boolean | 如果a为null就返回true，否则返回false  |
| isnotnull ( a ) | boolean | 如果a为非null就返回true，否则返回false |
| nvl(T value, T default_value)	 | T | 如果value值为NULL就返回default_value, 否则返回value，HIve 0.11后有 |
| COALESCE(T v1, T v2, ...) | T |  返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL |
| CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END | T | 如果a=b就返回c,a=d就返回e，否则返回f  |
| CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END | T | 如果a=ture就返回b,c= ture就返回d，否则返回e   |
| nullif( a, b ) | T | 如果a = b，则返回null，否则返回a ， CASE WHEN a = b then NULL else a，Hive 2.3.0后有 |
| assert_true(boolean condition) | void | 如果condition为真，则返回NULL，否则抛出异常  |


```hql
-- 1、If函数: if
hive> select if(1=2,100,200) from tableName;
200
hive> select if(1=1,100,200) from tableName;
100

-- 2、非空查找函数: COALESCE
hive> select COALESCE(null,'100','50') from tableName;
100

-- 3、条件判断函数：CASE
hive> Select case 100 when 50 then 'tom' when 100 then 'mary' else 'tim' end from tableName;
mary
hive> Select case 200 when 50 then 'tom' when 100 then 'mary' else 'tim' end from tableName;
tim

-- 4、条件判断函数：CASE
语法: CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END
hive> select case when 1=2 then 'tom' when 2=2 then 'mary' else 'tim' end from tableName;
mary
hive> select case when 1=1 then 'tom' when 2=2 then 'mary' else 'tim' end from tableName;
tom
```



## Hive字符串函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
| length(string A) | int | 返回字符串A的长度 |
| reverse(string A) | string | reverse(string A) |
| concat(string A, string B…) | string | 返回输入字符串连接后的结果，支持任意个输入字符串 |
| concat_ws(string SEP, string A, string B…) | string | 返回输入字符串连接后的结果，SEP表示各个字符串间的分隔符 |
| substr(string A, int start),substring(string A, int start) | string | 返回字符串A从start位置到结尾的字符串 |
| substr(string A, int start, int len),substring(string A, int start, int len) | string | 返回字符串A从start位置开始，长度为len的字符串 |
| upper(string A) | string | 字符串转大写函数：upper,ucase |
| lower(string A) lcase(string A) | string | 字符串转小写函数：lower,lcase |
| trim(string A) | string | 去除字符串两边的空格 |
| ltrim(string A) | string | 去除字符串左边的空格|
| rtrim(string A) | string | 右边去空格函数：rtrim |
| regexp_replace(string A, string B, string C) | string | 将字符串A中的符合java正则表达式B的部分替换为C。注意，在有些情况下要使用转义字符,类似oracle中的regexp_replace函数 |
| regexp_extract(string subject, string pattern, int index) | string | 将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符 |
| parse_url(string urlString, string partToExtract [, string keyToExtract]) | string | 返回URL中指定的部分。partToExtract的有效值为：HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, and USERINF |
| get_json_object(string json_string, string path) | string | 解析json的字符串json_string,返回path指定的内容。如果输入的json字符串无效，那么返回NULL。 |
| repeat(string str, int n) | string | 返回重复n次后的str字符串 |
| split(string str, string pat) | array | 按照pat字符串分割str，会返回分割后的字符串数组 |
| find_in_set(string str, string strList) | int | 返回str在strlist第一次出现的位置，strlist是用逗号分割的字符串。如果没有找该str字符，则返回0 |
| locate(string substr, string str[, int pos])  | int | 查找字符串str中的pos位置后字符串substr第一次出现的位置 |
| format_number(double str, int long)  | string | 转为千分位格式 |


```hql
-- 1、字符串长度函数：length
hive> select length('abcedfg') from tableName;
7

-- 2、字符串反转函数：reverse
hive> select reverse(abcedfg’) from tableName;
gfdecba

-- 3、字符串连接函数：concat
hive> select concat(‘abc’,'def’,'gh’) from tableName;
abcdefgh

-- 4、带分隔符字符串连接函数：concat_ws
hive> select concat_ws(',','abc','def','gh') from iteblog;
abc,def,gh

-- 5、字符串截取函数：substr,substring
hive> select substr('abcde',3) from tableName;
cde
hive> select substring('abcde',3) from tableName;
cde
hive>  select substr('abcde',-1) from tableName;  （和ORACLE相同）
e

-- 6、字符串截取函数：substr,substring
hive> select substr('abcde',3,2) from tableName;
cd
hive> select substring('abcde',3,2) from tableName;
cd
hive>select substring('abcde',-2,2) from tableName;
de

-- 7、字符串转大写函数：upper,ucase
hive> select upper('abSEd') from tableName;
ABSED
hive> select ucase('abSEd') from tableName;
ABSED

-- 8、字符串转小写函数：lower,lcase
hive> select lower('abSEd') from tableName;
absed
hive> select lcase('abSEd') from tableName;
absed

-- 9、去空格函数：trim
hive> select trim(' abc ') from tableName;
abc

-- 10、左边去空格函数：ltrim
hive> select ltrim(' abc ') from tableName;
abc

-- 11、右边去空格函数：rtrim
hive> select rtrim(' abc ') from tableName;
abc

-- 12、正则表达式替换函数：regexp_replace
hive> select regexp_replace('foobar', 'oo|ar', '') from tableName;
fb

-- 13、正则表达式解析函数：regexp_extract
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 1) from tableName;
the
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 2) from tableName;
bar
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 0) from tableName;
foothebar

-- 14、URL解析函数：parse_url
hive> select parse_url('https://www.iteblog.com/path1/p.php?k1=v1&k2=v2#Ref1', 'HOST') from tableName;
facebook.com
hive> select parse_url('https://www.iteblog.com/path1/p.php?k1=v1&k2=v2#Ref1', 'QUERY', 'k1') from tableName;
v1

-- 15、json解析函数：get_json_object
hive> select  get_json_object('{"store":
>   {"fruit":\[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}],
>    "bicycle":{"price":19.95,"color":"red"}
>   },
>  "email":"amy@only_for_json_udf_test.net",
>  "owner":"amy"
> }
> ','$.owner') from tableName;
amy

-- 16、空格字符串函数：space
hive> select space(10) from tableName;
hive> select length(space(10)) from tableName;
10

-- 17、重复字符串函数：repeat
hive> select repeat('abc',5) from tableName;
abcabcabcabcabc

-- 18、分割字符串函数: split
hive> select split('abtcdtef','t') from tableName;
["ab","cd","ef"]

-- 19、集合查找函数: find_in_set
hive> select find_in_set('ab','ef,ab,de') from tableName;
2
hive> select find_in_set('at','ef,ab,de') from tableName;
0

-- 20、千分位转换函数：format_number
hive> select format_number(21312442.12345, 3) from tableName;
21,312,442.123
```

## Hive集合统计函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
|  count(*), count(expr), count(DISTINCT expr[, expr_.]) | int | count(*)统计检索出的行的个数，包括NULL值的行；count(expr)返回指定字段的非空值的个数；count(DISTINCT expr[, expr_.])返回指定字段的不同的非空值的个数 |
| sum(col), sum(DISTINCT col) | double | sum(col)统计结果集中col的相加的结果；sum(DISTINCT col)统计结果中col不同值相加的结果 |
| avg(col), avg(DISTINCT col) | double | avg(col)统计结果集中col的平均值；avg(DISTINCT col)统计结果中col不同值相加的平均值 |
| min(col) | double | 统计结果集中col字段的最小值 |
| max(col) | double | 统计结果集中col字段的最大值 |
| collect_set (col) | array |  将 col 字段进行去重，合并成一个数组。 |
| collect_list (col) | array |  将 col 字段合并成一个数组,不去重 |

```SQL
-- 1、 统计个数函数: count
hive> select count(*) from tableName;
20
hive> select count(distinct t) from tableName;
10

-- 2、总和统计函数: sum
hive> select sum(t) from tableName;
100
hive> select sum(distinct t) from tableName;
70

-- 3、平均值统计函数: avg
hive> select avg(t) from tableName;
50
hive> select avg (distinct t) from tableName;
30

-- 4、最小值统计函数: min
hive> select min(t) from tableName;
20

-- 5、最大值统计函数: max
hive> select max(t) from tableName;
120

-- 6、集合去重数：collect_set
hive> select name,score from tableName;
dabai 20
dabai 20

hive> select name,collect_set(name) from tableName group by name;
dabai [20]

-- 7、集合不去重函数：collect_list
hive> select name,collect_list(name) from tableName group by name;
dabai [20, 20]

collect_set 函数和 collect_list 函数区别在与collect_list 不去重，collect_set 去重。
```


## Hive集合操作函数
| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
| array_contains(Array<T>, value)	 | boolean | 返回 Array<T>中是否包含元素 value |
| size(Map<K.V>)  | int | 返回Map的大小 |
| size(Array<T>  | int | 返回Array的大小 |
| map_keys(Map<K.V>)  |  array<K> | 返回 Map<K.V>中所有 key 的集合 |
| map_values(Map<K.V>) | array<V> | 返回 Map<K.V>中所有 value 的集合 |
| sort_array(Array<T>)	 | array<t> | 数组排序 |

```
-- 1、 判断元素数组是否包含元素：array_contains
hive> select array_contains(array(1,2,3,4,5),3) from tableName;
true

-- 2、 map 类型大小：size
hive> select size(map('k1','v1','k2','v2')) from tableName;
2

-- 3、array 类型大小：size
hive> select size(array(1,2,3,4,5)) from tableName;
5

-- 4、 获取 map 中所有 key 集合：map_keys
hive> select map_keys(map('k1','v1','k2','v2')) from tableName;
["k2","k1"]


-- 5、 获取 map 中所有 value 集合：map_values
hive> select map_values(map('k1','v1','k2','v2')) from tableName;
["v2","v1"]

-- 6、 数组排序：sort_array
hive> select sort_array(array(5,7,3,6,9)) from tableName;
[3,5,6,7,9]
```


## Hive类型转换函数
```
-- 1、类型转换函数：cast
hive> select cast(’1′ as bigint) from tableName;
1


-- timestamp 中的年/月/日的值是依赖与当地的时区，结果返回 date 类型
SELECT cast(timestamp as date)
-- 如果 string 是 yyyy-MM-dd 格式的，则相应的年/月/日的 date 类型的数据将会返回；
-- 但如果 string 不是 yyyy-MM-dd 格式的，结果则会返回 NULL
SELECT cast(string as date)
-- 基于当地的时区，生成一个对应 date 的年/月/日的时间戳值；
SELECT cast(date as timestamp)
-- date所代表的年/月/日时间将会转换成 yyyy-MM-dd 的字符串。
SELECT cast(date as string)
-- 转化为 bigint
SELECT cast('1' as BIGINT)
```


## Hive类型转换函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
|  cast(expr as <type>) | int | Hive类型转换。例如将字符”1″转换为整数:cast(’1′ as bigint)，如果转换失败返回NULL。 |

```
-- 1、类型转换函数：cast
hive> select cast(’1′ as bigint) from tableName;
1
```

## Hive表格生成转换函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
|  explode(ARRAY) | 多行 | 将数组中的元素拆分成多行显示 |
|  explode(MAP) | 多行 | 将数组中的元素拆分成多行显示 |

```
-- 1、数组拆分成多行：explode
hive> select explode(array(1,2,3)) from tableName;
OK
1
2
3

-- 2、Map 拆分成多行：explode
hive> select explode(map('k1','v1','k2','v2')) from tableName;
OK
k2 v2
k1 v1
```

## Hive混合函数

| 函数 | 类型 | 功能说明 |
| --- | --- | --- |
|  reflect | int |  |

```SQL

-- reflect (也可 java_method) 支持调用 java 自带函数，计算一行最高成绩
select reflect("java.lang.Math","max", englist, chinese) from tableName
-- 返回 1 true    3   2   3   2.718281828459045   1.0
SELECT reflect("java.lang.String", "valueOf", 1),
       reflect("java.lang.String", "isEmpty"),
       reflect("java.lang.Math", "max", 2, 3),
       reflect("java.lang.Math", "min", 2, 3),
       reflect("java.lang.Math", "round", 2.5),
       reflect("java.lang.Math", "exp", 1.0),
       reflect("java.lang.Math", "floor", 1.9)
```


## 其他

### WITH AS 短语
也叫做子查询部分（subquery factoring），可以定义一个SQL片断，该SQL片断会被整个SQL语句用到。一般当我们书写一些结构相对复杂的SQL语句时，可能某个子查询在多个层级多个地方存在重复使用的情况，这个时候我们可以使用 with as 语句将其独立出来，极大提高SQL可读性，简化SQL。目前 oracle、sql server、hive、MySQL8.0 等均支持 with as 用法。

```
-- 相当于建了 a、b 临时表
with
     a as (select * from scott.emp),
     b as (select * from scott.dept)

select * from a, b where a.deptno = b.deptno;
```

### UNION语句
SQL UNION 可以将两个要连接的 SQL 语句拼接在一起，它们字段个数必须一样，而且字段类型要“相容”（一致）。
```
-- UNION 操作符合并的结果集，不允许重复值；
select class, teacher as t from class
union
select class, teacher as t from class


-- UNION ALL 允许有重复值
select class, teacher as t from class
union all
select class, teacher as t from class


-- 整合操作，一般会在 UNION 的外层做一些操作，而不是对连接前的数据进行操作，比如排序，去重等等，这里使用排序举例，
select *
from (
         select class, teacher as name
         from class
         union
         select class, name as name
         from students
     )
order by class desc
```