## 1.Redis初识

#### 一. redis是什么

```
Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API
```

#### 二.Redis的特性

###### 1.速度快 

```
官方给出 10w OPS/sec  每秒十万次读写[操作]）

原因：
	1.数据存储于内存中（☆☆☆☆☆）
	2.C语言编写，代码高效
	3.线程模型为单线程的（内存的线程 速度很快）
```



###### 2.持久化

```
redis的持久化是说redis在断电的情况下可做到数据不丢失。
redis可以将内存数据通过RDB和AOF的方式写入磁盘，从而达到持久化的效果。
(新版本支持RDB-AOF混合持久化)
```



###### 3.多种数据结构

```
string
hash 
list
set
sorted set
新增延伸数据结构以下（参考：http://www.redis.cn/topics/data-types-intro.html）:
bitmaps:通过特殊的命令，你可以将 String 值当作一系列 bits 处理：可以设置和清除单独的 bits，数出所有设为 1 的 bits 的数量，找到最前的被设为 1 或 0 的 bit，等等。（布隆过滤器）
HyperLogLogs: 这是被用于估计一个 set 中元素数量的概率性的数据结构。（唯一计数）
GEO：地理信息定位（参考：https://www.jb51.net/article/136322.htm）
```



###### 4.支持多种语言

```
PHP
ActionScript
C
C++
C#
Clojure
Common Lisp
Dart
Erlang
Go
Haskell
Haxe
Io
Java
Node.js
Lua
Objective-C
Perl
Pure Data
Python
R
Ruby
Scala
Smalltalk
Tcl
```

###### 5.功能丰富

```
1.发布订阅
2.简单事务
3.lua脚本
4.pipeline
5.持久化
6.高可用
……
```

 

###### 7.主从复制

```
和MySQL主从复制的原因一样，Redis虽然读取写入的速度都特别快，但是也会产生读压力特别大的情况。为了分担读压力，Redis支持主从复制，Redis的主从结构可以采用一主多从或者级联结构，Redis主从复制可以根据是否是全量分为全量同步和增量同步。
```



###### 8.高可用，分布式

```
高可用（High Availability），是当一台服务器停止服务后，对于业务及用户毫无影响。 停止服务的原因可能由于网卡、路由器、机房、CPU负载过高、内存溢出、自然灾害等不可预期的原因导致，在很多时候也称单点问题。

分布式(distributed), 是当业务量、数据量增加时，可以通过任意增加减少服务器数量来解决问题。

redis Sentinel --->高可用
redis Cluster  --->分布式
```



#### Redis的典型使用场景

```
缓存系统
计数器
消息队列系统
排行榜功能
适合社交网络
实时系统
布隆过滤器
```

#### Redis的客户端

我们使用的是PHP客户端， Windows上可以使用Visual VMP来操作redis。

```
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

除此之外我们可以在github下载Windows版本的msi安装包，安装完成后既可以使用。

```
https://github.com/MicrosoftArchive/redis/releases
```

下载安装后，我们先要启动服务器端。

```shell

C:\Redis>redis-server.exe  redis.windows.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.100 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 3692
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[3692] 13 Sep 17:37:48.676 # Server started, Redis version 3.2.100
```

接着我们就可以启动客服端了。

```shell
C:\Redis>redis-cli.exe -h 127.0.0.1 -p 6379
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

$这里的端口是可以通过配置redis.windows.conf文件修改的$



###### redis的密码设置

我们可以通过`config set requirepass 密码`来设置redis的密码 。

通过`config get requirepass`来获取设置的密码

$tips:这里设置密码后如果重启服务，密码就会丢失$





## 2.通用命令和数据结构

#### 通用命令

###### keys  	redis数据库中所有的键

```shell
127.0.0.1:6379> set name 'jim'
OK
127.0.0.1:6379> set age '18'
OK
127.0.0.1:6379> set project 'fullstack'
OK
127.0.0.1:6379> keys *
1) "project"
2) "name"
3) "age"
#keys 还可以进行正则的匹配
127.0.0.1:6379> keys n*  
1) "name"
127.0.0.1:6379>
```

$$在实际线上环境，我们我们使用scan代替keys,keys扫描全库，效率问题$$



###### dbsize	数据库的大小

```
127.0.0.1:6379> dbsize
(integer) 3
127.0.0.1:6379> keys *
1) "name"
2) "age"
3) "project"
127.0.0.1:6379>
```



###### exists key    判断key是否存在

```
127.0.0.1:6379> exists name
(integer) 1
127.0.0.1:6379> exists hobby
(integer) 0
127.0.0.1:6379>
```



