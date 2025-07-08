---
title: "Go Redis项目使用"
date: 2022-10-23T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- redis
---

<h1><center>redis实战总结</center></h1>

# 一：redis存储验证码

## 相关代码

只展示相关技术点的代码，看完可自己实现

> 生成验证码

```go
func Code() string {
	rnd := rand.New(rand.NewSource(time.Now().UnixNano()))
	vcode := fmt.Sprintf("%06v", rnd.Int31n(1000000))
	return vcode
}
```

>存储和读取redis

```go
client := redis.NewClient("url")
// key可以为用户手机号、value为Code()生成的验证码；成功保存的返回值为：result:ok err:nil
result, err := client.Set(context.TODO(), "key", "value", time.Second*60).Result()
// result为 key 对应的 value
result, err = client.Get(context.TODO(), "key").Result()
```

>生成jwt/判断jwt

```go
/*参数结构体为自定义结构体：
type JwtClaims struct {
	tb.TbUser
	jwt.StandardClaims
}
只要包含jwt.StandardClaims即可，其中jwt.StandardClaims的ExpiresAt为设置过期时间
*/
func CreateToken(jwtClaims sys.JwtClaims) (string, error) {
	claims := jwt.NewWithClaims(jwt.SigningMethodHS256, &jwtClaims)
	token, err := claims.SignedString("miyao suibian")
	return token, err
}

func ParseToken(token string) (*sys.JwtClaims, error) {
	claims, err := jwt.ParseWithClaims(token, &sys.JwtClaims{}, func(token *jwt.Token) (interface{}, error) {
		return "miyao suibian", nil
	})
	if err != nil {
		return nil, err
	}
  // 返回值强转为自定义结构体的类型，方便取值
	return claims.Claims.(*sys.JwtClaims), nil
}
```



# 二：redis分布式锁

## 原因

>用redis分布式锁的原因：如果用语言自带的锁只能在当前的进程中锁，如果后续服务不再是单体服务而是横向扩展为多个服务同时运行在多个进程中的时候是无法共享锁的，所以需要用到分布式锁，用来在多个进程中共享锁

## 原理

>redis中有一个命令：setnx key value
>
>不存在这个key的时候执行该命令返回值为1，当存在key的时候返回值为0
>
>可以借助这个命令来实现锁：
>
>当多个线程并发过来的时候，都要先执行setnx命令，返回值为1的即为拿到锁，返回值为0的为没有拿到锁，当拿到锁的线程执行完毕后执行del key 命令删除该key，后续的线程再执行setnx返回值为1即拿到锁。根据这个原理来实现redis分布式锁

## 使用

>使用的时候先NewRedisLock(锁名称, 锁值, 锁的ttl)创建锁，然后TryLock()获取锁，最后UnLock()释放锁

## lua脚本

>实现redis分布式锁需要用到lua脚本，用lua脚本的目的是为了保证执行多条redis命令时的原子性

```lua
== UnLock()中要使用
UnLockLuaScript := `
if redis.call("GET",KEYS[1]) == ARGV[1] then
return redis.call("DEL",KEYS[1])
else
	return 0
end`
```

## 代码

```go
// 存储到redis中的key前缀，目的是为了方便查看
const lockPrefix = "lock:"

// RedisLock redis 分布式锁
type RedisLock struct {
	LockName  string	// 锁的名称，也就是存储到redis中的key
	LockValue string	// 锁的内容，也就是存储到redis中的value
	TTl       time.Duration	// 锁的过期时间防止死锁
}

// 创建锁，参数为：锁名称、锁的值、锁的过期时间
func NewRedisLock(lockName, lockValue string, tll time.Duration) *RedisLock {
	return &RedisLock{
		LockName:  lockName,
		LockValue: lockValue,
		TTl:       tll,
	}
}

// 获取锁，返回值为是否获取到锁
func (r *RedisLock) TryLock() bool {
	result, err := global.RedisDb.SetNX(context.TODO(), lockPrefix+r.LockName, r.LockValue, r.TTl).Result()
	if err != nil {
		log.Println(err)
    return false
	}
	return result
}

// 释放锁，不需要返回值
func (r *RedisLock) UnLock() {
  // 创建脚本，UnLockLuaScript的lua脚本上边有写
	script := redis.NewScript(UnLockLuaScript)
  // 运行脚本，第二个参数为client，第三个字符串数组表示的是lua脚本中KEYS，第四个参数为lua脚本中的ARGV
	script.Run(context.TODO(), redis.NewClient("url"), []string{lockPrefix + r.LockName}, r.LockValue)
}
```



