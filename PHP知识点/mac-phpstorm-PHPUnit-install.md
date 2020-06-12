---
title: Mac Os 上使用 PHPStorm 进行 PHPUnit单元测试
permalink: mac-phpstorm-PHPUnit-install
date: 2020-01-07 00:23:49
updated: 2020-01-07 21:12:50
tags: 
    - php
    - phpstorm
categories: php
toc: true
excerpt: PHPUnit是一个轻量级的PHP测试框架。它是在PHP5下面对JUnit3系列版本的完整移植，是xUnit测试框架家族的一员(它们都基于模式先锋Kent Beck的设计)。
---

PHPUnit是一个轻量级的PHP测试框架。它是在PHP5下面对JUnit3系列版本的完整移植，是xUnit测试框架家族的一员(它们都基于模式先锋Kent Beck的设计)。


### 什么是 PHPUnit？
PHPUnit是一个轻量级的PHP测试框架。它是在PHP5下面对JUnit3系列版本的完整移植，是xUnit测试框架家族的一员(它们都基于模式先锋Kent Beck的设计)。


### 解决问题
- 如果你想做个接口测试，但并不想公开内部接口
- 如果你只是想对自己封装的某块代码做个小测试
- 如果你想要编写代码边调试，又不想操作 Postman 或前端的功能来调用 API

### 参考文档
>PHPUnit 手册：http://www.phpunit.cn/manual/...
phpunit assert断言分类整理 ：https://www.cnblogs.com/nings...

### 安装配置

#### 项目目录中安装 `phpunit/phpunit`
```
➜  app cd amazon-advertising
➜  amazon-advertising git:(master) ✗ composer require phpunit/phpunit
```
#### 在项目目录中新建单元测试文件`tests`
```
mkdir tests
```

### 配置 phpunit.xml
在你的项目根目录下新建 phpunit.xml 文件，为引入autoload.php。
配置内容：
```
<?xml version="1.0" encoding="UTF-8"?>
<!--bootstrap指定启动测试时, 先加载vendor/autoload.php-->
<phpunit backupGlobals="false"
         backupStaticAttributes="false"
         bootstrap="vendor/autoload.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false">

    <!--testsuite指定测试文件的目录-->
    <testsuite name="service">
        <directory>tests</directory>
    </testsuite>

    <!--filter过滤依赖文件的位置-->
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">./src</directory>
        </whitelist>
    </filter>
</phpunit>
```

### 配置 PHPStorm 的 PHP CLi
![](https://static.studytime.xin/image/articles/20191113231957.png)

### 配置 PHPUnit
![](https://static.studytime.xin/image/articles/20191113232112.png)

### 新增测试例
![](https://static.studytime.xin/image/articles/20191113232339.png)

![](https://static.studytime.xin/image/articles/20191113232512.png)

```
<?php

use PHPUnit\Framework\TestCase;

class ConverterTest extends TestCase
{
    public function testHello()
    {
        $this->assertEquals('Hello', 'Hell' . 'o');
    }
}
```

### 特殊说明
- 测试用例类名称必须为Test结尾`ConverterTest`
- 测试用例必须继承`PHPUnit\Framework\TestCase`
- 测试用例方法名必须以test开头`testHello`

### 参考文件
- [https://dengxiaolong.com/article/2018/07/phpunit-on-phpstorm.html](https://dengxiaolong.com/article/2018/07/phpunit-on-phpstorm.html)
- [https://segmentfault.com/a/1190000016323574](https://segmentfault.com/a/1190000016323574)
