## 五种基本数据类型
###String

 String 类型的常用场景：

- 计数器

- 统计多单位的数量 (set userId:xxxx:article_number 0)

- 粉丝数

- 对象缓存存储 （setex)

  set

#### get append  strlen

```bash
###############################################################################################

127.0.0.1:6379> set redis_test_string_1 xuexiredis 					#set a string key-value value is "xuexiredis"
OK
127.0.0.1:6379> get redis_test_string_1 										#get a key's value
"xuexiredis"
127.0.0.1:6379> APPEND redis_test_string_1 _append_string  	#append string to string type key value data
(integer) 24
127.0.0.1:6379> get redis_test_string_1                     #get a key's value after append
"xuexiredis_append_string"
127.0.0.1:6379> APPEND redis_test_string_2 append_not_exist_string    #append string to an non-exist key, equal set new key
(integer) 23
127.0.0.1:6379> get redis_test_string_2
"append_not_exist_string"
127.0.0.1:6379> strlen redis_test_string_2 # strlen, return the string's length
(integer) 23
127.0.0.1:6379>

#########################################################################
```
####  incr decr incrby decrby

```bash
######################
# incr
# decr
# incrby
# decrby
127.0.0.1:6379> set redis_test_views 0    #counter increase and decrease;"incr" and "decr" Step lengths is 1
OK
127.0.0.1:6379> get redis_test_views
"0"
127.0.0.1:6379> incr redis_test_views
(integer) 1
127.0.0.1:6379> incr redis_test_views
(integer) 2
127.0.0.1:6379> decr redis_test_views
(integer) 1

127.0.0.1:6379> incrby redis_test_views 100  #"incrby" and "decrby" Step lengths can be set
(integer) 101
127.0.0.1:6379> decrby redis_test_views 20   #"incrby"/"decrby" key increment/decrement
(integer) 81
127.0.0.1:6379>
############################################################################ 
```
####  Getrange setrange

```bash
######################
# GETRANGE
# SETRANGE
127.0.0.1:6379> GETRANGE key start end # Intercept string
127.0.0.1:6379> GETRANGE redis_test_string_1 0 1 # "redis_test_string_1" is "xuexiredis"
"xu"
127.0.0.1:6379> GETRANGE redis_test_string_1 0 -1
"xuexiredis_append_string"

127.0.0.1:6379> SETRANGE key offset value  #Replace string with new string
127.0.0.1:6379> setrange redis_test_string_1 0 tihuan # xuexiredis_append_string
(integer) 24
127.0.0.1:6379> get redis_test_string_1 # tihuanedis_append_string
"tihuanedis_append_string"
127.0.0.1:6379> setrange redis_test_string_1 20 tihuan
(integer) 26
127.0.0.1:6379> get redis_test_string_1
"ttihuandis_append_sttihuan"
127.0.0.1:6379> setrange redis_test_string_1 27 haha  #offset + replacment string > strlen
(integer) 31
127.0.0.1:6379> get redis_test_string_1 							#offset > strlen
"ttihuandis_append_sttihuan\x00haha"
###############################################################################################
```
#### setex setnx
```bash
# setex (set with expire)
# setnx (set if not exist)
# distrubute lock use them
127.0.0.1:6379> setex key seconds value 
127.0.0.1:6379> setex redis_ttl_30 30 setttl_is_30s
OK
127.0.0.1:6379> ttl redis_ttl_30
(integer) 19
127.0.0.1:6379> ttl redis_ttl_30
(integer) 1
127.0.0.1:6379> ttl redis_ttl_30
(integer) -2
127.0.0.1:6379> get redis_ttl_30
(nil)

127.0.0.1:6379> setnx key3 "redis is remote dic service"
(integer) 1
127.0.0.1:6379> setnx key3 "mongo db"
(integer) 0
127.0.0.1:6379> get key3
"redis is remote dic service"
127.0.0.1:6379>
#########################################################################
```
#### mset mget msetnx
```bash
######################
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> mset k1 v1 k2 v2 k3 v3 # multi set key value
OK
127.0.0.1:6379[2]> mset k1 v11 k4 v4  # multi set exist key will rewrite the value
OK  
127.0.0.1:6379[2]> keys *
1) "k3"
2) "k4"
3) "k2"
4) "k1"
127.0.0.1:6379[2]> mget k1 k2 k3 k4
1) "v11"
2) "v2"
3) "v3"
4) "v4"
127.0.0.1:6379[2]> msetnx k1 v111 k5 v5 # multi set exist key will not success 原子性操作
OK  
(integer) 0
127.0.0.1:6379[2]> keys *
1) "k3"
2) "k4"
3) "k2"
4) "k1"
127.0.0.1:6379[2]> mget k1 k2 k3 k4
1) "v11"
2) "v2"
3) "v3"
4) "v4"


# object set
# set an key with attribute instead of use json, more effective
127.0.0.1:6379[2]> set user:1 {name:maxu,age:20}
OK
127.0.0.1:6379[2]> get user:1
"{name:maxu,age:20}"
127.0.0.1:6379[2]> mset user:1:name maxu user:1:age 20
OK
127.0.0.1:6379[2]> mget user:1:name user:1:age
1) "maxu"
2) "20"
#########################################################################
```
#### getset
```bash
######################
# getset
127.0.0.1:6379[2]> getset user:1:name nancy
"maxu"
127.0.0.1:6379[2]> get user:1:name
"nancy"
```