# 三：redis缓存

## 缓存

### 流程图：

>场景：客户端通过id查询对应的商铺信息

<img src="../../../img/go-redis1.png" alt="acl" style="zoom:100%;" />

### 代码：

>每一行都有注释，代码不难理解，这个思路要明白代码是很简单的

```go
func ById(id int) dto.Response {
  // global.ShopCache：前缀，查看方便
	key := global.ShopCache + strconv.Itoa(id)
	// 布隆过滤器判断是否有改key，解决缓存穿透，后边会介绍布隆过滤器
	if !global.Bloomfilter.Contain([]byte(key)) {
		return dto.Err(key + "不存在")
	}
	var shop tb.TbShop
	// 从redis缓存中获取店铺信息，这里就是简单的get命令只是这里进行了再次封装把redis的返回值转为对应的结构体而已
	boolRes, _ := utils.EnterUtilsApp.RedisCacheUtils.GetCacheData(key, &shop)
	// redis锁的key，这里锁的key用uuid+商铺id组成保证value的唯一以免删除了不属于自己的锁
  // key一样value不一样，当释放锁的时候会先判断一下自己的value和redis中存储的value是否一致，一致说明是自己的锁既可以释放(删除key)，如果value不一样则不进行删除操作因为这个不是自己的锁
	lockValue := uuid.NewV4().String() + strconv.Itoa(id)
	// false说明没有该缓存
	if !boolRes {
    // 创建锁
		rlock := redisLock.NewRedisLock(key, lockValue, time.Second*2)
		// 获取锁，拿到锁的去数据库查询重建缓存，拿不到的等待
		for !rlock.TryLock() {
			time.Sleep(time.Millisecond * 50)
		}
		// DCL双重检查缓存。获取到锁再次检查是否有缓存，防止拿到锁后多次重建缓存
		boolRes, _ := utils.EnterUtilsApp.RedisCacheUtils.GetCacheData(key, &shop)
		// 如果有缓存直接返回
		if boolRes {
			rlock.UnLock()
			return dto.OkData(shop)
		}
		// 模拟重建缓存需要很长时间
		//time.Sleep(time.Millisecond * 200)

		// 根据id查询商铺信息，不需要判断返回值，因为已经把id全部添加到布隆过滤器中了，而在开头已经判断了布隆过滤器中是否有这个id，代码能走到这里说明一定是存在这个id的
global.MysqlDb.Model(&tb.TbShop{}).Scopes(EnterServicesApp.PaginateService.byId(id)).Find(&shop).RowsAffected
		// 重建缓存，json序列化shop存入redis，这里是使用的再次封装的函数，这个函数只是把这个shop进行序列话然后存储redis中。
		utils.EnterUtilsApp.RedisCacheUtils.AddCacheDataAndSetTTL(key, shop)
		rlock.UnLock()
		return dto.OkData(shop)
	}
	return dto.OkData(shop)
}
```

## 更新缓存

>数据库对缓存数据进行修改的时候一起对redis的缓存数据进行修改，这里不再展示代码
>
>也有别的更新缓存策略这里也不再细说，有兴趣的自行百度学习



# 四：缓存三大问题：击穿、雪崩、穿透

## 产生问题的原因

>问题描述请看：https://editor.csdn.net/md/?articleId=127782108

## 解决

### 击穿：

