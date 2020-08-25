---
title: icarus 文章置顶功能实现
permalink: icarus-top
tags: 
    - icarus
    - Blog
categories: Blog
thumbnail: https://static.studytime.xin/article/20200710010415.png
date: 2020-07-09 23:10:43
updated: 2020-07-09 23:10:22
toc: true
excerpt: 本文主要整理了 icarus 文章置顶功能实现方案，希望对大家有所帮助。
---

icarus主题整体来讲还是集成的比较好的，例如评论，打赏等等。但是每个人想法一样，作者因为特殊需要，需要对特定文章进行置顶，但是经过确定，发现icarus原作者，暂时没有计划实现，所以此处整理了自己实现排序的想法，希望对其他小伙伴有帮助。

### 在配置文件中，增加top属性
```
index_generator:
  path: ''
  per_page: 10
  #  order_by: '-date'
  order_by:
    top: -1
    date: -1
```

### 修改generator.js，主要是在生成html代码时优先按照top排序
```
/⁨node_modules⁩/hexo-generator-index⁩/lib⁩/generator.js


const config = this.config;
  var paginationDir = config.pagination_dir || 'page';
  const posts = locals.posts.sort(config.index_generator.order_by);
  
  posts.data = posts.data.sort(function(a, b) {
    if(a.top && b.top) {
      if(a.top == b.top) return b.date - a.date;
      else return b.top - a.top;
    }
    else if(a.top && !b.top) {
      return -1;
    }
    else if(!a.top && b.top) {
      return 1;
    }
    else return b.date - a.date;
  });
  var path = config.index_generator.path || '';
```

![](https://static.studytime.xin/article/20200825223623.png)


### 修改前端样式标签
```
/themes/icarus/layout/common/article.jsx

{/*置顶图标*/}
                            {page.top > 0 ?
                                <div class="level-item tag is-danger"
                                     style="background-color: #3273dc;">已置顶</div> : null}
```


### 修改模板中的post.md，添加top属性并设置值为102，值越大权重越高，排位越高
```
source/_post/learn-resource.md

---
title: 超过1024G的IT学习资料免费领取（大数据、Java、Python等）
permalink: learn-resource
date: 2020-06-22 00:23:49
updated: 2020-06-22 00:23:49
tags: 
    - tool
categories: tool
toc: true
top: 102
thumbnail: https://static.studytime.xin/article/20200825214457.jpg
excerpt: 超过 1024G 的 IT 学习资料免费领取（大数据、Java、Python等），你值得拥有！
---
```

### 样式效果展示
![](https://static.studytime.xin/article/20200825224535.png)