###List

​    用来实现栈 队列 阻塞队列

- 实际上是一个链表 left Node right; left right都可以插入 删除

- 如果不存在进行新建链表

- 移除了所有值，空链表

- 在两边插入或者删除效率高。

  

#### lpush lrange rpush

```bash
127.0.0.1:6379[2]> lpush key value [value ...]
127.0.0.1:6379[2]> rpush key value [value ...]
127.0.0.1:6379[2]> lrange key start stop
```

```bash
#########################################################################
127.0.0.1:6379[2]> lpush list one two three
(integer) 3
127.0.0.1:6379[2]> lrange list 0 1
1) "three"
2) "two"
127.0.0.1:6379[2]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
127.0.0.1:6379[2]> rpush list rightval
(integer) 4
127.0.0.1:6379[2]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "rightval"
```

#### lpop rpop

```bash
127.0.0.1:6379[2]> lpop key
127.0.0.1:6379[2]> rpop key
```

```bash
#########################################################################
127.0.0.1:6379[2]> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "rightval"
127.0.0.1:6379[2]> lpop list
"three"
127.0.0.1:6379[2]> lrange list 0 -1
1) "two"
2) "one"
3) "rightval"
127.0.0.1:6379[2]> rpop list
"rightval"
127.0.0.1:6379[2]> lrange list 0 -1
1) "two"
2) "one"

```

#### lindex

```bash
127.0.0.1:6379[2]> lindex key index
```

```bash
#########################################################################
127.0.0.1:6379[2]> lrange list 0 -1
1) "two"
2) "one"
127.0.0.1:6379[2]> lindex list 1
"one"
127.0.0.1:6379[2]> lindex list 0
"two"
```

#### llen

```bash
127.0.0.1:6379[2]> llen key
```

```bash
127.0.0.1:6379[2]> lrange list 0 -1
1) "two"
2) "one"
127.0.0.1:6379[2]> llen list
(integer) 2
```

#### lrem

```bash
127.0.0.1:6379[2]> lrem key count value


127.0.0.1:6379[2]> lpush one two three four three four five
(integer) 6
127.0.0.1:6379[2]> lrange one 0 -1
1) "five"
2) "four"
3) "three"
4) "four"
5) "three"
6) "two"
127.0.0.1:6379[2]> lrem one 1 three # remove from left , remove 1 element which val =  "three"
(integer) 1
127.0.0.1:6379[2]> lrange one 0 -1
1) "five"
2) "four"
3) "four"
4) "three"
5) "two"


127.0.0.1:6379[2]> flushdb
OK
127.0.0.1:6379[2]>  lpush one two three four three four five
(integer) 6
127.0.0.1:6379[2]>  lrange one 0 -1
1) "five"
2) "four"
3) "three"
4) "four"
5) "three"
6) "two"
127.0.0.1:6379[2]> lrem one -1 three  # remove from right , remove 1 element which val =  "three"
(integer) 1
127.0.0.1:6379[2]> lrange one 0 -1
1) "five"
2) "four"
3) "three"
4) "four"
5) "two"
```

#### ltrim

```bash
127.0.0.1:6379[2]> ltrim key start stop

127.0.0.1:6379[2]> rpush mylist "one" "two" "three"
(integer) 3
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "one"
2) "two"
3) "three"
127.0.0.1:6379[2]> ltrim mylist 0 1
OK
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "one"
2) "two"
```