>缓存击穿也叫热点key问题，就是一个被`高并发访问`并且`缓存重建业务比较复杂`的key突然失效，无数的请求访问会在瞬间给数据库带来巨大的冲击。
>
>解决就是使用互斥锁，参考上面的redis缓存场景，当缓存失效的时候加锁，只有一个去重建缓存



### 雪崩：

>缓存雪崩是指在同一时段`大量的缓存key同时失效或者redis服务宕机`，导致大量请求直达数据库，带来巨大压力。
>
>代码层解决就是在对key设置ttl的时候不要全部都设置一样的，使key失效的时候不是大面积的失效就行
>
>其他办法：搭建集群，服务降级限流，添加多级缓存(redis缓存失效还有别的缓存支撑)



### 穿透：

>缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。
>
>代码解决：布隆过滤器
>
>布隆过滤器就是把key进行多次hash然后用hash后的值作为索引，索引对应的值设置为1，默认为0
>
>查询的时候一样把key 多次hash查询对应hash索引对应的值是否都为1，都为1说明存在否则不存在
>
>推荐使用库：
>
>```go
>"github.com/linvon/cuckoo-filter"
>```

# 布隆过滤器

使用：

```go
// 创建过滤器
cf := cuckoo.NewFilter(4, 9, 长度, cuckoo.TableTypePacked)
// 添加key
cd.Add("key")
// 判断是否存在,返回值是布尔类型，存在为true不存在false
cf.Contain([]byte(key))

// 项目使用的时候可以在项目运行后先执行添加key操作，把所有要进行缓存的key都查询出来添加到过滤器中，在后边查询缓存的时候先从过滤器中判断一下要查询的这个缓存key是否存在，存在的前提下再去查询缓存
```



# 五：秒杀

<img src="../../../img/go-redis2.png" alt="acl" style="zoom:100%;" />

>通过lua脚本可以避免超卖问题和线程安全问题，甚至于可以全程不加锁

>场景：优惠卷秒杀
>
>前提条件：在上架优惠卷的时候不仅把优惠卷信息存储到mysql中还要把信息存储到redis中，redis中的存储只需要优惠卷id和优惠卷数量即可

>lua脚本中要使用到的redis命令：
>
>sismember、incrby、sadd
>
>sismember key value	判断集合key中是否存在value，不存在返回0，存在返回1
>
>```shell
>127.0.0.1:6379> SISMEMBER k v1
>(integer) 0
>127.0.0.1:6379> sadd k v1
>(integer) 1
>127.0.0.1:6379> SISMEMBER k v1
>(integer) 1
>```
>
>sadd key value 	向集合key中添加value
>
>incrby key value  对一个key的value做加法
>
>```shell
>127.0.0.1:6379> set count 1
>OK
>127.0.0.1:6379> get count
>"1"
>127.0.0.1:6379> INCRBY count 1
>(integer) 2
>127.0.0.1:6379> get count
>"2"
>127.0.0.1:6379> INCRBY count -1
>(integer) 1
>127.0.0.1:6379> get count
>"1"
>```
>
>



## 代码：

>lua脚本

```lua
	IsQualificationLuaScript = `
	local voucherId = ARGV[1]
	local userId = ARGV[2]
	local stockKey = 'seckill:stock:' .. voucherId
	local orderKey = 'seckill:order' .. voucherId
	-- 判断是否有库存
	if (tonumber(redis.call('get',stockKey)) <= 0) then
		return 1
	end
	-- 判断用户是否下过单
	if (redis.call('sismember',orderKey,userId) == 1) then
		return 2
	end
	-- 扣库存，吧stockKey的值-1
	redis.call('incrby',stockKey,-1)
	-- 下单，把用户id存到orderKey这个set集合中
	redis.call('sadd',orderKey,userId)
	return 0
	`
```



