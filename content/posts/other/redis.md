---
title: "redis"
date: 2020-12-04T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- redis
- other
tags: 
- redis
- nosql
---

# String

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



# List

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



# Set

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



# ZSET

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
|                                 |                                                              |
|                                 |                                                              |
|                                 |                                                              |
|                                 |                                                              |
|                                 |                                                              |