###### del key      	删除key

```
127.0.0.1:6379> del age
(integer) 1
127.0.0.1:6379> keys *
1) "name"
2) "project"
127.0.0.1:6379>
```



###### expire key seconds	  key的过期时间

```
127.0.0.1:6379> expire name 5
(integer) 1
127.0.0.1:6379> keys *
1) "project"
127.0.0.1:6379>
```

$$扩展补充：我们可以使用ttl key查看key的剩余存活时间 如果是-2代表key不存在  persist key 去掉key的过期时间 $$

```
127.0.0.1:6379> keys *
1) "project"
127.0.0.1:6379> expire project 60
(integer) 1
127.0.0.1:6379> ttl project
(integer) 51
127.0.0.1:6379> persist project
(integer) 1
127.0.0.1:6379> ttl project
(integer) -1
127.0.0.1:6379>
127.0.0.1:6379> ttl name
(integer) -2
```



###### type key	key的类型

```
127.0.0.1:6379> set face dilireba
OK
127.0.0.1:6379> type face
string
127.0.0.1:6379> sadd outlook beauty amazing
(integer) 2
127.0.0.1:6379> type outlook
set
127.0.0.1:6379> lpush height 170
(integer) 1
127.0.0.1:6379> type height
list
```

$扩展：type返回的数据有string，hash，list，set，zset，none$

#### 数据结构及内部编码