#### rpoplpush

```bash
127.0.0.1:6379[2]> rpoplpush source destination

127.0.0.1:6379[2]> lrange mylist 0 -1
1) "one"
2) "two"
127.0.0.1:6379[2]> rpoplpush mylist yourlist
"two"
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "one"
127.0.0.1:6379[2]> lrange yourlist 0 -1
1) "two"
```

#### Lset  exists

lset replace list type data, index's value

```bash
127.0.0.1:6379[2]> exists testlist
(integer) 0
127.0.0.1:6379[2]> lset testlist 0 item
(error) ERR no such key
127.0.0.1:6379[2]> lpush testlist first_val
(integer) 1
127.0.0.1:6379[2]> exists testlist
(integer) 1
127.0.0.1:6379[2]> lset testlist 0 item
OK
127.0.0.1:6379[2]> lrange testlist 0 -1
1) "item"
127.0.0.1:6379[2]> lset testlist 1 other
(error) ERR index out of range

```

 #### Linsert

```bash
127.0.0.1:6379[2]> linsert key BEFORE|AFTER pivot value
```

##### 问题：list中存在重复值insert

```bash
127.0.0.1:6379[2]> lpush mylist "hello" "world"
(integer) 2
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "world"
2) "hello"
127.0.0.1:6379[2]> linsert mylist before "hello" mid
(integer) 3
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "world"
2) "mid"
3) "hello"
127.0.0.1:6379[2]>
# 问题：如果list中存在多个重复值怎么办 : 最左的第一个匹配的值进行插入
127.0.0.1:6379[2]> lpush mylist hello
(integer) 4
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "hello"
2) "world"
3) "mid"
4) "hello"
127.0.0.1:6379[2]> linsert mylist before "hello" mid2
(integer) 5
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "mid2"
2) "hello"
3) "world"
4) "mid"
5) "hello"
127.0.0.1:6379[2]> linsert mylist after "hello" mid2
(integer) 6
127.0.0.1:6379[2]> lrange mylist 0 -1
1) "mid2"
2) "hello"
3) "mid2"
4) "world"
5) "mid"
6) "hello"
```

### Set

- 集合数据不可重复

#### sadd smembers sismember

```bash
127.0.0.1:6379[2]> sadd key member [member ...] #向无序的set中添加memeber return int,not add return 0
127.0.0.1:6379[2]> smembers key 								# 查看set中的memebers return members of set
127.0.0.1:6379[2]> sismember key member					# member is set's member or not ,return 0 | 1
```



```bash
127.0.0.1:6379[2]> sadd myset hello how are you
(integer) 4
127.0.0.1:6379[2]> smembers myset
1) "you"
2) "are"
3) "hello"
4) "how"
127.0.0.1:6379[2]> sadd myset you
(integer) 0
127.0.0.1:6379[2]> sismember myset you
(integer) 1
127.0.0.1:6379[2]> sismember myset world
(integer) 0
```

#### scard

```bash
127.0.0.1:6379[2]> scard key #获取并返回set中member的个数
```

```bash
127.0.0.1:6379[2]> scard myset
(integer) 4
```

#### srem

- 可一次删除多个，删除的多个member中，如果有一个不在set中也会删除成功

```bash
127.0.0.1:6379[2]> srem key member [member ...]  #remove member from set

```

```bash
127.0.0.1:6379[2]> smembers myset
1) "hello"
2) "are"
3) "you"
4) "how"
127.0.0.1:6379[2]> srem myset world
(integer) 0
127.0.0.1:6379[2]> srem myset hello
(integer) 1
127.0.0.1:6379[2]> smembers myset
1) "are"
2) "you"
3) "how"
127.0.0.1:6379[2]> srem myset are world
(integer) 1
127.0.0.1:6379[2]> smembers myset
1) "you"
2) "how"
```



#### Srandmember

set为无序不重复集合，随机抽一个元素，随机数

```bash
127.0.0.1:6379[2]> srandmember key [count]
```

```bash
127.0.0.1:6379[2]> smembers myset
1) "you"
2) "how"
3) "learner"
4) "I"
5) "new"
6) "a"
7) "am"
27.0.0.1:6379[2]> srandmember myset
"new"
127.0.0.1:6379[2]> srandmember myset
"how"
127.0.0.1:6379[2]> srandmember myset 2
1) "I"
2) "learner"
127.0.0.1:6379[2]> srandmember myset 2
1) "how"
2) "am"
```

