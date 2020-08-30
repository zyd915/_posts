---
title: Flink 如何自定义mysql source与sink，实现mysql的读取和写入
permalink: flink-mysql-source-sink
date: 2020-08-30 17:30:38
updated: 2020-08-30 17:30:39
tags:
    - 大数据
    - flink
categories: flink
toc: true
excerpt: Flink 如何自定义mysql source与sink，实现mysql的读取和写入
---

## 数据库表准备

```sql
show databases;

create database flink;

use flink

CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `name` varchar(255) NOT NULL DEFAULT '' COMMENT '用户名称',
  `address` varchar(200) NOT NULL DEFAULT '' COMMENT '住址',
  `sex` tinyint(1) NOT NULL DEFAULT '0' COMMENT '性别[0:未知,1:男,2:女]',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='用户基础信息表';


INSERT INTO user  (name, address,sex)  VALUES  ('xiaoming','Beijing',1),('xiaoqiang','GuangZhou',2),('xiaohua','Hubei',0),('xiaoli','GuangZhou',1)


select * from user;
+----+-----------+-----------+-----+---------------------+---------------------+
| id | name      | address   | sex | create_time         | update_time         |
+----+-----------+-----------+-----+---------------------+---------------------+
| 1  | xiaoming  | Beijing   | 1   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 2  | xiaoqiang | GuangZhou | 2   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 3  | xiaohua   | Hubei     | 0   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 4  | xiaoli    | GuangZhou | 1   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
+----+-----------+-----------+-----+---------------------+---------------------+
```


## 通过maven在pom.xml中添加驱动依赖
```sql
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.35</version>
</dependency>
```


## 定义User实体类，用于封装数据
```java
package xin.studytime.java.model;

public class User {
    public int id;
    public String name;
    public String address;
    public int sex;

    public User(int id, String name, String address, int sex) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.sex = sex;
    }

    public User(String name, String address, int sex) {
        this.name = name;
        this.address = address;
        this.sex = sex;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", sex=" + sex +
                '}';
    }
}

```
## 实现flink 自定义mysql source
```java
package xin.studytime.java.source;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import xin.studytime.java.model.User;

import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.Connection;
import java.sql.ResultSet;

public class UserSourceFromMysql extends RichSourceFunction<User> {
    PreparedStatement preparedStatement;
    private Connection connection;

    public UserSourceFromMysql() {
        super();
    }

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open( parameters );
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://node02:3306/flink";
        String username = "root";
        String password = "!Qaz123456";

        Class.forName( driver );
        connection = DriverManager.getConnection( url, username, password );
        String sql = "select * from user;";
        preparedStatement = connection.prepareStatement( sql );
    }

    @Override
    public void close() throws Exception {
        super.close();
        if (connection != null) {
            connection.close();
        }
        if (preparedStatement != null) {
            preparedStatement.close();
        }
    }

    @Override
    public void run(SourceContext<User> sourceContext) throws Exception {
        try {
            ResultSet resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                User user = new User( resultSet.getInt( "id" ),
                        resultSet.getString( "name" ).trim(),
                        resultSet.getString( "address" ).trim(),
                        resultSet.getInt( "sex" ) );
                sourceContext.collect( user );
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void cancel() {

    }
}

```


