---
title: Icarus 使用 Fancybox 为博客添加图片放大预览功能
permalink: icarus-fancybox
tags: 
    - icarus
    - Blog
categories:[博客,icarus]
thumbnail: https://static.studytime.xin/article/20200710010415.png
date: 2020-07-09 22:10:43
updated: 2020-07-09 22:10:22
toc: true
excerpt: FancyBox 是一款基于jQuery的弹出库，用于呈现各种类型的媒体。 可以用于显示图片、视频，并可以响应很多交互操作。https://fancyapps.com/fancybox/3/
---

FancyBox 是一款基于jQuery的弹出库，用于呈现各种类型的媒体。 可以用于显示图片、视频，并可以响应很多交互操作。https://fancyapps.com/fancybox/3/

### 如何解决Icarus无法放大图片预览问题

在使用博客时候发现icarus没有查看图片放大的预览功能，点击图片时候会跳转到新的图片链接页面中，再次返回博客只能使用浏览器返回或者重新打开博客，查看图片太难了，真是。
Fancybox 提供的图片浏览功能可以很好的解决该问题。本文将综合官方文档和网络上所获得的介绍，分享一个安装及使用 Fancybox 的通用方法。

### 引入 FancyBox 库
FancyBox 库的引入通常释放到</head> 标签前。由于 Hexo / icarus 的特殊文件结构，head 部分通常被单独划分至 `/themes/icarus/layout/common/head.ixc 中`。在不同主题中该路径可能有所更改。

```
<script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.min.js"></script>

<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/fancyapps/fancybox@3.5.7/dist/jquery.fancybox.min.css" />
<script src="https://cdn.jsdelivr.net/gh/fancyapps/fancybox@3.5.7/dist/jquery.fancybox.min.js"></script>
```

### 嵌入使用
至此，Fancybox 最基本的功能已经成功安装，我们可以根据官方文档指引，在文章中加入下面的链接测试效果
```
<a data-fancybox="gallery" href="big_1.jpg"><img src="small_1.jpg"></a>
<a data-fancybox="gallery" href="big_2.jpg"><img src="small_2.jpg"></a>
```

### 调整全局适配
虽然上述的方法可行的，但是icarus写得文章都是用的markdown语法，如果去改历史文章，都每个图片属性上都加上`data-fancybox="gallery"`,就真实太难了，所以还是要修改。

打开hexo主题目录中的`themes/icarus/source/js/main.js`，大概在第六行左右，给a标签增加`data-fancybox="gallery" `属性。

```
vim blog/themes/icarus/source/js/main.js

# 将$(this).wrap('<a class="gallery-item" href="' + $(this).attr('src') + '"></a>');修改为

$(this).wrap('<a class="gallery-item" data-fancybox="gallery" href="' + $(this).attr('src') + '"></a>');
```

### 重启博客，预览效果
执行 hexo g 重新生成博客，FancyBox 应该就可以正常使用了。点击下方图片即可看到效果。

![测试图片](https://static.studytime.xin/article/20200710010415.png)





