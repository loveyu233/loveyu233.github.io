---
title: "redis"
date: 2020-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- redis
- nosql
---

# Redis数据类型

## 1. string字符串

### 分布式锁

```shell
setnx key value
del key
```



### 自增自减

```shell
incrby key value
decrby key value
```



## 2. list双端队列

### 消息订阅

>lpush key value
>
>rpop key value

## 3. hash

### 购物车

```shell
// 新增商品
hset shopcar:uid shopid number
// 商品个数添加
hincrby shopcar:uid shopid 1
// 查看用户全部购物车商品总数
hlen shopcar:uid
// 全部选择
hgetall shopcar:uid
```

```shell
127.0.0.1:6379> hset shopcar:01 001 1
(integer) 1
127.0.0.1:6379> hset shopcar:01 002 1
(integer) 1
127.0.0.1:6379> hlen shopcar:01
(integer) 2
127.0.0.1:6379> hgetall shopcar:01
1) "001"
2) "1"
3) "002"
4) "1"
127.0.0.1:6379> hincrby shopcar:01 001 1
(integer) 2
127.0.0.1:6379> hincrby shopcar:01 001 -1
(integer) 1
127.0.0.1:6379> hgetall shopcar:01
1) "001"
2) "1"
3) "002"
4) "1"
```



## 4. set无序无重复集合

### 抽奖

```shell
// 参与抽奖
sadd key userid
// 获取参与人数
scard key
// 开始抽奖,number为此次中奖人数
srandmember key number // 抽完奖后还可以继续参与抽奖
spop key number				// 抽完奖就弹出不可以参加下一轮抽奖
```

```shell
127.0.0.1:6379> sadd s 001
(integer) 1
127.0.0.1:6379> sadd s 002
(integer) 1
127.0.0.1:6379> sadd s 003
(integer) 1
127.0.0.1:6379> sadd s 004
(integer) 1
127.0.0.1:6379> scard s
(integer) 4
127.0.0.1:6379> srandmember s 2
1) "003"
2) "001"
127.0.0.1:6379> scard s
(integer) 4
127.0.0.1:6379> spop s 2
1) "003"
2) "004"
127.0.0.1:6379> scard s
(integer) 2
```



### 点赞

```shell
// 新增点赞
sadd msgid userid
// 取消点赞
srem msgid userid
// 获取所有点赞的用户id
smembers msgid
// 点赞统计
scard msgid
// 判断用户是否点过赞
sismember msgid userid
```

```shell
127.0.0.1:6379> sadd msg:1 001
(integer) 1
127.0.0.1:6379> sadd msg:1 002
(integer) 1
127.0.0.1:6379> sadd msg:1 003
(integer) 1
127.0.0.1:6379> smembers msg:1
1) "003"
2) "002"
3) "001"
127.0.0.1:6379> smembers msg:1
1) "003"
2) "002"
3) "001"
127.0.0.1:6379> scard msg:1
(integer) 3
127.0.0.1:6379> srem msg:1 002
(integer) 1
127.0.0.1:6379> sismember msg:1 002
(integer) 0
127.0.0.1:6379> sismember msg:1 001
(integer) 1
```

### 好友

```shell
// 添加好友
sadd uid huid
// A和B共同好友,交集
sinter uidA uidB
// A没有B有的,差集
sdiff uidA uidB
// A和B都有的,并集
sunion uidA uidB
```

```shell
127.0.0.1:6379> sadd uidA 001 002 003 004
(integer) 4
127.0.0.1:6379> sadd uidB 002 003 004 005
(integer) 4
127.0.0.1:6379> sinter uidA uidB
1) "003"
2) "002"
3) "004"
127.0.0.1:6379> sdiff uidA uidB
1) "001"
127.0.0.1:6379> sdiff uidB uidA
1) "005"
127.0.0.1:6379> sunion uidA uidB
1) "001"
2) "005"
3) "002"
4) "004"
5) "003"
```



## 5. zset有序无重复集合

### 排行榜相关

>例如: 点赞排行榜,商品热销排行榜,主播打赏排行榜,视频播放量排行榜

```shell
// 添加key
zadd key score value
// 增加key的value值
zincrby key value number
// 从大到小排行前三
zrange key 0 2 withscores
// 查询从scoreA到scoreB之间的,不包括就加上(
zrangebyscore min max withscores limit offset
// 统计从scoreA到scoreB之间的个数
zcount key min max
```