![img](https://images2017.cnblogs.com/blog/1260387/201712/1260387-20171217225104530-830166094.png) 

$$字符串类型的内部编码有3种：$$
  $ int：8个字节的长整型。$ 
  $ embstr：小于等于39个字节的字符串。$
   $raw：大于39个字节的字符串。$

```
127.0.0.1:6379> object encoding face
"embstr"
127.0.0.1:6379> object encoding outlook
"hashtable"
```



哈希类型的内部编码有两种：

- **ziplist**（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个），

　　　　同时所有值都小于hash-max-ziplist-value配置（默认64个字节）时，Redis会使用ziplist作为哈希的内部实现

　　　　ziplist使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比hashtable更加优秀。

- **hashtable**（哈希表）：当哈希类型无法满足ziplist的条件时，Redis会使用hashtable作为哈希的内部实现。

　　　　因为此时ziplist的读写效率会下降，而hashtable的读写时间复杂度为O(1)。



$当有value大于64个字节，内部编码会由ziplist变为hashtable： $

```
127.0.0.1:6379> object encoding user:1
"ziplist"
127.0.0.1:6379> object encoding user:2
"hashtable"
127.0.0.1:6379>
```



列表类型的内部编码有两种：

- **ziplist**（压缩列表）：当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）

　　　　同时所有值都小于hash-max-ziplist-value配置（默认64个字节）时，Redis会使用ziplist作为哈希的内部实现。

- **linkedlist**（链表）：当列表类型无法满足ziplist的条件时，Redis会使用linkedlist作为列表的内部实现。



$在3.2新版本中ziplist，linkedlist已经被quicklist取代$

```
127.0.0.1:6379> object encoding height
"quicklist"
127.0.0.1:6379> lpush key1 1 2 3 4 5
(integer) 5
127.0.0.1:6379> object encoding key1
127.0.0.1:6379> lpush key2 1 2 3 4 5 6 7 8 9 11 12 12 12 12
1 1  1 1 1 1  1 11  11 1 1 1  1 1 1 1 1 1 1  11 1 1  11  1
 1 1 1 1  11 1  1 1 1 1 1  11  11  111  11 1  1 1 11 1  1
(integer) 137
127.0.0.1:6379> object encoding key2
"quicklist"
127.0.0.1:6379>

```





集合类型的内部编码有两种：

- **intset**（整数集合）：当集合中的元素都是整数且元素个数小于set-max-intset-entries配置（默认512个）时，

　　　　Redis会选用intset来作为集合内部实现，从而减少内存的使用。

- **hashtable**（哈希表）：当集合类型无法满足intset的条件时，Redis会使用hashtable作为集合的内部实现。



$当元素个数较少时是intset，当元素个数超过512或者元素不为整数时内部编码为hashtable$

```
127.0.0.1:6379> sadd en 123 1231
(integer) 2
127.0.0.1:6379> object encoding en
"intset"
127.0.0.1:6379> sadd ao a b c 12 13
(integer) 5
127.0.0.1:6379> object encoding ao
"hashtable"
```



有序集合类型的内部编码有两种

- **ziplist**（压缩列表）：当有序集合的元素个数小于zset-max-ziplist-entries配置（默认128个）

　　　　同时每个元素的值小于zset-max-ziplist-value配置（默认64个字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist可以有效减少内存使用。

- **skiplist**（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时zip的读写效率会下降。



$当元素较少时是ziplist，多于128时是skiplist$

```
127.0.0.1:6379> zadd haode 12 a
(integer) 1
127.0.0.1:6379> zadd haode 12 12
(integer) 1
127.0.0.1:6379> zrange haode 0 -1
1) "12"
2) "a"
127.0.0.1:6379> object encoding haode
"ziplist"
```





#### 单线程架构

```
很多并发库是多线程，但是redis是单线程，为什么这么快呢？
因为redis是纯内存，
非阻塞IO，使用的epoll的模型
避免上下文切换和消耗竞争
```

$在有些时间使用的不仅仅是单线程，还有IO多路复用$



## 3.Redis的持久化

Redis 提供了不同级别的持久化方式:

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储.
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.
- 如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 你也可以同时开启两种持久化方式, 在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.
- 最重要的事情是了解RDB和AOF持久化方式的不同,让我们以RDB持久化方式开始:

#### RDB方式

- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.
- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.
- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

$redis默认的就是RDB方式的持久化方式$

如果我们安装的是单独的redis则没有使用持久化策略。

安装visual-nmp默认的是RDB持久化，持久化的数据在安装目录的`cache/redis/dump.rdb`里面。

```
save 900 1    900秒有一次改的就保存
save 300 10	  300秒内有十次改的就保存
save 60 10000 60秒内有10000次改的就保存

# The filename where to dump the DB   rdb存放的文件名
dbfilename dump.rdb

# Redis to log on the standard output.
logfile "redis.log"  配置redis的日志文件
```





#### AOF 方式

- 使用AOF 会让你的Redis更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.
- AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

###### AOF的三种策略

$always$     redis 的每条命令都从缓冲区写到磁盘中

$everysec$   redis每秒一次将命令从缓冲区写到硬盘（默认值）

$no$               redis根据系统决定是否将命令从缓冲区写到硬盘



###### AOF实现的两种方式

###### $bgrewriteaof命令$

要实现bgrewriteaof我们需要配置一下文件

```
auto-aof-rewrite-percentage 100      文件增长率（两次的对比）
auto-aof-rewrite-min-size 64mb       自动重写的最小尺寸
```

aof重写的相关统计项

```
aof_current_size   aof当前的文件大小
aof_base_size      aof上次重写或者启动时的大小
```

aof重写同时满足的条件

```
aof_current_size > auto-aof-rewrite-min-size
(aof_current_size - aof_base_size)/aof_base_size > auto-aof-rewrite-percentage
```



###### $aof重写配置$

```
appendonly no   是否重写
appendfilename "appendonly.aof"  重写的文件名
# appendfsync always
appendfsync everysec     默认的重写策略
# appendfsync no
no-appendfsync-on-rewrite no  重写的时候是否关键aof append
aof-load-truncated yes   重写数据有错误是否忽略

```



###### 测试

```
127.0.0.1:6379> config get appendonly 查看aof配置
1) "appendonly"
2) "no"
127.0.0.1:6379> config set appendonly yes
OK
127.0.0.1:6379> config rewrite
OK
127.0.0.1:6379> config get appendonly
1) "appendonly"
2) "yes"
127.0.0.1:6379>


```

设置完成后，目录生成appendonly.aof文件，文件内容如下。

```
……
SET
$4
name
$3
jim
*2
$6
SELECT
$1
0
*3
$3
set
$1
k
$5
hahah
……
```







## 4.Redis主从复制

redis中我们可以配置主从复制。

简单的说就是一个master节点可以有多个slave节点，一个slave节点只能属于一个master。数据是从master到slave的，就是说数据是单向的。

#### 在redis中配置主从复制。

这里也有两种方式。

```
slaveof命令
配置文件配置
```

#### 通过修改配置文件实现主从复制。

1.修改配置文件。如果是`主节点`的话，我们可以不需要修改配置。

2.修改slave节点

```
1.复制配置文件 并重命名
#修改内容如下
port 6380  端口号（☆☆☆☆☆）
logfile "redis6380.log"  redis log文件
dbfilename "dump80.rdb"  rdb文件
dir "C:\\Program Files\\Redis" 文件路径
slaveof 127.0.0.1 6379  从属于哪个主节点（☆☆☆☆☆）
```

#### 测试

master服务器端启动

```
redis-server.exe redis.windows.slave.conf
```

slave 服务器端启动

```
redis-server.exe redis.windows.slave.conf
```

master 客服端启动

```
redis-cli  #默认6379端口
```

slave客户端启动

```
redis-cli.exe -p 6380
```

master上写数据

```
127.0.0.1:6379> set test thisisatest
OK
127.0.0.1:6379>
```

slave上读数据

```
127.0.0.1:6380> get test
"thisisatest"
127.0.0.1:6380>
```



到此我们一个简单测试就完成了。

如果我们在slave服务器进行写操作呢？

```
127.0.0.1:6380> set test11 testfromslave
(error) READONLY You can't write against a read only slave.
127.0.0.1:6380>

```

$master服务器设置 slave-read-only yes 所以不能进行写操作 $

$tips 我们可以在master和slave上执行info     replication查看主从情况$

**master**

```
127.0.0.1:6379> info replication
# Replication
role:master   #角色是master
connected_slaves:1  #连接的slave 1台
slave0:ip=127.0.0.1,port=6380,state=online,offset=751,lag=1
master_repl_offset:765
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:764
127.0.0.1:6379>
```

**slave**

```
127.0.0.1:6380> info replication
# Replication
role:slave   #角色slave
master_host:127.0.0.1  #master的host
master_port:6379       #master的port
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
slave_repl_offset:933
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6380>
```

思考：如果master节点挂了，我们应该如何处理呢？

下面请看Redis Sentinel。

## 5.Redis Sentinel 

#### 作用：  

用于管理多个Redis服务实现HA  1.监控主服务器Master；  2.通知；  3.自动故障转移； 

####  协议：  

流言协议、投票协议  

#### 工作过程 

 1）服务器自身初始化，运行于redis-server中专用于sentinel功能的代码； 

 2）初始化sentinel状态，根据给定的配置文件，初始化监控的master服务器列表； 

 3）创建连向master的连接； 



#### 配置主从

这里我们配置了一主两从的主从配置配置和前面的第四章节的主从配置一样，这里我们就不再演示。

#### 配置sentinel

三个配置我们分别命名为sentinel12580.conf,sentinel12581.conf,sentinel12582.conf,配置项如下：

```
port 12580
sentinel monitor master 127.0.0.1 6379 2 
sentinel down-after-milliseconds master 5000
sentinel failover-timeout master 15000
```

#### 配置释义

```
port 12580     #sentinel 端口
sentinel monitor master 127.0.0.1 6379 2 去监视一个名为mymaster的主redis实例，这个主实例的IP地址为本机地址127.0.0.1，端口号为6379，而将这个主实例判断为失效至少需要2个 Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行  
sentinel down-after-milliseconds master 5000
指定了Sentinel认为Redis实例已经失效所需的毫秒数。当 实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行  
sentinel failover-timeout master 15000

指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长  
```



#### 启动主从服务器

先启动三个主从服务器，启动方式通第四章。启动完成后，在master服务器可以看到相关主从信息。

```
[8312] 16 Sep 10:40:52.867 # Server started, Redis version 3.2.100
[8312] 16 Sep 10:40:52.870 * DB loaded from disk: 0.001 seconds
[8312] 16 Sep 10:40:52.872 * The server is now ready to accept connections on port 6379
[8312] 16 Sep 10:40:53.188 * Slave 127.0.0.1:6381 asks for synchronization
[8312] 16 Sep 10:40:53.190 * Partial resynchronization not accepted: Runid mismatch (Client asked for runid '36a72ab0752754f28ba5db09d0
231dada9f9dca5', my runid is 'ccd2050bf885d9cf90b3942b4090235d68e02443')
[8312] 16 Sep 10:40:53.193 * Starting BGSAVE for SYNC with target: disk
[8312] 16 Sep 10:40:53.234 * Background saving started by pid 4080
[8312] 16 Sep 10:40:53.256 * Slave 127.0.0.1:6380 asks for synchronization
[8312] 16 Sep 10:40:53.259 * Partial resynchronization not accepted: Runid mismatch (Client asked for runid '36a72ab0752754f28ba5db09d0
231dada9f9dca5', my runid is 'ccd2050bf885d9cf90b3942b4090235d68e02443')
[8312] 16 Sep 10:40:53.263 * Waiting for end of BGSAVE for SYNC
[8312] 16 Sep 10:40:53.375 # fork operation complete
[8312] 16 Sep 10:40:53.378 * Background saving terminated with success
[8312] 16 Sep 10:40:53.388 * Synchronization with slave 127.0.0.1:6381 succeeded
[8312] 16 Sep 10:40:53.393 * Synchronization with slave 127.0.0.1:6380 succeeded
```



#### 启动Redis Sentinel

启动方式是依次使用redis-cli.exe sentinel12580.conf,redis-cli.exe sentinel12581.conf,redis-cli.exe sentinel12582.conf

```
C:\Program Files\Redis>redis-server.exe sentinel12580.conf
[1040] 14 Sep 10:44:54.281 #
*** FATAL CONFIG FILE ERROR ***

[1040] 16 Sep 10:44:54.283 # Reading the configuration file, at line 2

[1040] 16 Sep 10:44:54.283 # >>> 'sentinel myid 367e595d569bb8d30a0c2c96c2d5f5ff4ffbde7d'

[1040] 16 Sep 10:44:54.283 # sentinel directive while not in sentinel mode
```

$启动完成报了以上的错，是因为我们没有使用sentinel模式，只需要配置文件后面加上 --sentinel即可$

```
C:\Program Files\Redis>redis-server.exe sentinel12580.conf --sentinel
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.100 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 12580
 |    `-._   `._    /     _.-'    |     PID: 8320
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[8320] 16 Sep 10:46:59.725 # Sentinel ID is 367e595d569bb8d30a0c2c96c2d5f5ff4ffbde7d
[8320] 16 Sep 10:46:59.726 # +monitor master master 127.0.0.1 6379 quorum 2
[8320] 16 Sep 10:47:04.768 # +sdown sentinel 4867b5b8d310a25fd8d05e5fef2377deddefbb08 127.0.0.1 12582 @ master 127.0.0.1 6379
[8320] 16 Sep 10:47:04.769 # +sdown sentinel 3bea1b075787ab7b7bbe55d728f7ad68f0cffba5 127.0.0.1 12581 @ master 127.0.0.1 6379
```

出现以上内容，这说明我们的Redis Sentinel启动成功

启动成功后，我们可以看到redis sentinel在监控master节点了

```
[8320] 16 Sep 10:46:59.726 # +monitor master master 127.0.0.1 6379 quorum 2
```

#### 测试

###### 首先退出master节点

```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=52880,lag=1
slave1:ip=127.0.0.1,port=6380,state=online,offset=52880,lag=0
master_repl_offset:52880
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:52879
127.0.0.1:6379> shutdown
not connected> exit

C:\Program Files\Redis>
```

###### 主节点已经关闭

```
[8312] 16 Sep 10:40:53.263 * Waiting for end of BGSAVE for SYNC
[8312] 16 Sep 10:40:53.375 # fork operation complete
[8312] 16 Sep 10:40:53.378 * Background saving terminated with success
[8312] 16 Sep 10:40:53.388 * Synchronization with slave 127.0.0.1:6381 succeeded
[8312] 16 Sep 10:40:53.393 * Synchronization with slave 127.0.0.1:6380 succeeded
[8312] 16 Sep 10:52:43.850 # User requested shutdown...
[8312] 16 Sep 10:52:43.850 * Saving the final RDB snapshot before exiting.
[8312] 16 Sep 10:52:43.855 * DB saved on disk
[8312] 14 Sep 10:52:43.855 # Redis is now ready to exit, bye bye...
```

###### 查看Redis Sentinel的信息提示

```
Sep 10:52:58.611 # +switch-master master 127.0.0.1 6379 127.0.0.1 6380  已经切换成功
Sep 10:52:58.618 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6380
Sep 10:52:58.629 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ master 127.0.0.1 6380
Sep 10:53:03.633 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ master 127.0.0.1 6380
Sep 10:53:03.633 # +sdown slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6380
Sep 10:53:31.594 # -sdown slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6380
```



当我们再次启动主节点的时候会自动切换回6379主节点

```
Sep 10:59:17.920 # +switch-master master 127.0.0.1 6381 127.0.0.1 6379
Sep 10:59:17.925 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ master 127.0.0.1 6379
Sep 10:59:17.927 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6379
Sep 10:59:22.985 # +sdown slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6379
Sep 10:59:22.986 # +sdown slave 127.0.0.1:6380 127.0.0.1 6380 @ master 127.0.0.1 6379
Sep 10:59:38.185 # -sdown slave 127.0.0.1:6381 127.0.0.1 6381 @ master 127.0.0.1 6379
Sep 10:59:38.186 # -sdown slave 127.0.0.1:6380 127.0.0.1 6380 @ master 127.0.0.1 6379
```





到此Redis Sentinel基本使用应经讲解完成。

 

## 6.Redis Cluster