#### spop

随机移除一个元素/抽奖？

```bash
127.0.0.1:6379[2]> spop key [count]
```



```bash
127.0.0.1:6379[2]> smembers myset
1) "you"
2) "how"
3) "learner"
4) "I"
5) "new"
6) "a"
7) "am"
127.0.0.1:6379[2]> spop myset
"how"
127.0.0.1:6379[2]> smembers myset
1) "you"
2) "learner"
3) "I"
4) "new"
5) "a"
6) "am"
```

#### smove

将一个memeber移动到另外的set中

```bash
127.0.0.1:6379[2]> smove source destination member
```

- 移动的destination如果不存在，创建新的set

```bash
127.0.0.1:6379[2]> sadd myset "hello" "world" "redis"
(integer) 3
127.0.0.1:6379[2]> smove myset yourset hello
(integer) 1
127.0.0.1:6379[2]> smembers yourset
1) "hello"
```

#### sdiff sinter sunion

- set的交并补

- 适用场景：关注的人，粉丝, 好友，共同好友，二度好友（推荐好友)

```bash
127.0.0.1:6379[2]> sdiff key [key ...]
127.0.0.1:6379[2]> sinter key [key ...]
127.0.0.1:6379[2]> sunion key [key ...]
```

```bash
127.0.0.1:6379[2]> sadd key1 a b c d
(integer) 4
127.0.0.1:6379[2]> sadd key2 c d e f
(integer) 4
127.0.0.1:6379[2]> sdiff key key2
(empty list or set)
127.0.0.1:6379[2]> sdiff key1 key2
1) "b"
2) "a"
127.0.0.1:6379[2]> sinter key1
1) "b"
2) "d"
3) "c"
4) "a"
127.0.0.1:6379[2]> sinter key1 key2
1) "d"
2) "c"
127.0.0.1:6379[2]> sunion key1 key2
1) "b"
2) "f"
3) "d"
4) "e"
5) "c"
6) "a"
```

### Hash

hash可存一些经常变动的对象的信息，比如用户信息（姓名，年龄等等）

#### hset hget hmset hmget hdel

```bash
127.0.0.1:6379[2]> hset key field value
127.0.0.1:6379[2]> hget key field
127.0.0.1:6379[2]> hmset key field value [field value ...]
127.0.0.1:6379[2]> hmget key field [field ...]
127.0.0.1:6379[2]> hgetall key
127.0.0.1:6379[2]> hdel key field [field ...]
```



```bash
127.0.0.1:6379[2]> hset myhash name maxu
(integer) 1
127.0.0.1:6379[2]> hset myhash age 20
(integer) 1
127.0.0.1:6379[2]> hget myhash age
"20"
127.0.0.1:6379[2]> hmset myhash from china location se13gh
OK
127.0.0.1:6379[2]> hmget myhash name age from
1) "maxu"
2) "20"
3) "china"
127.0.0.1:6379[2]> hgetall myhash
1) "name"
2) "maxu"
3) "age"
4) "20"
5) "from"
6) "china"
7) "location"
8) "se13gh"
127.0.0.1:6379[2]> hdel myhash age
(integer) 1
127.0.0.1:6379[2]> hgetall myhash
1) "name"
2) "maxu"
3) "from"
4) "china"
5) "location"
6) "se13gh"
```



#### Hlen

```bash
127.0.0.1:6379[2]> hlen key
```



```bash
127.0.0.1:6379[2]> hgetall myhash
1) "name"
2) "maxu"
3) "from"
4) "china"
5) "location"
6) "se13gh"
127.0.0.1:6379[2]> hlen myhash
(integer) 3
```

#### Hexists	

```bash
127.0.0.1:6379[2]> HEXISTS key field
```



```bash
127.0.0.1:6379[2]> HEXISTS myhash name
(integer) 1
127.0.0.1:6379[2]> HEXISTS myhash age
(integer) 0
127.0.0.1:6379[2]> HEXISTS myhash1 age
(integer) 0
127.0.0.1:6379[2]> HEXISTS myhash1 name
(integer) 0
```



#### hkeys hvals

```bash
127.0.0.1:6379[2]> hkeys key
127.0.0.1:6379[2]> hvals key
```

```bash
127.0.0.1:6379[2]> hvals myhash
1) "maxu"
2) "china"
3) "se13gh"
127.0.0.1:6379[2]> hkeys myhash
1) "name"
2) "from"
3) "location"
127.0.0.1:637
```



