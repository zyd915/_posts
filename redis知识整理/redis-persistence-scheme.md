---
title: redis的持久化方案扩展讲解
permalink: redis-persistence-scheme
date: 2020-08-10 01:01:01
updated: 2020-08-10 01:01:47
tags:
    - redis
categories: redis
toc: true
excerpt: redis是一个内存数据库，所有数据都存储在内存当中，而内存中的数据极易丢失，所以redis的数据持久化就显得尤为重要，在redis当中，提供了两种数据持久化的方式，分别为RDB以及AOF，且redis默认开启的数据持久化方式为RDB方式。
---


redis是一个内存数据库，所有数据都存储在内存当中，而内存中的数据极易丢失，所以redis的数据持久化就显得尤为重要，在redis当中，提供了两种数据持久化的方式，分别为RDB以及AOF，且redis默认开启的数据持久化方式为RDB方式。

## RDB持久化方案介绍之RDB方案介绍
Redis会定期保存数据快照至一个rbd文件中，并在启动时自动加载rdb文件，恢复之前保存的数据。可以在配置文件中配置Redis进行快照保存的时机。

### RDS 配置命令 save [seconds] [changes]
在[seconds]秒内如果发生了[changes]次数据修改，则进行一次RDB快照保存。

`save 60 100`，Redis每60秒检查一次数据变更情况，如果发生了100次或以上的数据变更，则进行RDB快照保存。
可以配置多条save指令，让Redis执行多级的快照保存策略。Redis默认开启RDB快照。也可以通过SAVE或者BGSAVE命令手动触发RDB快照保存。

### 手动触发RDB
SAVE 和 BGSAVE 两个命令都会调用 rdbSave 函数，但它们调用的方式各有不同：
1.SAVE 直接调用 rdbSave ，阻塞 Redis 主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。
2.BGSAVE 则 fork 出一个子进程，子进程负责调用 rdbSave ，并在保存完成之后向主进程发送信号，通知保存已完成。 Redis 服务器在BGSAVE 执行期间仍然可以继续处理客户端的请求。

### RDB方案优点
1.对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。
2.每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。
3.使用RDB文件进行数据恢复比使用AOF要快很多

### RDB方案缺点
1.快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据。
2.如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间，影响Redis对外提供服务的能力。


### RDB方案配置
```
vim /usr/local/redis/etc/redis.conf

save 900 1
save 300 10
save 60 10000
save 5 1
```
重新启动redis服务，每次生成新的dump.rdb都会覆盖掉之前的老的快照。

## AOF持久化方案介绍之AOF方案介绍
采用AOF持久方式时，Redis会把每一个写请求都记录在一个日志文件里。在Redis重启时，会把AOF文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。AOF默认是关闭的，如要开启，进行`appendonly yes`配置开启。

### AOF提三种fsync配置
AOF提供了三种fsync配置，always、everysec、no，通过配置项[appendfsync]指定。
1.appendfsync no：不进行fsync，将flush文件的时机交给OS（操作系统）决定，速度最快
2.appendfsync always：每写入一条日志就进行一次fsync操作，数据安全性最高，但速度最慢
3.appendfsync everysec：折中的做法，交由后台线程每秒fsync一次


### AOF rewrite
随着AOF不断地记录写操作日志，因为所有的操作都会记录，所以必定会出现一些无用的日志。大量无用的日志会让AOF文件过大，也会让数据恢复的时间过长。不过Redis提供了AOF rewrite功能，可以重写AOF文件，只保留能够把数据恢复到最新状态的最小写操作集。
AOF rewrite可以通过BGREWRITEAOF命令触发，也可以配置Redis定期自动进行：
`auto-aof-rewrite-percentage 100auto-aof-rewrite-min-size 64mb`
上面两行配置的含义是，Redis在每次AOF rewrite时，会记录完成rewrite后的AOF日志大小，当AOF日志大小在该基础上增长了100%后，自动进行AOF rewrite。同时如果增长的大小没有达到64mb，则不会进行rewrite。


### AOF优点
1. 最安全，在启用appendfsync always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据
2. AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复。
3. AOF文件易读，可修改，在进行了某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。

### AOF的缺点：
1. AOF文件通常比RDB文件更大
2. 性能消耗比RDB高
3. 数据恢复速度比RDB慢

### 持久化方案选择
Redis的数据持久化工作本身就会带来延迟，需要根据数据的安全级别和性能要求制定合理的持久化策略。

#### AOF + fsync always
此方案设置虽然能够绝对确保数据安全，但每个操作都会触发一次fsync，会对Redis的性能有比较明显的影响

#### AOF + fsync every second
相对比较好的折中方案，每秒fsync一次

#### AOF + fsync never
此方案会提供AOF持久化方案下的最优性能

#### RDB持久化方案
使用RDB持久化通常会提供比使用AOF更高的性能，但需要注意RDB的策略配置。

每一次RDB快照和AOF Rewrite都需要Redis主进程进行fork操作。fork操作本身可能会产生较高的耗时，与CPU和Redis占用的内存大小有关。根据具体的情况合理配置RDB快照和AOF Rewrite时机，避免过于频繁的fork带来的延迟。

Redis在fork子进程时需要将内存分页表拷贝至子进程，以占用了24GB内存的Redis实例为例，共需要拷贝24GB / 4kB * 8 = 48MB的数据。在使用单Xeon 2.27Ghz的物理机上，这一fork操作耗时216ms。


### AOF方案配置
在redis中，aof的持久化机制默认是关闭的。AOF持久化，默认是关闭的，默认是打开RDB持久化。

#### 开启命令 appendonly yes
可以打开AOF持久化机制，在生产环境里面，一般来说AOF都是要打开的，除非你说随便丢个几分钟的数据也无所谓。
打开AOF持久化机制之后，redis每次接收到一条写命令，就会写入日志文件中，当然是先写入os cache的，然后每隔一定时间再fsync一下。
而且即使AOF和RDB都开启了，redis重启的时候，也是优先通过AOF进行数据恢复的，因为aof数据比较完整。
可以配置AOF的fsync策略，有三种策略可以选择，一种是每次写入一条数据就执行一次fsync; 一种是每隔一秒执行一次fsync; 一种是不主动执行fsync
always: 每次写入一条数据，立即将这个数据对应的写日志fsync到磁盘上去，性能非常非常差，吞吐量很低; 确保说redis里的数据一条都不丢，那就只能这样了
在redis当中默认的AOF持久化机制都是关闭的


### 配置redis的AOF持久化机制方式
cd /kkb/install/redis-3.2.8
```
vim /usr/local/redis/etc/redis.conf
appendonly yes
# appendfsync always
appendfsync everysec
# appendfsync no```



