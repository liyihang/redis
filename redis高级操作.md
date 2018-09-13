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





## 4.Redis复制的原理和优化





## 5.Redis Sentinel 



 

## 6.Redis Cluster