```shell
127.0.0.1:6379> zadd key 10 a 20 b 30 c 40 d 50 e
(integer) 5
127.0.0.1:6379> zrange key 0 -1 withscores
 1) "a"
 2) "10"
 3) "b"
 4) "20"
 5) "c"
 6) "30"
 7) "d"
 8) "40"
 9) "e"
10) "50"
127.0.0.1:6379> zincrby key 100 a
"110"
127.0.0.1:6379> zrange key 0 2 withscores
1) "b"
2) "20"
3) "c"
4) "30"
5) "d"
6) "40"
127.0.0.1:6379> zrevrange key 0 2 withscores
1) "a"
2) "110"
3) "e"
4) "50"
5) "d"
6) "40"
127.0.0.1:6379> zrangebyscore key 30 100 withscores
1) "c"
2) "30"
3) "d"
4) "40"
5) "e"
6) "50"
```



## 6. geo地理空间

### 附近商家之类的功能

```shell
// 添加经纬度
geoadd key 经度 维度 名称
// 查找经纬度
geopos key 名称
// 返回为编码后的
geohash key 名称
// 查找半径内的, withdist:返回结果中带上距离多少 withcoord:返回结果带上经纬度 withhash:返回结果带上编码后的 count:返回的个数
georadius key 当前位置的经度 当前位置的维度 距离 距离的单位 withdist withcoord withhash count 个数 desc
// 不用经纬度查询而是用已有的元素查询
georadiusbymember key 名称 距离 距离单位 withdist withcoord withhash count 个数 desc

```



## 7. hyperLoglog基数统计

>基数:去重后的数据,用来做大数据的统计,例如网站访问量,搜索统计等..

```shell
// 添加
pfadd key element...
// 统计
pfcount key
// 合并
pfcount newkey key1 key2
```



## 8. bitmap位图

>由0和1表现的二进制的bit数组

>not/and/or/xor : 非/与/或/异或(同为0异为1)
>
>A: 1 1 1 0 0 1 0 0
>
>B: 1 1 0 0 0 0 1 0
>
>and: 1 1 0 0 0 0 0
>
>or:1 1 1 0 0 1 1
>
>xor: 0 0 1 0 0 1 1 0
>
>notA: 0 0 0 1 1 0 1 1

### 签到

```shell
// 签到,key为用户id:时间年月 日 1
setbit userid:time day 1
// 统计一个月签到次数
bitcount userid:time
```



## 9. bitfield位域

## 10. stream流消息队列



# 命令

## String

|             命令             |                             文档                             |
| :--------------------------: | :----------------------------------------------------------: |
|        set key value         |                      添加一个key value                       |
|           get key            |                      获取指定key的value                      |
|       append key value       |     在原有值后面追加值,不存在则创建,返回值为追加后的长度     |
|           incr key           |                        自增相当于+=1                         |
|       incrby key value       | 增加指定的值,可以是正数也可以是负数,正数则是增加负数则是减少 |
|           decr key           |                        自减相当于-=1                         |
|       decrby key value       | 减去指定的值,可以是正数也可以是负数,正数则是减去负数则是增加 |
|          strlen key          |                    获取指定key的value长度                    |
|    getrange key start end    |            获取key从start到end长度的值,是左闭右闭            |
| mset key1 value1 key2 value2 |                          添加多个kv                          |
|        mget key1 key2        |                      获取多个key的value                      |
|       setnx key value        |        如果这个key不存在就创建返回1,存在则不创建返回0        |
|   setex key seconds value    |                 添加key value 并设置过期时间                 |



## List

>列表,相当于一个管道两头都可以进都可以出

|                  命令                   |                             文档                             |
| :-------------------------------------: | :----------------------------------------------------------: |
|           lpush key value...            |                         从左边添加值                         |
|           rpush key value...            |                         从右边添加值                         |
|          lrange key start end           |               获取key从start到end的值,左闭右闭               |
|             lpop key count              |            从左弹出count个值,弹出后就相当于删除了            |
|             rpop key count              |                      从右弹出count个值                       |
|         lrem key count element          | 删除key里面count个数的element,count大于0从表头开始,小于0从表尾开始,等于0则删除全部value等于element的值 |
|            lindex key index             |                获取key里面索引为index的value                 |
|         lset key index newvalue         | 修改key里面索引为index的value为newvalue,没有index索引则报错  |
|           ltrim key start end           |             删除key索引从start到end之外的全部值              |
| linsert key before/after value newvalue |               在key的vlaue的前/后添加newvalue                |