## flink 主程序引入source，测试数据读取
```java
package xin.studytime.java;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import xin.studytime.java.model.User;

public class UserSourceFromMysql {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<User> userDataStreamSource = env.addSource( new xin.studytime.java.source.UserSourceFromMysql() );

        userDataStreamSource.print();

        env.execute();
    }
}
```
运行结果：
```shell
/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/bin/java -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:58791,suspend=y,server=n -javaagent:/Users/baihe/Library/Caches/JetBrains/IntelliJIdea2020.1/captureAgent/debugger-agent.jar -Dfile.encoding=UTF-8 -classpath /Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/deploy.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/jfxrt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/javaws.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/jfxswt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/plugin.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/ant-javafx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/javafx-mx.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/packager.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/lib/tools.jar:/Users/baihe/Code/javaCode/flink-project/target/classes:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-library.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-xml_2.11-1.0.4.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-parser-combinators_2.11-1.0.4.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-actors-migration_2.11-1.1.0.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-actors-2.11.0.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-swing_2.11-1.0.2.jar:/Users/baihe/Code/tools/scala-2.11.8/lib/scala-reflect.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-streaming-java_2.11/1.9.0/flink-streaming-java_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-core/1.9.0/flink-core-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-annotations/1.9.0/flink-annotations-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-metrics-core/1.9.0/flink-metrics-core-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-shaded-asm-6/6.2.1-7.0/flink-shaded-asm-6-6.2.1-7.0.jar:/Users/baihe/.m2/repository/org/apache/commons/commons-lang3/3.3.2/commons-lang3-3.3.2.jar:/Users/baihe/.m2/repository/com/esotericsoftware/kryo/kryo/2.24.0/kryo-2.24.0.jar:/Users/baihe/.m2/repository/com/esotericsoftware/minlog/minlog/1.2/minlog-1.2.jar:/Users/baihe/.m2/repository/org/objenesis/objenesis/2.1/objenesis-2.1.jar:/Users/baihe/.m2/repository/commons-collections/commons-collections/3.2.2/commons-collections-3.2.2.jar:/Users/baihe/.m2/repository/org/apache/commons/commons-compress/1.18/commons-compress-1.18.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-runtime_2.11/1.9.0/flink-runtime_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-java/1.9.0/flink-java-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-queryable-state-client-java/1.9.0/flink-queryable-state-client-java-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-hadoop-fs/1.9.0/flink-hadoop-fs-1.9.0.jar:/Users/baihe/.m2/repository/commons-io/commons-io/2.4/commons-io-2.4.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-shaded-netty/4.1.32.Final-7.0/flink-shaded-netty-4.1.32.Final-7.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-shaded-jackson/2.9.8-7.0/flink-shaded-jackson-2.9.8-7.0.jar:/Users/baihe/.m2/repository/commons-cli/commons-cli/1.3.1/commons-cli-1.3.1.jar:/Users/baihe/.m2/repository/org/javassist/javassist/3.19.0-GA/javassist-3.19.0-GA.jar:/Users/baihe/.m2/repository/com/typesafe/akka/akka-actor_2.11/2.5.21/akka-actor_2.11-2.5.21.jar:/Users/baihe/.m2/repository/com/typesafe/config/1.3.3/config-1.3.3.jar:/Users/baihe/.m2/repository/org/scala-lang/modules/scala-java8-compat_2.11/0.7.0/scala-java8-compat_2.11-0.7.0.jar:/Users/baihe/.m2/repository/com/typesafe/akka/akka-stream_2.11/2.5.21/akka-stream_2.11-2.5.21.jar:/Users/baihe/.m2/repository/org/reactivestreams/reactive-streams/1.0.2/reactive-streams-1.0.2.jar:/Users/baihe/.m2/repository/com/typesafe/ssl-config-core_2.11/0.3.7/ssl-config-core_2.11-0.3.7.jar:/Users/baihe/.m2/repository/com/typesafe/akka/akka-protobuf_2.11/2.5.21/akka-protobuf_2.11-2.5.21.jar:/Users/baihe/.m2/repository/com/typesafe/akka/akka-slf4j_2.11/2.5.21/akka-slf4j_2.11-2.5.21.jar:/Users/baihe/.m2/repository/org/clapper/grizzled-slf4j_2.11/1.3.2/grizzled-slf4j_2.11-1.3.2.jar:/Users/baihe/.m2/repository/com/github/scopt/scopt_2.11/3.5.0/scopt_2.11-3.5.0.jar:/Users/baihe/.m2/repository/org/xerial/snappy/snappy-java/1.1.4/snappy-java-1.1.4.jar:/Users/baihe/.m2/repository/com/twitter/chill_2.11/0.7.6/chill_2.11-0.7.6.jar:/Users/baihe/.m2/repository/com/twitter/chill-java/0.7.6/chill-java-0.7.6.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-clients_2.11/1.9.0/flink-clients_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-optimizer_2.11/1.9.0/flink-optimizer_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-shaded-guava/18.0-7.0/flink-shaded-guava-18.0-7.0.jar:/Users/baihe/.m2/repository/org/apache/commons/commons-math3/3.5/commons-math3-3.5.jar:/Users/baihe/.m2/repository/org/slf4j/slf4j-api/1.7.15/slf4j-api-1.7.15.jar:/Users/baihe/.m2/repository/com/google/code/findbugs/jsr305/1.3.9/jsr305-1.3.9.jar:/Users/baihe/.m2/repository/org/apache/flink/force-shading/1.9.0/force-shading-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-streaming-scala_2.11/1.9.0/flink-streaming-scala_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/apache/flink/flink-scala_2.11/1.9.0/flink-scala_2.11-1.9.0.jar:/Users/baihe/.m2/repository/org/scala-lang/scala-reflect/2.11.12/scala-reflect-2.11.12.jar:/Users/baihe/.m2/repository/org/scala-lang/scala-library/2.11.12/scala-library-2.11.12.jar:/Users/baihe/.m2/repository/org/scala-lang/scala-compiler/2.11.12/scala-compiler-2.11.12.jar:/Users/baihe/.m2/repository/org/scala-lang/modules/scala-xml_2.11/1.0.5/scala-xml_2.11-1.0.5.jar:/Users/baihe/.m2/repository/org/scala-lang/modules/scala-parser-combinators_2.11/1.0.4/scala-parser-combinators_2.11-1.0.4.jar:/Users/baihe/.m2/repository/org/slf4j/slf4j-nop/1.7.2/slf4j-nop-1.7.2.jar:/Users/baihe/.m2/repository/mysql/mysql-connector-java/5.1.35/mysql-connector-java-5.1.35.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar xin.studytime.java.UserSourceFromMysql
Connected to the target VM, address: '127.0.0.1:58791', transport: 'socket'
6> User{id=2, name='xiaoqiang', address='GuangZhou', sex=2}
5> User{id=1, name='xiaoming', address='Beijing', sex=1}
2> User{id=6, name='lisi', address='Hunan', sex=1}
1> User{id=5, name='zhangsan', address='Beijing', sex=1}
7> User{id=3, name='xiaohua', address='Hubei', sex=0}
8> User{id=4, name='xiaoli', address='GuangZhou', sex=1}
Disconnected from the target VM, address: '127.0.0.1:58791', transport: 'socket'

Process finished with exit code 0
```


