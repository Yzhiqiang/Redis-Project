

#### 报错：config set stop-writes-on-bgsave-error **no**

redis-benchmark是一个压力测试工具！官方自带的性能测试工具

#### redis-benchmark命令参数

![image-20210920151728679](C:\Users\Yu_zhiqiang\AppData\Roaming\Typora\typora-user-images\image-20210920151728679.png)

测试：

```back
#测试100个并发连接，100000请求
redis-benchmark -h localhost -p 6739 -c 100 -n 100000

```



redis 默认有16个数据库：

![image-20210920154420019](C:\Users\Yu_zhiqiang\AppData\Roaming\Typora\typora-user-images\image-20210920154420019.png)

默认使用的是第0个

可以使用select进行切换

```bash
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> select 3    切换数据库
OK
127.0.0.1:6379[3]> dbsize    查看当前数据库大小
(integer) 0
127.0.0.1:6379[3]> 
```

查看当前数据库中所有的key

```bash
127.0.0.1:6379> keys *
1) "name"

flushdb   清空当前数据库
flushall  清空所有数据库

```

redis 是单线程的！！！

Redis是很快的，Redis是基于内存操作的，CPU不是Redis性能瓶颈，Redis的瓶颈是根据机器的内存和带宽，既然可以使用单线程来实现，就是用单线程。



#### Redis为什么单线程还这么快？

1.误区1；高性能的服务器一定是多线程的

2.误区2：多线程（CPU上下文会切换！）一定比单线程效率高



核心：

redis是将所有的数据全部放在内存中的，所以说使用单线程操作效率就是最高的，多线程（CPU上下文切换，非常耗时间）对于内存系统来说，如果没有上下文切换效率就是最高的！

#### 五大基本数据类型：

Redis-key

```bash
exists  name   判断key是否存在
move name 1   移除key=name
expire  name    设置过期时间
ttl name    查看当前key的剩余时间
type name   查看当前key的类型
append name  向一个字符串后面添加一个字符串
strlen name 一个字符串的长度


incr  views     加1
incrby  views 10  加10
decr  views     减1
decrby views 10 减10


getrange key start end 截取字符串从start到end的子串
setrange key 开始 要替换的字符串 替换指定位置开始的字符串！
setex   set with expire   设置过期时间
setex key3 30 "hello"   设置key3的值为hello，30秒后过期
setnx   set if not exists  如果不存在就设置
mset   k1 v1 k2 v2 k3 v3   同时创建多个
mget k1 k2 k3 同时获取多个

msetex  同时设置多个值的过期时间
msetnx  同时设置如果某个值不存在的话，原子性操作，要么一起失败，要么一起成功

getset，  如果不存在则返回null返回0，如果存在则返回原来的值，设置新的值

set user:1{name:hanxing, age:1}=
mset user:1:name=hanxing user:1:age=1
```



List

所有的list操作前面都以l开头

```bach
lpush  list one  从列表的左边插入一个值
lrange list 0 -1  获取list中的值
rpush list righr  从列表的右边插入一个值
lpop list 从左边移除第一个值
Lindex list 1 取list中下标为1的值
Llen list   获取list的长度

lrem  list 1 one  移除list中为one的1个
ltrim  mylist  1 2 截取list中1到2

rpoppush   移动列表的最后一个元素将它放到新的列表中
lset
将列表中指定下标的值替换为另外一个值，更新操作

Linsert mylist before "world" "other"
```

>小结：

实际上是一个链表，befor Node after， left， right都可以插入值

如果key不存在，创建新的链表

如果key存在，新增内容

如果移除所有值，空链表，也代表不存在

在两边插入或者改动值，效率最高，中间元素，相对来说效率会低一点！



消息排队，消息队列



set

```bach
sadd myset hello  set集合中添加值
sadd myset kuangshen
sadd myset loveyu

smemebers myset  查看指定set的所有制

sismember myset hello  查看指定set中是否存在hello
scard myset  查看集合中元素的个数
srem myset hello

srandmember myset 2   随机抽出集合中的指定个数的元素
spop myset         随机删除集合中的一个元素
sdiff set1 set2   求两个集合的差集
sinter set1 set2  求两个集合的交集
sunion set1 set2  求两个集合的并集
```



hash

Map集合，key-map，本质和String类型没有太大区别，还是一个简单的key-value！

```bash
127.0.0.1:6379> hset myhash field1 kuangshen  #set 一个具体的key-value
(integer) 1
127.0.0.1:6379> hget myhash field
(nil)
127.0.0.1:6379> hget myhash field1
"kuangshen"
127.0.0.1:6379> hmset myhash field1 hello field2 world  #同时设置多个key-value
OK
127.0.0.1:6379> hmget myhash field1 field2 #同时获取多个字段值
1) "hello"
2) "world"
127.0.0.1:6379> hgetall myhash   #获取全部的数据
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> hdel myhash field1   #删除一个指定key-value
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
hlen    myhash   #求hash里面的key-value字段的个数
hkeys   myhash   #获取所有的字段名
hvalues  myhash  #获取所有的

```

hash变更的数据 user name age, 尤其是用户信息之类的，经常变动的信息！hash更适合对象的存储，String更加是个字符串存储！



Zset（有序集合）

在set基础上，增加了一个值，zset k1 score1 v1

```bash
zadd myset 1 one   # 添加一个值
 
zadd myset 2 two 3 three   #添加多个值
###############################################################
实现排序
127.0.0.1:6379> zrangebyscore salary -inf +inf  #从小到大排序
1) "kuangshen"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> zrevrange salary 0 -1     		#从大到小排序
1) "zhangsan"
2) "kuangshen"

127.0.0.1:6379> zrangebyscore salary -inf +inf withscores    #从小到大排序，并且附带成绩
1) "kuangshen"
2) "500"
3) "xiaohong"
4) "2500"
5) "zhangsan"
6) "5000"

zrem salary xiaohong   #移除一个元素
zcard salary       #获取集合中的个数
zcount salary     #获取指定区间的元素的数量


```



GEO

![image-20210921212129315](C:\Users\Yu_zhiqiang\AppData\Roaming\Typora\typora-user-images\image-20210921212129315.png)

>geoadd      

```bash
#getadd   添加地理位置
#规则：  两级无法直接添加，我们一般会下载城市数据，直接通过java程序一次性导入
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqin 114.05 22.52 shengzhen
(integer) 2
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2

```

>geopos

127.0.0.1:6379> geopos china:city beijing

1) "116.39999896287918091"
2) "39.90000009167092543"

127.0.0.1:6379> geopos china:city beijing chongqin

1. "116.39999896287918091"

2. "39.90000009167092543"

   1."106.49999767541885376"

   2."29.52999957900659211"

>GEODIST

两个人之间的距离：

单位：

​	m表示单位为米

​	km表示单位为千米

​	mi表示单位为英里

​	ft表示单位为英尺

```push
27.0.0.1:6379> GEodist china:city beijing shanghai
"1067378.7564"
127.0.0.1:6379> GEodist china:city beijing shanghai km   #查看北京到上海的直线距离
"1067.3788" 
127.0.0.1:6379> GEodist china:city beijing chongqin km    #查看北京到重庆的直线距离
"1464.0708"

```

>GEOredius  以给定经纬度为中心，找出某一半径内的元素

我附近的人（获得所有附近的人的地址，定位！），通过半径来查询！

```bash
127.0.0.1:6379> georadius china:city 110 30 500 km    # 以110 30 为中心，500km为半径查找
1) "chongqin"
2) "xian"
```

