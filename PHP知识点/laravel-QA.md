---
title: Larvel 遇到的问题以及解决方案
permalink: laravel-QA
date: 2020-01-08 00:23:49
updated: 2020-01-08 00:23:50
tags: 
    - php
    - laravel
categories: php
toc: true
thumbnail: https://static.studytime.xin/article/20200618130414.png
excerpt: 整理php框架laravel使用过程中遇到的一些问题以及解决方案
---

### `Job has been attempted too many times or run too long ` 和 `大量MaxAttemptsExceededException`问题

此种问题的产生主要是因为`retry_after`和`timeout`参数设置过短，引起的任务超时影响。


#### retry_after 参数说明
在`config/queue.php`配置文件中，laravel为每个队列连接定义一个 `retry_after` 选项。此选项指定在重试正在处理的任务之前，队列连接应等待多少秒。

#### timeout 参数说明
`queue:work` Artisan 命令暴露一个 `--timeout` 选项。`--timeout` 选项指定在杀死正在处理作业的子队列 `worker` 之前，Laravel 队列主进程将等待多长时间。有时，由于各种原因，子队列进程可能会被 “冻结”。 `--timeout` 选项用来删除超过指定时间限制的冻结进程:

#### 解决方案
```
# 修改config/queue.php配置文件中retry_after参数值

'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => (60 * 30), // 30 minutes
        'block_for' => null,
],

# queue:work 制定timeout时间长度为
php artisan queue:work --timeout=1755
```

#### 特殊说明
`retry_after` 值设置为你的任务完成处理所需的最大秒数。
`retry_after` 配置选项和 `--timeout CLI `选项是不同的，但它们共同确保不会丢失任务，并且任务只被成功处理一次。
`--timeout` 值应该总是比 `retry_after` 配置值至少短几秒。这将确保处理给定任务的 `worker` 总是在重试作业之前被杀死。如果你的 `--timeout` 选项比你的 `retry_after` 配置值长，你的任务可能会被处理两次。


#### 参考文件
- [https://learnku.com/docs/laravel/7.x/queues/7491#a8780c](https://learnku.com/docs/laravel/7.x/queues/7491#a8780c)
- [https://stackoverflow.com/questions/53075318/job-has-been-attempted-too-many-times-or-run-too-long](https://stackoverflow.com/questions/53075318/job-has-been-attempted-too-many-times-or-run-too-long)