#### Hincrby

```bash
127.0.0.1:6379[2]> hincrby key field increment
```

```bash
127.0.0.1:6379[2]> HINCRBY myhash age -1
(integer) 19
127.0.0.1:6379[2]> HINCRBY myhash age 1
(integer) 20
```

#### hsetnx	

```bash
127.0.0.1:6379[2]> hsetnx key field value
```



```bash
127.0.0.1:6379[2]> hkeys myhash
1) "name"
2) "from"
3) "location"
4) "age"
127.0.0.1:6379[2]> hsetnx myhash name 20
(integer) 0
127.0.0.1:6379[2]> hsetnx myhash birthday 19920410
(integer) 1
```

### zset

Zset 有序集合，在set基础上加一个score进行排序；

带权重计算或区分数据；排名等可以使用zset

#### zadd zrange zrevrange

```bash
127.0.0.1:6379[2]> zadd key [NX|XX] [CH] [INCR] score member [score member ...]
127.0.0.1:6379[2]> zrange key start stop [WITHSCORES]
127.0.0.1:6379[2]> ZREVRANGE key start stop [WITHSCORES]

```



```bash
127.0.0.1:6379[2]> zadd myzset 1 "maxu" 2 "hi" 0 "how are u"
(integer) 3
127.0.0.1:6379[2]> zrange myzset 0 -1 withscores
1) "how are u"
2) "0"
3) "maxu"
4) "1"
5) "hi"
6) "2"
127.0.0.1:6379[2]> zrange myzset 0 -1
1) "how are u"
2) "maxu"
3) "hi"
```

#### zrangebyscore zrevrangebyscore

- Zrangebyscore key min max只能从小到大排序

```bash
127.0.0.1:6379[2]> ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
127.0.0.1:6379[2]> ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

```bash
127.0.0.1:6379[2]> zrangebyscore myzset -inf +inf
1) "how are u"
2) "maxu"
3) "hi"
127.0.0.1:6379[2]> zrangebyscore myzset -inf +inf withscores
1) "how are u"
2) "0"
3) "maxu"
4) "1"
5) "hi"
6) "2"
127.0.0.1:6379[2]> zrangebyscore myzset -inf 1 withscores
1) "how are u"
2) "0"
3) "maxu"
4) "1"

127.0.0.1:6379[2]> zrevrangebyscore myzset inf -inf withscores
1) "hi"
2) "2"
3) "maxu"
4) "1"
5) "how are u"
6) "0"
```

#### zrem zcard zcount

```bash
127.0.0.1:6379[2]> zrem key member [member ...]
127.0.0.1:6379[2]> ZCARD key
127.0.0.1:6379[2]> zcount key min max #返回score区间内的member的数量
```

```bash
127.0.0.1:6379[2]> zrevrangebyscore myzset inf -inf withscores
1) "hi"
2) "2"
3) "maxu"
4) "1"
5) "how are u"
6) "0"
127.0.0.1:6379[2]> zrem myzset hi
(integer) 1
127.0.0.1:6379[2]> zrevrangebyscore myzset inf -inf withscores
1) "maxu"
2) "1"
3) "how are u"
4) "0"
127.0.0.1:6379[2]> ZCARD myzset
(integer) 2


