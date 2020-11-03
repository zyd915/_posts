---
title: hexo博客搭建以及部署到服务器方案
permalink: hexo-deploy-server
tags: 
    - Blog
categories:[博客,hexo]
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



# 如果服务器没有安装git的话，需要安装git。


此外，因为我的服务器上正在跑着我的个人网站，它已经占用了80端口，要再加一个网站的话需要做反向代理，于是需要安装nginx。


首先把我的个人网站的端口改成10000备用，把80端口空出来，然后安装Nginx：`


sudo yum -y install nginx
nginx # 启动nginx试一下

建立Git仓库

Hexo可以使用Git来部署，这样每次写完之后就都可以使用git来一键部署了，比较方便。


注意：如果本地没有配置ssh，先配一个ssh。这里因为我已经有了，就不再配置了。


开始前先切到root用户，然后执行：


useradd git # 添加一个新用户
passwd git # 设置git用户密码
su git # 切换用户进行后续操作
cd /home/git/
mkdir -p projects/blog # 把项目目录建立起来
mkdir repos && cd repos
git init --bare blog.git # 创建仓库
cd blog.git/hooks
vim post-receive # 创建一个钩子

post-receive文件的内容如下：


#!/bin/sh
git --work-tree=/home/git/projects/blog --git-dir=/home/git/repos/blog.git checkout -f

之后退出vim，继续进行用户相关的操作：


chmod +x post-receive # 添加可执行权限
exit # 返回到root用户
chown -R git:git /home/git/repos/blog.git # 给git用户添加权限

这样Git仓库就配好了。在本地试一下能不能把空仓库拉下来：


git clone git@server_ip:/home/git/repos/blog.git

如果能拉下来，就说明配置成功了。


本地建立SSH信任关系

目前每次对git仓库进行操作都需要输入密码，不太方便。但是我们已经配置了SSH，就可以通过建立SSH信任关系来免去输入密码的步骤：


ssh-copy-id -i ~/.ssh/id_rsa.pub git@server_ip # 建立信任关系
ssh git@server_ip # 试一下能不能登录

如果不能登录或者还是要输入密码，就说明前面的操作有误，再检查一下吧。


更改git用户默认的shell

为了安全起见，这个部署用的git用户最好只能用git clone等操作，所以需要更改它默认的shell：


cat /etc/shells # 查看 git-shell 是否在登录方式里面
which git-shell # 找到git-shell的路径，记下来
vim /etc/shells

然后把刚才记下来的路径添加进去，保存，然后vim /etc/passwd，把git:x:1000:1000::/home/git:/bin/bash修改为git:x:1000:1000:,,,:/home/git:/usr/bin/git-shell。


这样本地再用ssh应该就没法登录了，只能进行git操作。


部署上线

有了git仓库，就可以部署hexo文件上去了。


首先改一下_config.yml：


deploy:
  type: git
  repo: git@your_ip:/home/git/repos/blog.git
  branch: master

然后安装hexo-deployer-git，否则没法使用git部署：


sudo npm install --save hexo-deployer-git

编辑package.json文件，添加部署脚本：


"scripts": {
&#34;build&#34;: &#34;hexo generate&#34;,
&#34;clean&#34;: &#34;hexo clean&#34;,
&#34;deploy&#34;: &#34;hexo clean &amp;&amp; hexo g -d&#34;,
&#34;server&#34;: &#34;hexo server&#34;
},


然后执行npm run deploy就可以把文件部署上去了，出现类似于下面的提示，就说明部署成功。


配置Nginx反向代理

首先需要配置一下域名解析记录，把tutorial.wendev.site也绑定到现在的ip上，以便进行下一步操作。


前面配置完仓库，也把文件部署上去了，我们把Nginx反向代理也来配置一下。


cd /etc/nginx
cp nginx.conf nginx_backup.conf # 备份配置文件
vim nginx.conf

主要是开头的


user root;

和服务器部分的


    server {
    listen       80;
    server_name  www.wendev.site;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://localhost:10000;
    }
}

server {
listen      80;
server_name wendev.site;

# Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:10000;
    }
}

server {
listen 80;
server_name tutorial.wendev.site;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        root /home/git/projects/blog;
    index index.html index.htm;
    }

error_page 500 502 503 504 /50x.html;
location &#61; /50x.html {
    root html;
}
}


这样就可以把两个不同的域名 www.wendev.site和tutorial.wendev.site 分别绑定到10000端口（已有的个人网站）和新的hexo网站了。


然后重新加载一下配置：


nginx -s reload

访问tutorial.wendev.site、wendev.site和www.wendev.site，发现三个网站都正常，大功告成！