---
title: centos7 下 jekins 的安装和使用
date: 2020-02-08 20:00:50
updated: 2020-02-08 21:23:50
tags: 
    - centos
    - jenkins
categories: tool
toc: true
excerpt: 本文主要记录了centos7下的jekins的安装步骤，以及常见会出问题的细节点，对此做的一个总结记录。
---

本文主要记录了centos7下的jekins的安装步骤，以及常见会出问题的细节点，对此做的一个总结记录。

### jdk配置安装
前置条件需要安装jdk,具体安装方法不再此说明。
```
java -version 确定查看java版本，需要在1.8以上
```

### 安装jenkins

添加Jenkins库到yum库，Jenkins将从这里下载安装。

```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install -y jenkins
```

如果不能安装就到官网下载jenkis的rmp包，可以使用下面的国内镜像
```
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.204.5-1.1.noarch.rpm
jenkins-2.204.5-1.1.noarch.rpm
```

### 配置jenkis的端口
```
vi /etc/sysconfig/jenkins
找到修改端口号：
JENKINS_PORT="8080"  此端口不冲突可以不修改
```
### 配置jdk路径
```
# 查看java安装路径
which java

# 修改配置
vim /etc/init.d/jenkins

# 搜索查询 /usr/bin/java 替换成 你安装的java路径
```
若此步骤不进行配置，启动jekins时，则会报错。具体提示信息为：
`Starting jenkins (via systemctl):  Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.`

根据提示使用命令systemctl status jenkins.service可以看到启动的失败详情
![](https://static.studytime.xin/image/articles/20200319231402.png)

### 启动jenkins
```
service jenkins start/stop/restart
```
- 安装成功后Jenkins将作为一个守护进程随系统启动
- 系统会创建一个“jenkins”用户来允许这个服务，如果改变服务所有者，同时需要修改/var/log/jenkins, /var/lib/jenkins, 和/var/cache/jenkins的所有者
- 启动的时候将从/etc/sysconfig/jenkins获取配置参数
- 默认情况下，Jenkins运行在8080端口，在浏览器中直接访问该端进行服务配置
- Jenkins的RPM仓库配置被加到/etc/yum.repos.d/jenkins.repo

### 打开jenkins 

首次进入会要求输入初始密码，使用 `cat /var/lib/jenkins/secrets/initialAdminPassword ` 命令可以查看初始化密码。

![](https://static.studytime.xin/image/articles/20200319230351.png)

此时打开页面可能会提示 `Please wait while Jenkins is getting ready to work`，长时间没反应，则按照以下步骤处理。
```
vim /var/lib/jenkins/hudson.model.UpdateCenter.xml

https://updates.jenkins.io/update-center.json" 
修改为
http://mirror.xmission.com/jenkins/updates/update-center.json

并重启服务
systemctl daemon-reload 
```

### 选择安装插件
选择“Install suggested plugins”安装默认的插件，下面Jenkins就会自己去下载相关的插件进行安装。这个界面需要停留很久时间。

![](https://static.studytime.xin/image/articles/20200319231916.png)

### 创建超级管理员账号 
![](https://static.studytime.xin/image/articles/20200319231954.png)

### 安装完成
![](https://static.studytime.xin/image/articles/20200319232009.png)
