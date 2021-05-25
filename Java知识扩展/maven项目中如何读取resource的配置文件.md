---
title: Maven项目中读取src/main/resources目录下的配置文件
permalink: maven-resource-conf
thumbnail: https://static.studytime.xin//studytime/image/articles/4ac1b82yy9bc491959e9e7c6e277d8f8.jpeg
date: 2021-05-25 19:10:27
updated: 2021-05-25 19:10:23
tags: 
    - maven
    - java
categories: java
toc: true
excerpt: 在Maven项目的开发中，当需要读取src/下的配置文件时，该怎么做？

---

在Resources下有一个文件名为kafka.properties的配置文件，怎么加载呢，下面整理了一些方法和说明。

### 一、在java类中读取
若配置文件不在src/main/resources目录下，可以直接使用

```java
Properties prop = new properties();
prop.load(new InputStream("kafka.properties"));
```

当配置文件放在src/main/resources的目录下时，只能使用Class.getResourceAsStream()方法来加载

```java
Properties prop = new properties();
prop.load(this.getClass().getResourceAsStream("/kafka.properties"));
```

此时，getResourceAsStream(String name)方法中参数路径的写法：

- 若写成"kafka.properties"，则是去当前类的class文件同一目录下找（但是显然在正常项目不会有人将配置文件放在这种位置）。

- 若写成"/kafka.properties"，则是去整个项目的classes目录下去找，即target/classes

### 二、在spring框架的xml文件中读取

```vbnet
// 配置文件kafka.properties
kafka.topic=topic
serializer.class=kafka.serializer.StringEncoder
key.serializer.class=kafka.serializer.StringEncoder
```

- 首先可以在spring的bean中配置

```html
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
  <property name="locations">
    <list>
      <value>/kafka.properties</value>
    </list>
  </property>
</bean>
```

这里还可以在list标签中配置多个value，这样就可以在bean中读取一个甚至多个配置文件。



```html
<bean id="kafkaService" class="com.wws.service.impl.KafkaServiceImpl">
	<!-- <property name="topic"><value>topic</value></property> -->
	<property name="topic"><value>${kafka.topic}</value></property>
</bean>
```

这样就可以在后面的bean中成功调用配置文件中的参数，以上被注释的那段property和被注释掉的那行是同样效果

- 或者也可以使用如下方法

```html
<context:property-placeholder location="classpath:kafka.properties"/>
```

直接在spring配置文件中配置context:property-placeholder，有多个配置文件可以用逗号隔开，例如

```html
<context:property-placeholder location="classpath:kafka.properties,classpath:jdbc.properties"/>
```

调用的方法跟上面一样，这里就不赘述了。