---
title: Hexo 解决使用 Gitalk 登录授权报 403 的问题
permalink: hexo-gitalk-error
tags: 
    - icarus
    - Blog
categories: [博客,icarus]
thumbnail: https://static.studytime.xin//studytime/image/articles/HZhIP6.jpg
date: 2021-05-22 22:10:43
updated: 2021-05-22 22:10:46
toc: true
excerpt:  Hexo 解决使用 Gitalk 登录授权报 403 的问题
---

最近一段时间个人博客使用的 gitalk 在登录 github 授权时经常出现错误 `error: request failed with status code 403` 的问题，到底是什么原因造成的和该怎样解决呢？

### 原因排查

通过检查网络请求发现获取Github Token时请求了以下链接 `https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token`，查询GitTalk官方文档发现github.com的oauth是不允许跨域请求的，cors-anywhere.herokuapp.com是一个第三方提供的CORS代理服务，会默认放行所有CORS请求。目前由于该CORS代理服务遭到滥用，因此做了限制，导致GitTalk失效。

### 解决方案

通过nginx在博客域名上配置反向代理，替换第三方的代理服务域名即可解决。

#### nginx反向代理配置

```
location /github {
   # 配置www.studytime.xin跨域，避免滥用风险
	add_header Access-Control-Allow-Origin www.studytime.xin;
  add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
  add_header Access-Control-Allow-Headers 'DNT,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
  if ($request_method = 'OPTIONS') {
  	return 204;
  }
  proxy_pass https://github.com/; # 尾部斜杠不能少
}
```

使用`nginx -s reload`命令重启nginx服务。

### 修改gitalk初始化参数

作者使用的是hexo+icarus主题，其他主题或者博客系统也是类似做法，编辑themes/icarus/layout/comment/gitalk.ejs

```javascript
<script>
    var gitalk = new Gitalk({
        clientID: '<%= get_config('comment.client_id') %>',
        clientSecret: '<%= get_config('comment.client_secret') %>',
        id: '<%= md5(page.path) %>',
        repo: '<%= get_config('comment.repo') %>',
        owner: '<%= get_config('comment.owner') %>',
        admin: <%- JSON.stringify(get_config('comment.admin'))%>,
        createIssueManually: <%= get_config('comment.create_issue_manually', false) %>,
        distractionFreeMode: <%= get_config('comment.distraction_free_mode', false) %>,
        proxy: '/github/login/oauth/access_token' // 新添加的
    })
    gitalk.render('comment-container')
</script>
```

### 访问测试

![image-20210525225150253](https://static.studytime.xin//studytime/image/articles/image-20210525225150253.png)

查看Chrome网络状况，可以看到已经走了自己配置的CORS跨域了。

```
Request URL: https://www.studytime.xin/github/login/oauth/access_token
Request Method: POST
Status Code: 200 
Remote Address: 106.52.24.199:443
Referrer Policy: strict-origin-when-cross-origin
```

