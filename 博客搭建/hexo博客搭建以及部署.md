---
title: hexo博客搭建以及部署到服务器方案
permalink: hexo-deploy-server
tags: 
    - Blog
categories: [博客, hexo]
date: 2020-07-10 23:10:43
updated: 2020-07-10 23:10:22
toc: true
excerpt: 本文主要整理了hexo博客搭建以及部署到服务器方案，希望对大家有所帮助。
---

### 安装git
`yum install git`

### 安装node.js

### 服务器端创建git用户
```
useradd git
passwd git

# 给git用户配置sudo权限
chmod 740 /etc/sudoers
vim /etc/sudoers
# 找到root ALL=(ALL) ALL，在它下方加入一行
git ALL=(ALL) ALL

chmod 400 /etc/sudoers
```

### 服务器端git用户配置免密登录
```
su - git
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorzied_keys
chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys    #拷贝本地秘钥粘贴进去
```

###  服务器端创建git仓库并视同git-hook实现自动部署
```
sudo mkdir -p /data/repo    #新建目录，这是git仓库的位置
sudo mkdir pp /data/wwwroot/blog
cd /data/repo   #转到git仓库的文件夹
sudo git init --bare blog.git #创建一个名叫blog的仓库
sudo vim /data/repo/blog.git/hooks/post-update
```

#### post-update的内如如下
```
#!/bin/bash
git --work-tree=/data/wwwroot/blog --git-dir=/data/repo/blog.git checkout -f
```

#### 给post-update文件权限
```
cd /data/repo/blog.git/hooks/
sudo chown -R git:git /data/repo/
sudo chmod +x post-update  #赋予其可执行权限
```

### 配置nginx
```
cd /usr/local/nginx/conf/vhost
mkdir -p /data/log/blog/
vim blog.conf

server {
  listen        80;
  server_name   studytime.xin www.studytime.xin;
  rewrite ^(.*)$ https://${server_name}$1 permanent;
}


server {
  listen       443;
  server_name  www.studytime.xin;
  ssl          on;
  root /data/wwwroot/blog;
  index index.html;
  ssl_certificate  /usr/local/nginx/ssl/studytime/3193593_studytime.xin.pem;
  ssl_certificate_key  /usr/local/nginx/ssl/studytime/3193593_studytime.xin.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
  ssl_prefer_server_ciphers on;
  access_log /data/log/blog/access.log;
  error_log  /data/log/blog/error.log;
}
```

#### 重启nginx
```
nginx -t 
nginx -s reload
```

## 本地hexo项目（mac）
### 前置条件
- 安装git
- 安装node.js、npm
- 安装hexo以及扩展
```
sudo npm install hexo-cli hexo-server hexo-deployer-git -g
```

### 初始化常见hexo博客项目和站点
```
hexo init ~/blog
npm install hexo-deployer-git --save
```

### 修改hexo站点配置
```
# 修改Hexo的deploy配置
cd blog
vim _config.yml

# 找到deploy配置部分
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: root@xxx.xx.xxx.xxx:/var/repo/blog.git # IP填写自己服务器的IP即可
  branch: master
```

### 本地hexo生成站点以及部署
```
# 清除缓存
hexo clean

# 生成静态页面
hexo generate

# 将本地静态页面目录部署到云服务器
hexo delopy
```

## 浏览器打开站点
[白程序员的自习室](https://www.studytime.xin/)