## 实现flink 自定义mysql sink
```java
package xin.studytime.java.sink;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import xin.studytime.java.model.User;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

public class UserSinkToMysql extends RichSinkFunction<User> {
    private Connection connection = null;
    private PreparedStatement preparedStatement = null;

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open( parameters );
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://node02:3306/flink";
        String username = "root";
        String password = "!Qaz123456";
        Class.forName( driver );
        connection = DriverManager.getConnection( url, username, password );
        String sql = "insert into user(name,address,sex)values(?,?,?);";
        preparedStatement = connection.prepareStatement( sql );
    }

    @Override
    public void close() throws Exception {
        super.close();
        if (connection != null) {
            connection.close();
        }
        if (preparedStatement != null) {
            preparedStatement.close();
        }
    }

    @Override
    public void invoke(User value, Context context) throws Exception {
        try {
            preparedStatement.setString( 1, value.getName() );
            preparedStatement.setString( 2, value.getAddress() );
            preparedStatement.setInt( 3, value.getSex() );
            preparedStatement.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


## flink 主程序引入sink，测试数据写入mysql
```java
package xin.studytime.java;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import xin.studytime.java.model.User;

public class UserSinkToMysql {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<User> users = env.fromElements( new User( "zhangsan", "Beijing", 1 ), new User( "lisi", "Hunan", 1 ) );

        users.addSink( new xin.studytime.java.sink.UserSinkToMysql() );


        env.execute();
    }
}
```
运行结果：
```shell
+----+-----------+-----------+-----+---------------------+---------------------+
| id | name      | address   | sex | create_time         | update_time         |
+----+-----------+-----------+-----+---------------------+---------------------+
| 1  | xiaoming  | Beijing   | 1   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 2  | xiaoqiang | GuangZhou | 2   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 3  | xiaohua   | Hubei     | 0   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 4  | xiaoli    | GuangZhou | 1   | 2020-08-30 15:27:05 | 2020-08-30 15:27:05 |
| 5  | zhangsan  | Beijing   | 1   | 2020-08-30 16:11:28 | 2020-08-30 16:11:28 |
| 6  | lisi      | Hunan     | 1   | 2020-08-30 16:11:28 | 2020-08-30 16:11:28 |
+----+-----------+-----------+-----+---------------------+---------------------+
```