127.0.0.1:6379[2]> ZrevRANGebyscore myzset inf -inf withscores
1) "learn redis zset"
2) "100"
3) "be proud of you"
4) "100"
5) "maxu"
6) "1"
7) "how are u"
8) "0"
127.0.0.1:6379[2]> zcount myzset 0 100
(integer) 4
127.0.0.1:6379[2]> zcount myzset 99 100
(integer) 2
127.0.0.1:6379[2]> zcount myzset 99 (100
(integer) 0
```

## 三种其他数据类型

### geospatial



geo是zset实现的数据类型，可用zset的指令进行操作

地理位置

应用：定位，距离计算，附近的人

The exact limits, as specified by EPSG:900913 / EPSG:3785 / OSGEO:41001 are the following:

- Valid longitudes are from -180 to 180 degrees.
- Valid latitudes are from -85.05112878 to 85.05112878 degrees.

#### geoadd

添加经纬度数据到key中

[GEOADD](https://redis.io/commands/geoadd) also provides the following options:

- **XX**: Only update elements that already exist. Never add elements.
- **NX**: Don't update already existing elements. Always add new elements.
- **CH**: Modify the return value from the number of new elements added, to the total number of elements changed (CH is an abbreviation of *changed*). Changed elements are **new elements added** and elements already existing for which **the coordinates was updated**. So elements specified in the command line having the same score as they had in the past are not counted. Note: normally, the return value of [GEOADD](https://redis.io/commands/geoadd) only counts the number of new elements added.

Note: The **XX** and **NX** options are mutually exclusive.

```bash
127.0.0.1:6379[2]> GEOADD key longitude latitude member [longitude latitude member ...
```

```bash
127.0.0.1:6379[2]> GEOADD china:city 116.23128 40.22077 beijing
(integer) 1
127.0.0.1:6379[2]> GEOADD china:city 121.48941 31.40527 shanghai
(integer) 1
127.0.0.1:6379[2]> geoadd china:ciyt 113.88308 22.55329 shenzhen
(integer) 1
127.0.0.1:6379[2]> geoadd china:city 113.88308 22.55329 shenzhen
(integer) 1
127.0.0.1:6379[2]> geoadd china:city 118.95927 42.26581 chifeng
(integer) 1
```

#### geodist

获取distance

```bash
127.0.0.1:6379[2]> geodist key member1 member2 [unit]
```

Return the distance between two members in the geospatial index represented by the sorted set.

Given a sorted set representing a geospatial index, populated using the [GEOADD](https://redis.io/commands/geoadd) command, the command returns the distance between the two specified members in the specified unit.

If one or both the members are missing, the command returns NULL.

The unit must be one of the following, and defaults to meters:

- **m** for meters.
- **km** for kilometers.
- **mi** for miles.
- **ft** for feet.

The distance is computed assuming that the Earth is a perfect sphere, so errors up to 0.5% are possible in edge cases.

```bash
127.0.0.1:6379[2]> GEODIST china:city chifeng beijing
"322131.7755"
127.0.0.1:6379[2]> GEODIST china:city chifeng beijing km
"322.1318"
```

#### geohash

```bash
127.0.0.1:6379[2]> geohash key member [member ...]
```

- **Available since 3.2.0.**

- **Time complexity:** O(log(N)) for each member requested, where N is the number of elements in the sorted set.

#### geopos

获取数据中的经纬度

```bash
127.0.0.1:6379[2]> geohash key member [member ...]
```



```bash
127.0.0.1:6379[2]> GEOPOS china:city chifeng
1) 1) "118.95927160978317261"
   2) "42.26580996767268772"
127.0.0.1:6379[2]> GEOPOS china:city chifeng beijing
1) 1) "118.95927160978317261"
   2) "42.26580996767268772"
2) 1) "116.23128265142440796"
   2) "40.22076905438526495"
```



#### georadius

定位位置周围的member

```bash
127.0.0.1:6379[2]> georadius key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH]

```



```bash
127.0.0.1:6379[2]> georadius china:city 120 38 100 km withdist
(empty list or set)
#withcoord 返回经纬度； withdist 返回距离 withhash 返回哈希值
127.0.0.1:6379[2]> georadius china:city 120 38 1000 km withdist
1) 1) "beijing"
   2) "408.3496"
2) 1) "chifeng"
   2) "482.6418"
3) 1) "shanghai"
   2) "746.0104"
127.0.0.1:6379[2]> georadius china:city 120 31 1000 km withdist
1) 1) "shanghai"
   2) "148.6926"
127.0.0.1:6379[2]> georadius china:city 120 31 1000 km withcoord
1) 1) "shanghai"
   2) 1) "121.48941010236740112"
      2) "31.40526993848380499"
127.0.0.1:6379[2]> georadius china:city 120 31 1000 km withcoord withhash
1) 1) "shanghai"
   2) (integer) 4054807796443227
   3) 1) "121.48941010236740112"
      2) "31.40526993848380499"
# with count 限制返回值      
127.0.0.1:6379[2]> georadius china:city 120 31 10000 km withcoord withhash count 1
1) 1) "shanghai"
   2) (integer) 4054807796443227
   3) 1) "121.48941010236740112"
      2) "31.40526993848380499"
127.0.0.1:6379[2]> georadius china:city 120 31 10000 km withcoord withhash count 2
1) 1) "shanghai"
   2) (integer) 4054807796443227
   3) 1) "121.48941010236740112"
      2) "31.40526993848380499"