## Set

>无序集合

|              命令               |                             文档                             |
| :-----------------------------: | :----------------------------------------------------------: |
|        sadd key value...        |                  添加集合key里面多个value值                  |
|            scard key            |                   查询key集合里面元素个数                    |
|  sdiff/sinter/sunion key1 key2  |              查询key1与key2的差集/交集/并集元素              |
|          smembers key           |                  获取key集合里面全部的元素                   |
|       sismember key value       |         判断key集合里面有没有value,有返回1没有返回0          |
|      smove key1 key2 value      |           把key1集合里面的value移动到key2的集合中            |
|      srandmember key count      |                 返回key里面count个数的随机值                 |
|        srem key value...        |             删除key里面指定value值,可以删除多个              |
| sscan key cur match v1 count v2 | 返回模糊查找key集合里面符合v1条件的v2个数的值的集合,cur为游标一般为0即可 |



## ZSET

>有序集合,排序是按照score进行从小到大进行排序

|              命令               |                             文档                             |
| :-----------------------------: | :----------------------------------------------------------: |
|     zadd key score value...     | 向key集合里面添加分数为score值为value的数据,如果有value这个值则会更新value这个值的score |
|            zcard key            |                    返回key集合的元素个数                     |
|       zcount key min max        |           返回key集合里面分数从min到max的元素个数            |
|    zincrby key score member     |          把key的集合里面member元素的分数加等上score          |
| zrange key start end withscores | 返回key集合里面从start到end的元素,携带withscores参数则会返回时带上对应的分数,从小到大 |
|     zrevrange key start end     | 返回key集合里面从start到end的元素,携带withscores参数则会返回时带上对应的分数,从大到小 |
|  zrevrangebyscore key max min   |              返回key集合的分数从max到min的元素               |
|      zlexcount key min max      | 返回key集合里面从min到max的个数,例如:zlexcount key - + , -表示最小+表示最大,可以用 [ 符号来指定元素,例如: zlexcount key [a [f 查询从a到f之间的元素包含a和f |
|        zrank key member         |               返回key集合里面member元素的索引                |
|         zrem key member         |                       删除key集合里面                        |
|  zremrangebyscore key min max   |        删除key集合里面分数大于等于min小于等于max的值         |
|   zremrangebyrank key min max   |              删除key集合里面索引从min到max的值               |
|       zrevrank key member       |       返回key集合里面member元素的排名,从大到小,从0开始       |
| zmpop count key min/max count 1 |       第一个count为有多少个key,第二个count为弹出的个数       |



# 事务

>redis的事务没有回滚功能,也不能保证原子性
>
>multi: 开启事务
>
>exec: 提交事务
>
>开启事务到提交事务过程中如果有语法上的错误的话这个事务会直接报错无法执行
>
>如果没有语法上的错误的话这个事务提交后就会逐条执行,正确的就执行错误的就报错,不会回滚不能保证原子性



## watch

>命令: watch key...
>
>监控key,当这个key被监控后开启事务并修改这个key,在没有提交前别的客户端对这个key做了修改,然后提交这个事务就会失败,这个事务中的所有命令都会执行失败,返回nil
>
>事务提交后监控会自己取消

## unwathch

>取消监控



# 管道

>一次执行多条命令, 没有原子性
>
>先把命令保存到一个文件中
>
>然后执行
>
>文件名 | redis-cli -- pipe



# BigKey问题

## 禁用keys * flushall flushdb命令

```shell
== SECURITY ==
rename command keys ""
rename command flushall ""
rename command flushdb ""
```



## 禁用了keys *后怎么获取指定前缀的key

>scan 游标 匹配表达式 返回个数
>
>分别有sscan hscan zscan
>
>第一次游标为0,查询后会返回一个游标,如果返回的游标为0则表示已经查询完了,不是0则下次查询的游标就为上次返回的游标值
>
>scan的遍历顺序不是从0到最后,而是采用高位进位加法来遍历,使用这样特殊的方式进行遍历是考虑到字典的扩容和缩容时避免槽位的遍历重复和遗漏



```shell
127.0.0.1:6379> scan 0 match k* count 3
655360
k788708
k952061
k621947
127.0.0.1:6379> scan 655360 match k* count 3
327680
k822248
k323028
k857178
```

## 多大的key算大key

1. string类型的10kb内
2. 其他类型元素个数不能超过5000



## 危害

1. 内存分布不均匀,集群建议困难
2. 超时删除
3. 网络流量阻塞



## 怎么产生的

1. 暴增式的,例如突然爆火的明星网红粉丝瞬间变多
2. 汇总式的,例如某个报表,每天积累上去的



## 如何发现

1. ```shell
   redis-cli --bigkeys
   == --bigkeys只会返回每个类型的最大的key并给出每种数据类型的个数和平均大小
   
   ~ > redis-cli --bigkeys
   
   # Scanning the entire keyspace to find biggest keys as well as
   # average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
   # per 100 SCAN commands (not usually needed).
   
   [00.00%] Biggest string found so far '"k788708"' with 7 bytes
   [100.00%] Sampled 1000000 keys so far
   
   -------- summary -------
   
   Sampled 1000000 keys in the keyspace!
   Total key length in bytes is 6888889 (avg len 6.89)
   
   Biggest string found '"k788708"' has 7 bytes
   
   0 lists with 0 items (00.00% of keys, avg size 0.00)
   0 hashs with 0 fields (00.00% of keys, avg size 0.00)
   1000000 strings with 6888889 bytes (100.00% of keys, avg size 6.89)
   0 streams with 0 entries (00.00% of keys, avg size 0.00)
   0 sets with 0 members (00.00% of keys, avg size 0.00)
   0 zsets with 0 members (00.00% of keys, avg size 0.00)
   ```

   

2. ```shell
   memory usage key
   == 返回这个key占的大小
   127.0.0.1:6379> MEMORY USAGE k1000
   (integer) 72
   ```



## 如何删除

> 非字符串类型的不要使用del直接删除,要使用scan渐进式的删除,注意大key的过期自动删除

1. string
   1. 一般 ``del`` , 特别大就用 ``unlink``
2. list
   1. 先使用 ``llen`` 获取list的大小
   2. 然后使用 ``ltrim`` 渐进式删除
   3. 最后使用 ``del`` 删除这个key
   4. ``ltrim key start end`` 只保留从start到end的其余全部删除
3. hash
   1. 使用 ``hscan`` 每次获取少量的field-value
   2. 然后使用 ``hdel`` 删除每个key
   3. 最后使用 ``del`` 删除这个key
4. set
   1. 使用 ``sscan`` 扫描部分元素
   2. 再使用 ``srem`` 逐步删除
   3. 最后使用 ``del`` 删除这个key
   4. ``srem key member....`` 
5. zset
   1. 使用 ``zscan`` 扫描部分元素
   2. 再使用 ``zremrangebyrank`` 逐步删除
   3. 最后使用 ``del`` 删除这个key
   4. ``zremrangebyrank key start end`` 



## bigkey生产调优

1. ``del`` 阻塞删除
2. ``unlink`` 非阻塞删除

在conf配置文件中的 ``LAZY FREEING`` 中

1. lazyfree- lazv- eviction no

   1. >针对redis内存使用达到maxmeory，并设置有淘汰策略时；在被动淘汰键时，是否采用lazy free机制； 因为此场景开启lazy free, 可能使用淘汰键的内存释放不及时，导致redis内存超用，超过maxmemory的限制。

2. Lazyfree- lazy- expire no

   1. >针对设置有TTL的键，达到过期后，被redis清理删除时是否采用lazy free机制；

3. lazyfree. lazy- server- del .no

   1. >针对有些指令在处理已存在的键时，会带有一个隐式的DEL键的操作。如rename命令，当目标键已存在, redis会先删除目标键，如果这些目标键是一个big key, 那就会引入阻塞删除的性能问题。 此参数设置就是解决这类问题，建议可开启。

4. replica- lazy- flush no

   1. >针对slave进行全量数据同步，slave在加载master的RDB文件前，会运行flushall来清理自己的数据场景， 参数设置决定是否采用异常flush机制。
      >
      >如果内存变动不大，建议可开启。可减少全量同步耗时，从而减少主库因输出缓冲区爆涨引起的内存使用增长。

5. lazyf ree- lazy- user- del no
   1. 修改为yes



# 双写一致性

## 双检加锁策略

>缓存失效后重构

- 多个线程同时去查询数据库的这条数据，就在第一个查询数据的请求上使用一个互斥锁来锁住他。
- 其他线程获取不到锁就一直等待，等第一个线程查询到了数据，然后做了缓存
- 后面的线程进来发现已经有了缓存，就直接走缓存



## 更新策略

>对数据库数据进行修改后缓存要更新的操作

### 先更新数据库再更新缓存

1. #### 更新完数据库后缓存更新失败了

   1. >例如数据库库存为 ``100`` ,现在下单
      >
      >1. 修改数据库数据,从 ``100`` 修改成 ``99`` 
      >2. 数据库修改后开始修改缓存,但是缓存修改失败
      >3. 现在的缓存依旧是 ``100`` 

2. #### 多线程情况下造成了缓存覆盖

   1. >例如数据库库存为 ``100`` ,现在A和B下单,在多线程的情况下
      >
      >1. A线程修改数据库库存为 ``99`` ,B线程修改数据库库存为 ``98`` 
      >2. B线程抢先一步先修改了缓存为 ``98`` 
      >3. A线程然后又修改了缓存为 ``99`` 
      >4. 现在缓存变为 ``99``,但是实际应该为 ``98`` 才对

### 先更新缓存再更新数据库

1. #### 不推荐

   1. 原因为: 一般把数据库的数据库作为 ``底单数据`` 也就是最终数据

2. #### 多线程情况下造成了数据库的数据库覆盖

   1. >例如数据库库存为 ``100`` ,现在A和B下单,在多线程的情况下
      >
      >1. A线程修改缓存为 ``99`` ,B线程修改缓存为 ``98`` 
      >2. B线程抢先一步把数据库的数据修改为了 ``98``
      >3. 然后A线程又把数据库的数据修改为了 ``99`` 
      >4. 现在缓存数据为 ``98`` ,但是数据库的数据为 ``99`` 

### 先删除缓存再更新数据库

>1. A删除缓存,然后更新数据库,但是因为某些原因例如网络问题,在数据库更新时B来查询数据
>2. B发现缓存里面没有数据,然后去数据库查询数据,但是这时A还没有给新的数据更新进去,于是B读取到了旧数据,然后把旧数据写入了缓存
>3. 然后A线程更新数据库成功了,但是这时缓存有数据了,这个数据和数据库数据不一致了

#### 解决方案

##### 延时双删

>#### A先删除缓存然后更新数据库在数据库更新完毕后休眠一段时间再次把缓存数据删除

##### 问题

>#### A从更新数据到再次删除缓存的这段时间要大于B读取数据库并重建缓存的时间
>
>``那么这个时间要设置为多少?`` 
>
>1. 自行评估耗时时间,然后设置为这个时间
>2. 新起一个后台监控程序, 使用WatchDog监控



### 先更新数据库再删除缓存

#### 问题

>1. A先修改数据库数据,在A修改的过程中,B来查询
>2. B查询到了旧的数据
>3. A修改完成,删除缓存数据
>
>``问题就是在修改数据库数据到删除缓存过程中其他查询的线程会读取到旧数据``

## 工程案列

### mysql主从复制原理

1. 当master主服务器上的数据发生改变时，将其改变写入二进制事件日志文件中
2. slave 从服务器会在一定时间间隔内对master 主服务器上的二进制日志进行探测，探测其是否发生过改变
   1. 如果探测到 master 主服务器的二进制时间发生了改变，则开始一个 I/O Thread 请求 master 二进制事件日志
3. 同时 master 主服务器为每个 I/O Thread 启动一个 dump Thread，用于向其发送二进制事件日志
4. slave 从服务器将接收到的二进制事件日志保存至自己本地的中继日志文件中
5. slave 从服务器将启动 SQL Thread 从中继日志中读取二进制日志，在本地重放，使得其数据和主服务器保持一致
6. 最后 I/O Thread 和 SQL　Thread　将进入睡眠状态，等待下一次被唤醒

### canal工作原理

1. canal 模拟 Mysql slave 的交互协议，将自己作为 Mysql slave ，向 Mysql master 发送 dump 协议
2. MySQL master 收到 dump 请求，开始推送 binary log 给 slave（即cancal）
3. canal 解析 binary log 对象（原始为 byte 流）

``mysql设置``

```sql
select version();

show master status ;

show variables like 'log_bin' ;

select * from mysql.user ;
create user 'canal'@'%' identified by 'canal';
GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' WITH GRANT OPTION;
# 因为mysql8.0.3后身份检验方式为caching_sha2_password，但canal使用的是mysql_native_password，因此需要设置检验方式（如果该版本之前的可跳过），否则会报错IOException: caching_sha2_password Auth failed
ALTER USER 'canal'@'%' IDENTIFIED WITH mysql_native_password BY 'canal';
flush privileges ;
```



### 代码

>连接canal,进行监听,当获取到mysql的操作后再做对应的redis操作

```go
package main

import (
	"fmt"
	"log"
	"os"
	"redisDemo/services/redis"
	"time"

	"github.com/golang/protobuf/proto"
	"github.com/withlin/canal-go/client"
	pbe "github.com/withlin/canal-go/protocol/entry"
)

func main() {
	// 192.168.199.17 替换成你的canal server的地址
	// example 替换成-e canal.destinations=example 你自己定义的名字
	connector := client.NewSimpleCanalConnector("127.0.0.1", 11111, "", "", "example", 60000, 60*60*1000)
	err := connector.Connect()
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}
	/*
		// https://github.com/alibaba/canal/wiki/AdminGuide
		//mysql 数据解析关注的表，Perl正则表达式.
		//
		//多个正则之间以逗号(,)分隔，转义符需要双斜杠(\\)
		//
		//常见例子：
		//
		//  1.  所有表：.*   or  .*\\..*
		//	2.  canal schema下所有表： canal\\..*
		//	3.  canal下的以canal打头的表：canal\\.canal.*
		//	4.  canal schema下的一张表：canal\\.test1
		//  5.  多个规则组合使用：canal\\..*,mysql.test1,mysql.test2 (逗号分隔)
	*/
	//err = connector.Subscribe(".*\\..*")
	err = connector.Subscribe("test\\.test1")
	if err != nil {
		log.Println(err)
		os.Exit(1)
	}

	for {
		message, err := connector.Get(100, nil, nil)
		if err != nil {
			log.Println(err)
			os.Exit(1)
		}
		batchId := message.Id
		if batchId == -1 || len(message.Entries) <= 0 {
			time.Sleep(300 * time.Millisecond)
			//fmt.Println("===没有数据了===")
			continue
		}

		printEntry(message.Entries)

	}
}

func printEntry(entrys []pbe.Entry) {

	for _, entry := range entrys {
		if entry.GetEntryType() == pbe.EntryType_TRANSACTIONBEGIN || entry.GetEntryType() == pbe.EntryType_TRANSACTIONEND {
			continue
		}
		rowChange := new(pbe.RowChange)

		err := proto.Unmarshal(entry.GetStoreValue(), rowChange)
		checkError(err)
		if rowChange != nil {
			eventType := rowChange.GetEventType()
			header := entry.GetHeader()
			fmt.Println(fmt.Sprintf("================> binlog[%s : %d],name[%s,%s], eventType: %s", header.GetLogfileName(), header.GetLogfileOffset(), header.GetSchemaName(), header.GetTableName(), header.GetEventType()))

			for _, rowData := range rowChange.GetRowDatas() {
				if eventType == pbe.EventType_DELETE {
					/*
						====>binlog[mysql-bin.ooe001 : 3292],name [test, test1], eventType ： DELETE id : 1 update= false
						name : aaa update= false
						age : 23 update= false
						password : updateTest update= false
						phoneNumber : update= false
						text : update= false
					*/
					cs := rowData.GetBeforeColumns()
					printColumn(cs)
          // 这里做对redis的操作
					redis.Delete(cs)
				} else if eventType == pbe.EventType_INSERT {
					/*
						====>binlog[mysql-bin.000001 : 26381,name [test, test1], eventType: INSERT id : 1 update= true
						name : abc update= true
						age : 11 update= true
						password : asdasdasd update= true
						phoneNumber : update= true
						text : update= true
					*/
					cs := rowData.GetAfterColumns()
					printColumn(cs)
          // 这里做对redis的操作
					redis.Update(cs)
				} else {
					/*
						===>binlog [mysql-bin.000001 : 2957],name [test, test1], eventType : UPDATE
						-> before
						id : 1 update= false
						name : abc update= false
						age : 11 update= false
						password : asdasdasd update= false
						phoneNumber ： update= false
						text : update= false
						一------> aften
						id : 1 update= false
						name : aaa update= trve
						age : 23 update= true
						password : updateTest update= true
						phoneNumber • update= false
						text • update= false
					*/
					//fmt.Println("-------> before")
					//printColumn(rowData.GetBeforeColumns())
					//fmt.Println("-------> after")
					cs := rowData.GetAfterColumns()
					printColumn(cs)
          // 这里做对redis的操作
					redis.Update(cs)
				}
			}
		}
	}
}

func printColumn(columns []*pbe.Column) {
	for _, col := range columns {
		fmt.Println(fmt.Sprintf("%s : %s  update= %t", col.GetName(), col.GetValue(), col.GetUpdated()))
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}

```



# 数据统计

## 统计类型

1. 聚合统计
   1. 求交集并集和差集
2. 排序统计
   1. 求前n条数据
3. 二值统计
   1. 统计结果只有0和1的数据
4. 基数统计
   1. 不重复数据的统计



## hyperLoglog

### UV

>Unique Visitor 独立访客一般为客户端ip, 需要去重

### PV

>Page View 页面浏览量, 不需要去重

### DAU

>Daily Active User 日活跃用户量, 登录或使用了某个产品的用户数,需要去重, 常用于反馈运营情况

### MAU

>Monthly Active User 月活跃用户量





# 分布式锁

1. 独占性
   1. 任何时刻有且仅有一个线程拥有这个锁
2. 高可用
   1. redis在集群环境下不能因为主节点挂了而导致这个锁不可用
   2. 高并发下性能依旧好
3. 防死锁
   1. 要有超时控制
4. 不乱入
   1. 不能unlock别的线程的锁
5. 重入性
   1. 同一个节点的同一个线程获得锁后它也可以再次获得这个锁
   1. 重入锁也叫递归锁



>不使用setnx因为setnx无法做到可重入,所以使用hset

```go
package lua

import (
	"context"
	"github.com/go-redis/redis/v8"
	"github.com/google/uuid"
	"runtime"
	"sync"
	"time"
)

type RedisLock struct {
	redisClient *redis.Client   //redis客户端
	context     context.Context //上下文
	lockKey     string          //锁名称
	lockValue   string          //锁的值,防止乱入
	expire      int             //重入了多少次
}

const (
	REDIS = "redis"
	MYSQL = "mysql"
)

// NewDistributedLocks 使用工厂设计模式易扩展
func NewDistributedLocks(lockType, lockName string, context context.Context, redisClient *redis.Client) sync.Locker {
	if lockType == REDIS {
		return &RedisLock{
			redisClient: redisClient,
			context:     context,
			lockKey:     lockName,
			lockValue:   uuid.New().String(),
			expire:      10,
		}
	} else if lockType == MYSQL {
		//	TODO
	}
	return nil
}

// Lock 加锁
func (r RedisLock) Lock() {
	lock := "if redis.call('EXISTS', KEYS[1]) == 0 or redis.call('HEXISTS', KEYS[1], ARGV[1]) == 1 then   redis.call('HINCRBY', KEYS[1], ARGV[1], 1)    redis.call('EXPIRE', KEYS[1], ARGV[2])    return 1    else    return 0    end"
	for true {
		re, _ := redis.NewScript(lock).Run(r.context, r.redisClient, []string{r.lockKey}, r.lockValue, r.expire).Int()
		if re != 1 {
			// 自旋
			runtime.Gosched()
		} else {
			// 开启一个goroutine用来检测是否需要续期
			go r.Renewal()
			break
		}
	}
}

// Unlock 释放锁
func (r RedisLock) Unlock() {
	unlock := "if redis.call('HEXISTS', KEYS[1], ARGV[1]) == 0 then    return -1    elseif redis.call('HINCRBY', KEYS[1], ARGV[1], -1) == 0 then    return redis.call('DEL', KEYS[1])    else    return 0    end"
	redis.NewScript(unlock).Run(r.context, r.redisClient, []string{r.lockKey}, r.lockValue)
}

// Renewal 续期
func (r RedisLock) Renewal() {
	renewalLock := "if redis.call('HEXISTS', KEYS[1], ARGV[1]) == 1 then    return redis.call('EXPIRE', KEYS[1], ARGV[2])    else    return 0    end"
	for true {
		select {
		case <-time.Tick(time.Duration(r.expire/3) * time.Second):
			if res, _ := redis.NewScript(renewalLock).Run(r.context, r.redisClient, []string{r.lockKey}, r.lockValue, r.expire).Int(); res == 0 {
				break
			}
		}
	}
}

```





