```go
func (VouCher) VoucherOrderId(voucherId string, c *gin.Context) dto.Response {
	// 获取用户信息，因为用户信息都在jwt中，只需要解析jwt即可拿到用户信息
	user := utils.EnterUtilsApp.JwtUtils.JwtGetUser(c)
	// 执行lua脚本判断用户是否有资格抢购优惠卷; 返回值： 0：有资格；1：没有库存；2：已经抢购过了；
	result, _ := redis.NewScript(global.IsQualificationLuaScript).Run(global.Content, global.RedisDb, []string{}, voucherId, user.ID).Result()
	if result == nil {
		return dto.Err("不存在该优惠卷")
	}
	// 根据返回值来判断
	switch result.(int64) {
	case 1:
		return dto.Err("库存不足")
	case 2:
		return dto.Err("一人只可抢购一单")
	}
	// 封装订单信息
	voucherOrder := returnVoucherOrder(user.ID, voucherId)
	// 订单信息json序列化
	orderJson, _ := json.Marshal(voucherOrder)
  // 发送订单信息，订单消息通过mq进行异步处理，只需要再开启线程用来专门的处理订单信息即可(在数据库中创建订单信息)，这样可以增加服务的处理能力，这里不再展示这部分代码
	rabbitmq.Producer(global.VoucherOrderRabbitMQQueueName, orderJson)
	// 返回订单ID
	return dto.OkData(voucherOrder.ID)
}

func returnVoucherOrder(userId uint64, voucherID string) *tb.TbVoucherOrder {
	// 新增订单
	voucherOrder := tb.TbVoucherOrder{}
	// 订单ID，全局唯一ID，这里使用的是自己封装生成唯一id的函数，也可以使用uuid等..
	voucherOrder.ID = utils.EnterUtilsApp.RedisIDWorker.NextId("voucherOrder")
	// 用户ID
	voucherOrder.UserID = userId
	// 优惠卷ID
	id, _ := strconv.Atoi(voucherID)
	voucherOrder.VoucherID = uint64(id)
	// 默认为 未支付、余额支付、订单创建时间
	voucherOrder.Status = 1
	voucherOrder.PayType = 1
	voucherOrder.CreateTime = global.TimeNow
	// 返回订单结构体
	return &voucherOrder
}
```



# 六：pv和uv统计

>pv：记录网站有多少用户访问
>
>uv：记录网站的点击量

## redis命令

>相同的元素是不会重复添加的

```shell
127.0.0.1:6379> PFADD pv 1 2 3 4 5 6 7
(integer) 1
127.0.0.1:6379> PFCOUNT pv
(integer) 7
127.0.0.1:6379> PFADD pv 1 2 3 4 5 6 7
(integer) 0
127.0.0.1:6379> PFCOUNT pv
(integer) 7
```



## 代码

>这里用年-月作为key，用来统计每月的pv和uv
>
>统计uv的时候key为年-月，value为用户的id
>
>而pv就随便了，只要是不重复的即可这里用的是uuid
>
>使用方法也很简单，把这些函数封装成中间件，然后在gin中use即可

```go
func UV(userId uint64) {
	key := fmt.Sprintf("%s:%d-%s", "uv", time.Now().Year(), time.Now().Format("01"))
	global.RedisDb.PFAdd(global.Content, key, userId)
}

func PV() {
	key := fmt.Sprintf("%s:%d-%s", "pv", time.Now().Year(), time.Now().Format("01"))
	global.RedisDb.PFAdd(global.Content, key, uuid.NewV4().String())
}

func UVCount() int64 {
	key := fmt.Sprintf("%s:%d-%s", "uv", time.Now().Year(), time.Now().Format("01"))
	result, err := global.RedisDb.PFCount(global.Content, key).Result()
	if err != nil {
		return 0
	}
	return result
}

func PVCount() int64 {
	key := fmt.Sprintf("%s:%d-%s", "pv", time.Now().Year(), time.Now().Format("01"))
	result, err := global.RedisDb.PFCount(global.Content, key).Result()
	if err != nil {
		return 0
	}
	return result
}
```