2) 1) "beijing"
   2) (integer) 4069896088584598
   3) 1) "116.23128265142440796"
      2) "40.22076905438526495"
```



#### georadiusbymember

```bash
127.0.0.1:6379[2]> georadiusbymember key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key] [STOREDIST key]
```

```bash
127.0.0.1:6379[2]> GEORADIUSBYMEMBER china:city chifeng 400 km withdist
1) 1) "beijing"
   2) "322.1318"
2) 1) "chifeng"
   2) "0.0000"
   
   127.0.0.1:6379[2]> GEORADIUSBYMEMBER china:city chifeng 400 km withdist desc
1) 1) "beijing"
   2) "322.1318"
2) 1) "chifeng"
   2) "0.0000"
127.0.0.1:6379[2]> GEORADIUSBYMEMBER china:city chifeng 400 km withdist asc
1) 1) "chifeng"
   2) "0.0000"
2) 1) "beijing"
   2) "322.1318"
```



#### geosearch

```bash
127.0.0.1:6379[2]> geosearch key [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius m|km|ft|mi] [BYBOX width height m|km|ft|mi] [ASC|DESC] [COUNT count [ANY]] [WITHCOORD] [WITHDIST] [WITHHASH]
```



#### geosearchstore

```bash
127.0.0.1:6379[2]> geosearchstore destination source [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius m|km|ft|mi] [BYBOX width height m|km|ft|mi] [ASC|DESC] [COUNT count [ANY]] [STOREDIST]
```





### hyperloglog

> **什么是基数**
>
> A{1,3,5,6,8} B{1,3,5,6}
>
> 基数：两个集合中不重复元素的个数

> 简介
>
> - Redis Hyperloglog 基数统计的算法！
>
> - **网页访问人次统计UV (一个人访问多次，仍算作一个人访问）**
>
>   传统方式：set中进行userId的记录，统计set中元素的数量进行UV的统计（如果保存大量用户id，相对比较占用内存）。
>
> - 优点：内存占用固定，2^64不用元素的基数，只需要12KB的内存; 错误率在1%以内



#### pfadd pfcount

```bash
127.0.0.1:6379[2]> PFadd key element [element ...]
127.0.0.1:6379[2]> pfcount key [key ...]
127.0.0.1:6379[2]> pfmerge destkey sourcekey [sourcekey ...]
```

```bash
127.0.0.1:6379[2]> PFadd mykey a b c d e f g h i j
(integer) 1
127.0.0.1:6379[2]> pfcount mykey
(integer) 10
127.0.0.1:6379[2]> pfadd mykey2 a d l w o k
(integer) 1
127.0.0.1:6379[2]> pfcount mykey2
(integer) 6
127.0.0.1:6379[2]> pfcount mykey2 mykey #并集
(integer) 14
127.0.0.1:6379[2]> pfmerge destkey mykey mykey2 #并集存到destkey中
OK
127.0.0.1:6379[2]> pfcount destkey
(integer) 14
```



### bitmaps

> 位存储
>
> 统计用户是否在线、是否打卡、用户是否活跃

#### Setbit getbit

```bash
127.0.0.1:6379[2]> setbit key offset value
127.0.0.1:6379[2]> getbit key offset
```

#### bitcount

统计

```bash
127.0.0.1:6379[2]> bitcount key [start end]
```

```bash
127.0.0.1:6379[2]> setbit week_online 0 1
(integer) 0
127.0.0.1:6379[2]> setbit week_online 1 0
(integer) 0
127.0.0.1:6379[2]> setbit week_online 2 1
(integer) 0
127.0.0.1:6379[2]> setbit week_online 3 1
(integer) 0
127.0.0.1:6379[2]> setbit week_online 4 0
(integer) 0
127.0.0.1:6379[2]> setbit week_online 5 0
(integer) 0
127.0.0.1:6379[2]> setbit week_online 6 0
(integer) 0
127.0.0.1:6379[2]> bitcount week_online
(integer) 3
127.0.0.1:6379[2]>
```

## 事务

### 说明

> Redis事务的本质：一组命令的集合，一个事务中所有的命令都被序列化，执行事务的过程中，按照顺序执行指令。
>
> 一次性 顺序性 排他性的执行一系列的指令 

```bash
--------multi  [set get set sadd] exec --------------
```

**redis没有事务隔离级别的概念**

所有指令在事务中，没有被直接执行而是exec的时候按顺序执行。

**redis单条指令是保证原子性的，但是事务不保证原子性**



> redis事务：
>
> - multi
> - [Get/set/lpush/sadd/scard/hset/...]
> - Exec/Discard



锁：redis可以实现乐观锁

```bash
127.0.0.1:6379[2]> multi
OK
127.0.0.1:6379[2]> setex user:maxu 10 age:12
QUEUED
127.0.0.1:6379[2]> lpush user:1 1 2 3456
QUEUED
127.0.0.1:6379[2]> PFADD name 12 12
QUEUED
127.0.0.1:6379[2]> exec #执行事务
1) OK
2) (integer) 3
3) (integer) 1
127.0.0.1:6379[2]> multi
OK
127.0.0.1:6379[2]> set key 1
QUEUED
127.0.0.1:6379[2]> discard #取消事务
OK
```

> 事务中有错误怎么处理？？
>
> 编译型异常：代码有问题，命令出错；所有指令都不会执行
>
> ```bash
> 
> 127.0.0.1:6379[2]> multi
> OK
> 127.0.0.1:6379[2]> set k1 v1
> QUEUED
> 127.0.0.1:6379[2]> set k2 v2
> QUEUED
> 127.0.0.1:6379[2]> set k3 v3
> QUEUED
> 127.0.0.1:6379[2]> setget k3
> (error) ERR unknown command 'setget'
> 127.0.0.1:6379[2]> set key4 v454
> QUEUED
> 127.0.0.1:6379[2]> exec
> (error) EXECABORT Transaction discarded because of previous errors.
> 127.0.0.1:6379[2]> get key4
> (nil)
> 127.0.0.1:6379[2]> get k1
> (nil)
> ```
>
> 运行时异常：代码没问题（类似1/0），如事务队列中有某条指令非代码异常但是有逻辑错误，执行时，其他命令可正常执行。
>
> ```bash
> 127.0.0.1:6379[2]> multi
> OK
> 127.0.0.1:6379[2]> set k1 v1
> QUEUED
> 127.0.0.1:6379[2]> incr k1
> QUEUED
> 127.0.0.1:6379[2]> set k2 2
> QUEUED
> 127.0.0.1:6379[2]> decr k2
> QUEUED
> 127.0.0.1:6379[2]> set k3 v3
> QUEUED
> 127.0.0.1:6379[2]> get k1
> QUEUED
> 127.0.0.1:6379[2]> exec
> 1) OK
> 2) (error) ERR value is not an integer or out of range #命令无问题，有逻辑错误，本条指令不执行。
> 3) OK
> 4) (integer) 1
> 5) OK
> 6) "v1"
> ```
>
> 

### 锁- watch

> 监视！ Watch



> 悲观锁：
>
> - 无论执行什么操作都会有问题，无论什么操作都加锁执行
>
> 乐观锁：
>
> - 无论什么时候都不会有问题，所以不上锁，更新数据的时候判断一下在此期间是否有被修改过数据。
> - 获取version
> - 更新的时候比较version



**redis测试监测**

- Watch key
- 新开一个进程模拟多线程更改对应的key.
- 原进行执行已经键入的事务，将执行失败：

> 插队的更新操作：
>
> ```bash
> redis-cli
> 127.0.0.1:6379> select 2
> OK
> 127.0.0.1:6379[2]> get money
> "100"
> 127.0.0.1:6379[2]> set money 1000
> OK
> 127.0.0.1:6379[2]>
> ```
>
> 原等待执行的事务：
>
> ```bash
> 127.0.0.1:6379[2]> set money 100
> OK
> 127.0.0.1:6379[2]> set out 0
> OK
> 127.0.0.1:6379[2]> watch money
> OK
> 127.0.0.1:6379[2]> multi
> OK
> 127.0.0.1:6379[2]> decrby money 20
> QUEUED
> 127.0.0.1:6379[2]> incrby money 20
> QUEUED
> ```
>
> 执行事务结果如下：
>
> ```bash
> 127.0.0.1:6379[2]> watch money
> OK
> 127.0.0.1:6379[2]> multi
> OK
> 127.0.0.1:6379[2]> decrby money 20
> QUEUED
> 127.0.0.1:6379[2]> incrby money 20
> QUEUED
> 127.0.0.1:6379[2]> exec #执行之前另一个线程修改money
> (nil)
> ```
>
> 

**使用watch指令可以充当redis乐观锁** (不存在ABA问题)

