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

## config文件

### 启动

```bash
started with the file path as first argument:
# 启动服务带配置文件

./redis-server /path/to/redis.conf
```



###内存设定

```bash
# 内存大小的unit不区分大小写 可设置k kb m mb g gb
# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

### 包含文件

```bash
# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#- config文件可以包含其他文件，当有一个标准的设置可以适用所有的redis server，
#- 但是需要对单一的 server 单独设置，可以使用include other files.
# 

# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
##- include 不会被 command命令覆盖。
##- 最好把include放在config文件的头部，避免覆盖config

# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```



### modules

#### 注意，后续要学习。

**可以通过loadmodule加载module，实现一些原生redis不能实现的功能**

```bash

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so
```

### NETWORK

#### bind

- bind 如果不写，监听所有可用网络接口的连接

- Bind 加ip地址监听一个或者多个ip地址的链接

- > Warning : 服务器上设置时，直接将redis服务暴露在internet上时，所有的接口暴露在internet十分危险；所以默认时只接受本机的连接：127.0.0.1 ::1

- 

```bash

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 lookback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 ::1
```
#### protected-mode

- 安全设置，防止暴露在internet上时引起安全问题

- 当protected mode 开启时, 如果
  - bind命令绑定了其他的地址
  - 密码没有设置

- only支持本机的connection

- 默认开启

```bash
# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.	
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes
```
#### port
监听端口指令，6379默认

```bash
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
```
#### tcp-backlog

高并发场景下需要backlog，避免client的较慢连接产生的问题

要结合/proc/sys/net/core/somaxconn使用

```bash
# TCP listen() backlog.
#
# In high requests-per-second environments you need an high backlog in order
# to avoid slow clients connections issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511
```
#### unix socket

```bash
# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /tmp/redis.sock
# unixsocketperm 700
```
#### timeout

```bash
# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0
```
#### tcp-keepalive

```bash
# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300
```

### GENERAL

常用设置？

#### daemonize

是否run as a deamon. When yes, 将write a pid file in /usr/local/var/run/redis.pid

default no
```bash
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /usr/local/var/run/redis.pid when daemonized.
daemonize no
```

#### supervised

默认no, 可以通过设置与监督树交互管理redis进程()
```bash
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no
```
#### pidfile

deamon运行时生成pidfile 指令规定生成的文件的位置和名字
```bash
# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/usr/local/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid
```
#### loglevel [debug|verbose|notice|warning]

日志级别

```bash
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
```

#### logfile

给log file命名


```bash
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""
```

#### syslog-enabled

是否将日志记录到system log中

```bash
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no
```
#### syslog-ident

标识log中的名字

```bash
# Specify the syslog identity.
# syslog-ident redis
```
#### syslog-facility

```bash
# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
```

#### databases

数据库的数量设置 默认16


```bash

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```
#### always-show-logo

开启的时候显示logo

```bash
# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY. Basically this means
# that normally a logo is displayed only in interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
always-show-logo yes
```

### snapshotting

数据持久化

#### save

**save  <seconds>  <changes>**

内存中存储 断电即失

设置持久化，保存db到磁盘

```bash
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```
#### stop-writes-on-bgsave-error

持久化开启后，如果保存出错是否让redis继续写入数据到磁盘

```bash
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
```

#### rdbcompression

是否压缩rdb 需要消耗一些cpu资源但是会使得文件更小

```bash
# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes
```
#### rdbchecksum

以一定的效率为代价，保证数据的一致性
关闭校验和时，校验和是0，来告诉代码跳过校验

```bash
# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes
```
#### dbfilename 

持久化数据文件名

```bash
# The filename where to dump the DB
dbfilename dump.rdb
```
#### dir

持久化数据存放目录

```bash
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /usr/local/var/db/redis/
```

### replication

用 slaveof 命令来复制另一个redis server

需要知道的内容：

- redis复制是异步的，但是可以配置一个master主机，如果没有设置数量的从机连接master, 停止接收写入

- 如果复制链路在相对较小的时间内丢失，Redis从机能够与主机进行部分重新同步。可自行配置backlog的size
- 复制是自动的，当从机尝试连接后，自动进行数据的同步



#### slaveof

**Slaveof  <masterip> <masterport>**


```bash
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of slaves.
# 2) Redis slaves are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition slaves automatically try to reconnect to masters
#    and resynchronize with them.
#
# slaveof <masterip> <masterport>
```
#### masterauth

master有密码验证的时候，需要设置masterauth 否则拒绝slave的connection

Masterauth <master-password>

```bash
# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
# masterauth <master-password>
```
#### slave-serve-stale-data
  设置从机失联或者正在同步数据的时候，从机的工作模式
  - yes 仍处理client的请求 数据可能为空 或者未同步的旧数据
  - no 直接不处理，返回error 
```bash
# When a slave loses its connection with the master, or when the replication
# is still in progress, the slave can act in two different ways:
#
# 1) if slave-serve-stale-data is set to 'yes' (the default) the slave will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#
# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO and SLAVEOF.
#
slave-serve-stale-data yes
```
#### slave-read-only
设置从机是否只读，是否写操作
```bash

# You can configure a slave instance to accept writes or not. Writing against
# a slave instance may be useful to store some ephemeral data (because data
# written on a slave will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.

# Since Redis 2.6 by default slaves are read-only.
#
# Note: read only slaves are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only slave exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only slaves using 'rename-command' to shadow all the
# administrative / dangerous commands.
slave-read-only yes

```
#### repl-diskless-sync
默认no
设置复制的策略（磁盘或者socket复制）
> 警告：无磁盘复制现在还属于试验阶段 
- disk-backed redis的master创建一个进程，然后在disk上写文件，然后该文件通过parent进程传给从机
- diskless redis master直接通过socket向 slave传数据 
- 在disk-backed复制，在生成RDB文件的同时，只要当前生成RDB文件的子进程完成工作，就可以将更多的从属程序排入队列并提供RDB文件。
- diskless复制，一旦传输开始，新到达的从机将被排队，当当前的传输终止时，新的传输将开始。
- 当使用无盘复制时，主站在开始传输前会等待一段可配置的时间（以秒为单位），希望有多个从站到达，传输可以并行化。
- 对于慢速磁盘和快速（大带宽）网络，无盘复制效果更好。
```bash
# Replication SYNC strategy: disk or socket.
#
# -------------------------------------------------------
# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
# -------------------------------------------------------
#
# New slaves and reconnecting slaves that are not able to continue the replication
# process just receiving differences, need to do what is called a "full
# synchronization". An RDB file is transmitted from the master to the slaves.
# The transmission can happen in two different ways:
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the slaves incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to slave sockets, without touching the disk at all.
#
# With disk-backed replication, while the RDB file is generated, more slaves
# can be queued and served with the RDB file as soon as the current child producing
# the RDB file finishes its work. With diskless replication instead once
# the transfer starts, new slaves arriving will be queued and a new transfer
# will start when the current one terminates.
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple slaves
# will arrive and the transfer can be parallelized.
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
repl-diskless-sync no
```




```bash
# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the slaves.
#
# This is important since once the transfer starts, it is not possible to serve
# new slaves arriving, that will be queued for the next RDB transfer, so the server
# waits a delay in order to let more slaves arrive.
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
repl-diskless-sync-delay 5

# Slaves send PINGs to server in a predefined interval. It's possible to change
# this interval with the repl_ping_slave_period option. The default value is 10
# seconds.
#
# repl-ping-slave-period 10

# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of slave.
# 2) Master timeout from the point of view of slaves (data, pings).
# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-slave-period otherwise a timeout will be detected
# every time there is low traffic between the master and the slave.
#
# repl-timeout 60

# Disable TCP_NODELAY on the slave socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to slaves. But this can add a delay for
# the data to appear on the slave side, up to 40 milliseconds with
# Linux kernels using a default configuration.
#
# If you select "no" the delay for data to appear on the slave side will
# be reduced but more bandwidth will be used for replication.
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and slaves are many hops away, turning this to "yes" may
# be a good idea.
repl-disable-tcp-nodelay no
```



#### repl-backlog-size

  规定backlog的size
- backlog相当是一个buffer, 当slave失去连接时，累计存储slave data，当重连后，不需要完全做同步，只需要进行一部分数据的一致性同步
- 这个buffer的size个更大，可允许的断开连接的时间就更长
- backlog至少有一个slave连接到master中时才会有backlog
```bash
# Set the replication backlog size. The backlog is a buffer that accumulates# slave data when slaves are disconnected for some time, so that when a slave
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the slave missed while
# disconnected.
#
# The bigger the replication backlog, the longer the time the slave can be
# disconnected and later be able to perform a partial resynchronization.
#
# The backlog is only allocated once there is at least a slave connected.
#
# repl-backlog-size 1mb
```
#### repl-backlog-ttl
- master失去slave的连接后，若干时间后backlog will be freed
- slave 不会有超时时间free backlog的问题；因为slave可能后面被选为master,需要部分一致性同步其他slave的数据

```bash
# After a master has no longer connected slaves for some time, the backlog
# will be freed. The following option configures the amount of seconds that
# need to elapse, starting from the time the last slave disconnected, for
# the backlog buffer to be freed.
#
# Note that slaves never free the backlog for timeout, since they may be
# promoted to masters later, and should be able to correctly "partially
# resynchronize" with the slaves: hence they should always accumulate backlog.
#
# A value of 0 means to never release the backlog.
#
# repl-backlog-ttl 3600
```
#### slave-priority
设置从机的优先级，当master不能正常工作的时候，根据优先级选举master
- 数值越低，优先级越高
- 数值为0 不参与选举
- 默认100
```bash
# The slave priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a slave to promote into a
# master if the master is no longer working correctly.
#
# A slave with a low priority number is considered better for promotion, so
# for instance if there are three slaves with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the slave as not able to perform the
# role of master, so a slave with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
slave-priority 100
```
#### min-slaves-to-write max-lag

- 当 slave 的数量少于 N时，master stop accepting写操作，N个slaves需要在线
- 大于 max lag时，master stop accepting写操作
- lag计算通常通过每秒ping一次
- 将其中一个设置为0 禁用此功能
- 默认slaves-to-write为0 max-lag为10

```bash
# It is possible for a master to stop accepting writes if there are less than
# N slaves connected, having a lag less or equal than M seconds.
# The N slaves need to be in "online" state.
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the slave, that is usually sent every second.
#
# This option does not GUARANTEE that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough slaves
# are available, to the specified number of seconds.
#
# For example to require at least 3 slaves with a lag <= 10 seconds use:
#
# min-slaves-to-write 3
# min-slaves-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.
```
#### slave-announce-ip port

Redis master可以通过一些不同的方式获取slaves的地址和端口

> info replication
>
> role

ip的获取方法：通过检查主从之间的socket的对等地址来获得。

port的获取方法：通过进行数据同步时的握手进行获取

> NAT转发 network address时，可能会是不同的ip and port

```bash
# A Redis master is able to list the address and port of the attached
# slaves in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover slave instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a master.
#
# The listed IP and address normally reported by a slave is obtained
# in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the slave to connect with the master.
#
#   Port: The port is communicated by the slave during the replication
#   handshake, and is normally the port that the slave is using to
#   list for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the slave may be actually reachable via different IP and port
# pairs. The following two options can be used by a slave in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
#
# slave-announce-ip 5.5.5.5
# slave-announce-port 1234

```

### security

#### requirepass

如果需要其他客户端连接redis时auth认证，键入此命令

Requirepass  yourpwd

> Warning: 外部用户可以150k/s 尝试破解密码，需要set very strong password



```bash
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
```
#### rename-command
> 安全起见：把cofig 命令rename : rename-command config xxx

```bash
# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.
```

### clients

#### maxclients

设置最大的客户端的数量；超出数量返回error；

```bash
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000
```



### memory management

#### maxmemory
设置内存使用的内存限制
- 当内存使用达到设置的字节数时，根据maxmemory-policy的设置来remove keys.
- 如果设置的policy = "noevictioon" 或者 不能remove keys, 将会返回errors
- 对于需要占用更多内存的命令（set lpush), 返回错误
- 对于read-only的指令（类似get）, 仍然返回结果
- 将redis作为LRU / LFU cache时此命令有用
> 如果slave 开启此命令，缓冲区的buffer需要满足扣slave已用的内存，因此key被删除的时候不会导致数据不同步的问题，所以slave的设置要相对低，留给系统一定的Free Ram作为 output buffer(如果policy设置为noeviction则不需要设置)

```bash
# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
```
#### maxmemory-policy

当redis memory使用达到设置值，设置移除key的policy

- volatile-lru 只对设置了过期时间的key进行LRU
- Allkeys-lru 删除lru算法的key
- Volatile-lfu 只对设置了过期时间的key进行LFU
- Allkeys-lfu 对所有lfu算法的key进行删除
-  volatile-random 对设置过期时间的key随机删除
- allkeys-random 对所有的key进行随机删除
- volatile-ttl 删除即将过期的key
- noeviction 永不删除，返回内存满的错误

```bash
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction
```
#### maxmemory-samples

内存回收算法不是完全精确的，需要进行采样来检查，此设置影响精度。默认5


```bash


# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. For default Redis will check five keys and pick the one that was
# used less recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
#
# maxmemory-samples 5
```



### LAZY FREEING	

Redis 删除key的命令：

- DEL
  - 对object的阻塞删除。（服务器停止处理新的命令直到删除完毕）
  - Key - value , value是 small object时需要的时间相对短（O(1) 或 O(log_N)）
  - Value包含数百万个元素的聚合值，server可能会阻塞很久（甚至几秒钟）来处理del
- UNLINK (非阻塞删除） 以及FLUSHDB FLUSHALL（异步）
  - 后台多开一个thread来进行内存释放

以上命令均为用户控制的指令来进行内存释放，有时候还有一些其他场景，由redis自行释放一定的内存空间：

- eviction，内存回收机制，设置了最大使用内存和内存回收的方式后，有时候需要释放内存为新的data腾出空间
- key设置了过期时间，过期时自动由redis进行释放
- 在存储新的data时，可能会发生覆盖的情况，覆盖的操作实际上是先删除原数据，再重写新数据的过程。
- 主从复制时，发生全部的文件复制时，复制完成后删除slave原来的数据，将最新的db file加载。

```bash
# Redis has two primitives to delete keys. One is called DEL and is a blocking
# deletion of the object. It means that the server stops processing new commands
# in order to reclaim all the memory associated with an object in a synchronous
# way. If the key deleted is associated with a small object, the time needed
# in order to execute the DEL command is very small and comparable to most other
# O(1) or O(log_N) commands in Redis. However if the key is associated with an
# aggregated value containing millions of elements, the server can block for
# a long time (even seconds) in order to complete the operation.
#
# For the above reasons Redis also offers non blocking deletion primitives
# such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and
# FLUSHDB commands, in order to reclaim memory in background. Those commands
# are executed in constant time. Another thread will incrementally free the
# object in the background as fast as possible.
#
# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.
# It's up to the design of the application to understand when it is a good
# idea to use one or the other. However the Redis server sometimes has to
# delete keys or flush the whole database as a side effect of other operations.
# Specifically Redis deletes objects independently of a user call in the
# following scenarios:
#
# 1) On eviction, because of the maxmemory and maxmemory policy configurations,
#    in order to make room for new data, without going over the specified
#    memory limit.
# 2) Because of expire: when a key with an associated time to live (see the
#    EXPIRE command) must be deleted from memory.
# 3) Because of a side effect of a command that stores data on a key that may
#    already exist. For example the RENAME command may delete the old key
#    content when it is replaced with another one. Similarly SUNIONSTORE
#    or SORT with STORE option may delete existing keys. The SET command
#    itself removes any old content of the specified key in order to replace
#    it with the specified string.
# 4) During replication, when a slave performs a full resynchronization with
#    its master, the content of the whole database is removed in order to
#    load the RDB file just transfered.
#
# In all the above cases the default is to delete objects in a blocking way,
# like if DEL was called. However you can configure each case specifically
# in order to instead release memory in a non-blocking way like if UNLINK
# was called, using the following configuration directives:

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
```

###  APPEND ONLY MODE

aof （另外一种数据持久化方案）

默认的方式将数据持久化到磁盘中是一个不错的方案，当服务器突然断电可能引发几分钟的写入数据丢失（这取决于配置的数据保存节点）

AOF(append only file)是一种替代的持久化方案，提供更好的持久性。

- AOF 使用的默认的数据fsync policy,可以使服务器发生断电情况下，仅损失一秒的写入数据；或者是redis process本身出错，操作系统仍能正常工作时，仅丢失最后一次的写入。
- RDB 和 AOF可以共同开启。如果AOF在启动时被启用，Redis将加载AOF(具有更好的持久性保证的文件)。



```bash
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
```
#### appendonly

是否启用 默认 no
```bash
# Please check http://redis.io/topics/persistence for more information.

appendonly no

```
#### appendfilename
文件名 of appendfile
```bash
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```
#### appendfsync

设置fsync的policy，三种可选：always everysec no

```bash
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no
```

#### no-appendfsync-on-rewrite

当fsync policy 是everysec 或者always的时候，一个后台用于保存或者日志重写的process将会执行大量的I/O操作，redis会在调用fsync造成阻塞；目前没有解决这个问题。

为了缓解此问题，当在 bgsave or bgerwriteaof 在调用时，主进程禁止调用fsync.(这会有时候损失30秒的日志数据)

最好是将此配置设置为no， 除非有一些延迟上的问题需要将此配置改为yes

```bash


# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no
```
#### 

#### 自动重写aof

auto-aof-rewrite-percentage

auto-aof-rewrite-min-size

当aof的log size增长到一定的%，redis可以自动的重写log（通过隐式调用bgrewriteaof)

- Redis会记住最近一次重写后AOF文件的大小（如果重启后没有发生重写，则使用启动时AOF的大小）
- 这个文件大小作为base size与当前aof进行比较。
- 如果当前aof的size大于指定的百分比，触发重写
- 另外，还需要为要重写的AOF文件指定一个最小的size，这对于避免重写AOF文件是很有用(有时候即使达到了增加的百分比，但aof仍然相当小）。

```bash
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

#### aof-load-truncated

- 在Redis启动过程中，当AOF数据被加载回内存时，可能会发现AOF文件在末端被截断了。这可能发生在Redis运行的系统崩溃时，特别是当ext4文件系统被挂载而没有data=ordered选项时（然而这不会发生在Redis本身崩溃或中止但操作系统仍然正常工作时）。
- 当这种情况发生时，Redis可以以错误退出或者尽可能多地加载数据（现在的默认值），如果发现AOF文件在末尾被截断，则启动。下面的选项可以控制这种行为。
  - 如果aof-load-truncated被设置为yes，一个被截断的AOF文件被加载，Redis服务器开始发出一个日志来通知用户这个事件。
  - 如果该选项被设置为no，服务器会以一个错误中止并拒绝启动。用户需要在重启服务器之前使用 "redis-check-aof "工具修复AOF文件。
- 如果aof文件在中间发生错误，redis仍退出并返回error（这种情况仅会在redis尝试读取aof更多的数据但是没找到足够的字节数时发生）

```bash
# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
# 
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes
```
#### aof-use-rdb-preamble

当重写AOF文件时，Redis能够在AOF文件中使用RDB前言，以加快重写和恢复。当这个选项被打开时，重写的AOF文件由两个不同的句子组成。当加载Redis的时候，识别到AOF文件以 "REDIS "字符串开始，并加载前缀的RDB文件，并继续加载AOF尾部。
目前默认情况下这是关闭的，以避免意外的 的格式变化，但在某些时候会被作为默认值。

```bash
# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, and continues loading the AOF
# tail.
#
# This is currently turned off by default in order to avoid the surprise
# of a format change, but will at some point be used as the default.
aof-use-rdb-preamble no
```

### LUA SCRIPTING

#### lua-time-limit

单位 milliseconds

lua脚本超过了设置的时间仍在执行将会在log中记录返回error

执行的脚本超过了设定的时间，可通过script kill and shutdown nosave来停止。第一个用来停止一个还没有调用的脚本；第二条命令是在脚本已经发出写命令但用户不想等待脚本自然终止的情况下关闭服务器的唯一方法。



```bash
# Max execution time of a Lua script in milliseconds.
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
lua-time-limit 5000
```



### REDIS CLUSTER

#### cluster-enabled

> redis cluster被认为是稳定的，但是需要一些user部署到生产环境中验证

普通的 Redis 实例不能成为 Redis 集群的一部分；只有作为cluster node启动的node可以。为了将 Redis 实例作为cluster node启动，启用集群支持 需要设置cluster-enabled yes

```bash
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes
```
#### cluster-config-file

涉及config-file的名字，这个文件直接被创建和更新，每个cluster node需要一个不同的名字

- 需要确认所有的instances在相同的系统中运行

```bash
# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
#
# cluster-config-file nodes-6379.conf
```
#### cluster-node-timeout

单位milliseconds

ttl设置，超时认为node in failure state.

```bash
# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
#
# cluster-node-timeout 15000
```
cluster-config-file

一个出错的master的slave的data如果看起来太旧，则需要避免启动故障转移；没有一个简单的方式来衡量一个slave的data是否too old,

因此需要如下两种check:

1) 如果有多台slave可以做failover, 相互之间交换信息来确定有best replication offset的slave获得优势

Slave将尝试通过偏移量获得评级，并对故障转移的开始 应用与其评级成比例的延迟。

2）每一个slave都要计算与master最后一次交互的time。这可以是最后一次收到的ping或命令（如果master仍然处于 "连接 "状态），或者是与master断开连接后所经过的时间（如果复制链路目前是关闭的）。 如果最后的交互时间太长，slave不会尝试故障转移。

- 用户可以对第二点进行调整，如果自上次与master互动以来，经过的时间大于time_a，则从站将不执行故障转移:
- Time_a = (node-timeout * slave-validity-factor) + repl-ping-slave-period

```bash
# A slave of a failing master will avoid to start a failover if its data
# looks too old.
#
# There is no simple way for a slave to actually have an exact measure of
# its "data age", so the following two checks are performed:
#
# 1) If there are multiple slaves able to failover, they exchange messages
#    in order to try to give an advantage to the slave with the best
#    replication offset (more data from the master processed).
#    Slaves will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#
# 2) Every single slave computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the slave will not try to failover
#    at all.
#
# The point "2" can be tuned by user. Specifically a slave will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
#
#   (node-timeout * slave-validity-factor) + repl-ping-slave-period
#
# So for example if node-timeout is 30 seconds, and the slave-validity-factor
# is 10, and assuming a default repl-ping-slave-period of 10 seconds, the
# slave will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
```

Slave-validity-factor

该参数较大可能选出一个data特别旧的slave进行failover, 较小的值可能使cluster中找不到合适的slave来做failover

为了获得最大的可用性，可以将slave-validity-factor设置为0，这意味着，无论最后一次与master交互是什么时候，slave都会尝试故障转移master。(然而，它们总是试图应用一个与它们的偏移等级成比例的延迟）。
0是唯一能够保证当所有分区愈合时，集群总是能够继续下去的值。

```bash

#
# A large slave-validity-factor may allow slaves with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a slave at all.
#
# For maximum availability, it is possible to set the slave-validity-factor
# to a value of 0, which means, that slaves will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
#
# cluster-slave-validity-factor 10
```

cluster slaves机能够迁移到无主的master(没有在工作中的slave的master)上。这提高了集群抵御故障的能力，因为如果没有工作的slave，在故障情况下，无主的master就不能被转移。

- migration barrier:只有当旧master至少还有一定数量的其他在工作中的slave时，slaves才会迁移到新的orphaned master。
- migration barrier = 1意味着，只有在至少有1个其他工作的master的slave时，slaves才会迁移，以此类推。
- 它通常反映了你希望集群中每个master都有多少个slave。
- 默认设置为1，当想要disable的时候设置一个特别大的数，设置为0的话只在debugging使用，在production使用是危险的

```bash
# Cluster slaves are able to migrate to orphaned masters, that are masters
# that are left without working slaves. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working slaves.
#
# Slaves migrate to orphaned masters only if there are still at least a
# given number of other working slaves for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a slave
# will migrate only if there is at least 1 other working slave for its master
# and so forth. It usually reflects the number of slaves you want for every
# master in your cluster.
#
# Default is 1 (slaves migrate only if their masters remain with at least
# one slave). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
#
# cluster-migration-barrier 1
```
#### cluster-require-full-coverage

默认情况下，如果Redis集群的nodes检测到至少有一个hash slot未被覆盖（没有可用的node为其服务），则停止接受查询。

这样，如果cluster部分停机（例如，一部分的hash slot 不再被覆盖），所有的cluster最终会变得不可用。一旦所有的slots被覆盖，它就会自动恢复可用。
然而，有时你想让正在工作的cluster的子集继续接受对仍被覆盖的key的控件进行部分查询。为了做到这一点，只需将cluster-require-full-coverage选项设置为no。

```bash
# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#
# cluster-require-full-coverage yes
```
#### cluster-slave-no-failover

设置为 "yes "时，可以防止slave在master发生故障时尝试故障转移其master。然而，master仍然可以执行手动故障转移，如果被迫这样做的话。
这在不同的情况下是很有用的，特别是在多个数据中心运行的情况下，我们希望一方永远不会被提升，如果不是 在DC完全失败的情况下。

```bash
# This option, when set to yes, prevents slaves from trying to failover its
# master during master failures. However the master can still perform a
# manual failover, if forced to do so.
#
# This is useful in different scenarios, especially in the case of multiple
# data center operations, where we want one side to never be promoted if not
# in the case of a total DC failure.
#
# cluster-slave-no-failover no

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.
```

### CLUSTER DOCKER/NAT support

- 容器或其他容器方式部署cluster的时候，Redis Cluster节点的地址发现失败，因为地址被NAT屏蔽或端口被转发

- 为了redis cluster可以在这种环境中使用，需要进行一些静态配置，让每个node知道其public address：

  - cluster-announce-ip
  - cluster-announce-port
  - cluster-announce-bus-port

  > 每个配置项分别指示node的地址、客户端口和集群消息总线端口
  >
  > 然后，这些信息被公布在总线数据包的头部，以便其他node能够正确映射公布信息的node的地址。
  > 如果不使用上述选项，将使用正常的Redis集群自动检测来代替。

- 请注意，当重新映射时，总线端口可能不在client port + 10000的固定偏移量上，所以你可以指定任何端口和总线端口，取决于它们如何被重新映射。如果没有设置总线端口，通常会使用10000的固定偏移。

```bash
# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node knows its public address is needed. The
# following two options are used for this scope, and are:
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# Each instruct the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead.
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usually.
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380
```



### SLOW LOG

Redis慢速日志是一个记录超过指定执行时间的查询的系统。执行时间不包括I/O操作，如与客户端对话、发送回复等，而只是实际执行命令所需的时间（这是命令执行的唯一阶段，线程被阻塞，在此期间不能为其他请求服务）。

可以用两个参数来配置慢速日志：一个是告诉Redis超过多少执行时间（以微秒为单位），命令才会被记录下来，另一个参数是慢速日志的长度。当一个新的命令被记录下来时，最旧的命令会从 的队列中删除。



#### slowlog-log-slower-than

- 时间是以微秒为单位的，所以1000000相当于相当于一秒钟
- 请注意，一个负数会禁用慢速日志
- 而数值为零则强制记录每一条命令

#### slowlog-max-len

- 这个长度没有限制。只是要注意它将消耗内存。可以用SLOWLOG RESET来回收慢速日志所使用的内存。

```bash
# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.

# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128
```

#### LATENCY MONITOR

延迟监测

- Redis延迟监控子系统在运行时对不同的操作进行采样，以收集与Redis实例的可能延迟来源有关的数据。
  通过 LATENCY 命令，用户可以获得这些信息，可以打印图表和获得报告。
- 系统只记录那些在等于或大于通过latency-monitor-threshold配置指令指定的毫秒量的时间内执行的操作。当它的值被设置为0时，延迟监控被关闭。
- 默认情况下，延迟监控是禁用的，因为如果你没有延迟问题，它大多是不需要的，而且收集数据对性能有影响，虽然非常小，但可以在大负载下测量。
- 延迟监测可以很容易地在运行时使用以下命令来启用 "CONFIG SET latency-monitor-threshold <milliseconds>"，如果需要的话，可以很容易地在运行时启用。

#### latency-monitor-threshold

```bash
# The Redis latency monitoring subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports.
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
latency-monitor-threshold 0
```



### EVENT NOTIFICATION

Redis可以通知Pub/Sub客户端在key 空间中发生的事件。该功能在http://redis.io/topics/notifications。

```bash
# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
# 如果keyspace事件通知被启用，一个client对存储在database 0中的key "foo "执行了DEL操作，两个消息将通过Pub/Sub发布:
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
# 可以在一组类中选择Redis将通知的事件。每个类都由一个字符来标识。
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events ""

```

#### notify-keyspace-events

- 默认禁用，根据需要进行发布订阅的配置。

### ADVANCED CONFIG

高级配置

### ACTIVE DEFRAGMENTATION

## 数据持久化

数据持久化：在指定的时间间隔中，将内存的数据快照写入到disk，恢复时将快照文件读入内存。

snapshotting, 持久化的有两种: RDB AOF

### RDB

数据持久化关于RDB的相关配置在snapshotting中。

- redis开启一个后台的子进程来进行持久化操作

- 先将内存中的数据写入到临时RDB文件中
- 快照写入完成后，临时RDB文件替换上次的持久化RDB文件
- 子进程退出

整个过程中，主进程不进行I/O操作，保证了性能。

问题：

- 性能相对影响不大，如果需要大规模的数据恢复，且对数据的完整性不是十分敏感，RDB相比于AOF更加高效
- 最后一次持久化到出现问题的时间两个时间差内的数据可能会发生丢失。（这段时间的数据没有备份）
- fork新的后台进程时，占用内存空间

说明：

- 默认情况下就是用RDB,一般情况不需要进行修改。

> RDB持久化的触发机制：
>
> - 配置文件中设置的save规则触发时：
>   - 例如：save 50 4 表示50s内如果有大于等于4次的key的保存操作，则会触发生成RDB文件
>
> - 执行flushall
>
> - 退出redis时



RDB恢复RDB文件：

- 将.rdb文件放在启动目录下，redis启动时将会自动读取.rdb中的文件到内存。

```bash
127.0.0.1:6379[2]> config get dir
1) "dir"
2) "/usr/local/var/db/redis"
```

### AOF

append only file

把所有的命令都记录一遍，恢复的时候就把这个文件中的命令全部都执行一遍来达到持久化的目的。

- 以日志的形式把所有的写操作记录下来，记录所有写操作到log中
- 只追加文件但是不改写文件
- redis重启后根据这个文件顺序执行指令，恢复数据。
- 保存的文件名一般是：appendonly.aof (文件名配置文件中可以修改)

默认不开启，如需使用手动配置为yes

自动重写append only file， append only file一直append下去很大，所以配置了一些参数自动重写aof.

（回顾相关配置查看说明auto-aof-rewrite-percentage auto-aof-rewrite-min-size）

- 如果aof文件有截断，会发生一些错误导致redis不能启动，具体配置见aof-load-truncated（**可以用redis-check-aof修复	**）



优点：

- 文件完整性高

缺点：

- I/O操作较多影响性能，每次写操作都要记录日志到aof中
- 数据量大的时候修复起来很慢

## Redis 主从复制

### 概念

将一台Redis服务器的数据复制到其他Redis服务器。前者为Master，后者为salve;数据复制为单向，由master node复制到slave node；master主要进行写操作，slave主要进行读操作。

主从复制的作用：

1. 数据冗余：主从复制实现数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：master出现问题可以由slave node提供服务，实现快速的故障恢复；实际上市一种服务的冗余。
3. 负载均衡：主从复制的基础上配合读写分离，分担服务器的负载；尤其是在写少读多的场景下，提高并发量
4. 高可用的基础：主从复制是哨兵和集群能够实施的基础。

### 配置

- redis默认情况下为master
- 配置slave的conf
- 配置多个slave.conf （更改端口，ip， pid，rdb文件，log等名字）
- 其他请见replication部分的配置。

查看当前库的信息

info replication

```bash
127.0.0.1:6379[2]> info replication
# Replication
role:master
connected_slaves:0
master_replid:7e84cb61e94e6b7347ba8095f10a548e4f1a6da9
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

127.0.0.1:16371> get k2
"v2"
127.0.0.1:16371> set k1
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:16371> set k1 v2
(error) READONLY You can't write against a read only slave.
127.0.0.1:16371>  info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:739
slave_priority:20
slave_read_only:1
connected_slaves:0
master_replid:a8f051f1fd9176c0f11d21d5141f435a25401ca1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:739
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:739
127.0.0.1:16371>

127.0.0.1:16372> get k1
"v1"
127.0.0.1:16372> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:725
slave_priority:40
slave_read_only:1
connected_slaves:0
master_replid:a8f051f1fd9176c0f11d21d5141f435a25401ca1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:725
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:725
127.0.0.1:16372>
```

**目前还没有哨兵模式，当master宕机后，slave不会被选举为master**



> 复制原理
>
> slave 启动成功连接到master后会发送一个sync同步命令
>
> master接到命令，启动后台的存盘进程，同时收集所有接收到的修改命令，后台进程执行完毕后，**master将整个数据文件传到slave，并完成一次完全同步**

**全量复制**：slave接收到master的数据库文件后，将其存盘并且加载到内存中

**增量复制**：master继续将新的命令一次传给slave,完成同步

## 哨兵模式

redis master server出错后，需要将其中一台slave切换为master,哨名模式开一个独立的进程监控，自动进行切换。

redis提供了邵明的命令，哨兵发送数据等待redis服务器的响应情况来监控多个redis服务的运行状态。

1. 配置文件：redis-sentinel.conf
   1. 本配置文件安装redis默认会在配置文件目录中。
2. 哨兵的配置通常配置奇数台数量的哨兵，防止哨兵宕机以及选举平票
3. 哨兵推举master的算法（后面看）

```bash
# Example sentinel.conf

# *** IMPORTANT ***
#
# By default Sentinel will not be reachable from interfaces different than
# localhost, either use the 'bind' directive to bind to a list of network
# interfaces, or disable protected mode with "protected-mode no" by
# adding it to this configuration file.
# 默认情况下，如果不使用bind命令或者使用protected-mode来关闭保护模式，sentinel只能在本机访问。
# Before doing that MAKE SURE the instance is protected from the outside
# world via firewalling or other means.
# 如果要开启外部访问，请确保外部访问通过防火墙或者其他服务后是安全的
# For example you may use one of the following:
#
# bind 127.0.0.1 192.168.1.1
#
# protected-mode no

# port <sentinel-port>
# The port that this sentinel instance will run on
# 开启的实例监听的端口号
port 26379

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# The above two configuration directives are useful in environments where,
# because of NAT, Sentinel is reachable from outside via a non-local address.
# announce-ip port两个指令在NAT的环境中使用，似的sentinel可以从外部的一个非本地地址进行访问

# When announce-ip is provided, the Sentinel will claim the specified IP address
# in HELLO messages used to gossip its presence, instead of auto-detecting the
# local address as it usually does.
# 当提供announce-ip时，哨兵将在HELLO消息中宣称指定的IP地址，而不是像通常那样自动侦测本地地址。
# Similarly when announce-port is provided and is valid and non-zero, Sentinel
# will announce the specified TCP port.
# 同样，当announce-port被提供并且是有效的和非零时，哨兵将宣布指定的TCP端口。
# The two options don't need to be used together, if only announce-ip is
# provided, the Sentinel will announce the specified IP and the server port
# as specified by the "port" option. If only announce-port is provided, the
# Sentinel will announce the auto-detected local IP and the specified port.
#这两个选项不需要一起使用，如果只提供announce-ip，哨兵将公布指定的IP和 "port "选项所指定的服务器端口。如果只提供announce-port，那么 哨兵将公布自动检测到的本地IP和指定的端口。
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# Every long running process should have a well-defined working directory.
# 每个长期运行的进程都应该有一个定义明确的工作目录。
# For Redis Sentinel to chdir to /tmp at startup is the simplest thing
# for the process to don't interfere with administrative tasks such as
# unmounting filesystems.
# 对于 Redis Sentinel 来说，在启动时 chdir 到 /tmp 是最简单的事情，因为该进程不会干扰管理任务，如卸载文件系统。
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
# 
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
# 告诉哨兵监控这个主站，只有在至少<quorum>的哨兵同意的情况下，才会认为它处于O_DOWN（客观下降）状态。就是说需要最少要的哨兵数量选举slave,进行slave->master的变更，进行failover
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
# 请注意，无论ODOWN的法定人数是多少，哨兵都需要被大多数已知的哨兵选中才能开始故障转移，所以在少数情况下不能进行故障转移。
# Slaves are auto-discovered, so you don't need to specify slaves in
# any way. Sentinel itself will rewrite this configuration file adding
# the slaves using additional configuration options.
# Also note that the configuration file is rewritten when a
# slave is promoted to master.
# slave是自动发现的，所以你不需要以任何方式指定slave。Sentinel本身会重写这个配置文件，使用额外的配置选项添加从属设备。
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
# 还要注意的是，当一个从站被提升为主站时，该配置文件会被重写。
# 注意：主站名称不应包括特殊字符或空格。
# 有效的字符集是A-z 0-9和三个字符".-_"。
sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel auth-pass <master-name> <password>
# 主机设置了密码访问的，需要进行此配置
# Set the password to use to authenticate with the master and slaves.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for slaves, so it is not
# possible to set a different password in masters and slaves instances
# if you want to be able to monitor these instances with Sentinel.
# 注意：这个密码也用于slaves,所以如果想用sentinel监控，master 和 slave的 auth 密码需要相同
# However you can have Redis instances without the authentication enabled
# mixed with Redis instances requiring the authentication (as long as the
# password set is the same for all the instances requiring the password) as
# the AUTH command will have no effect in Redis instances with authentication
# switched off.
# 然而，你可以将没有启用认证的Redis实例与需要认证的Redis实例混在一起（只要对所有需要密码的实例设置相同的密码），因为AUTH命令在认证关闭的Redis实例中没有效果 关闭的 Redis 实例中，AUTH 命令将不起作用。
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached slave or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
# master(或任何附属的slave或哨兵)要在指定的时间内无法连接(例如，不能接受对PING的回复，连续的)，才能认为它处于S_DOWN状态(Subjectively Down）。)
# Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numslaves>
#
# How many slaves we can reconfigure to point to the new slave simultaneously
# during the failover. Use a low number if you use the slaves to serve query
# to avoid that all the slaves will be unreachable at about the same
# time while performing the synchronization with the master.
# 在故障切换期间，我们可以重新配置多少个slave，使其同时指向新的slave。如果你使用从站提供查询服务，请使用一个较低的数字，以避免所有的从站在与主站进行同步时，在差不多的时间内无法到达。在执行与主站的同步时，所有的从站将无法到达。
sentinel parallel-syncs mymaster 1
# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
# 定义failover的超时时间（milliseconds）
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
# - 在一个给定的哨兵对同一主站已经尝试过一次故障转移后，重新启动故障转移所需的时间是故障转移超时的2倍。

# - The time needed for a slave replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
# - 根据Sentinel当前配置复制到错误的主站的从站被强制复制到正确的主站所需的时间正是故障转移超时（从Sentinel检测到错误配置的时刻开始计算）。
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted slave).
# - 取消已经在进行中但没有产生任何配置变化的故障转移所需的时间（SLAVEOF NO ONE还没有被晋升的从属设备确认）。
# - The maximum time a failover in progress waits for all the slaves to be
#   reconfigured as slaves of the new master. However even after this time
#   the slaves will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
# - 正在进行的故障切换等待所有slaves被重新配置为新master的slave的最长时间。
# - 然而，即使在这个时间之后，slaves还是会被Sentinels重新配置，但不是按照指定的确切的平行同步进度。
# Default is 3 minutes.
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
# sentinel notification-script和sentinel reconfig-script是用来配置脚本的，这些脚本被调用来通知系统管理员或在故障切换后重新配置客户端。这些脚本在执行时有以下错误处理规则。
# 如果脚本以 "1 "退出，以后会重新执行（目前设定的最大次数为10）。
# 如果脚本以 "1 "退出，将在稍后重试（目前设定的最大次数为10）。目前设定为10次）。)
# 如果脚本以 "2"（或更高的值）退出，脚本的执行就不会重试。
# 如果脚本因为收到一个信号而终止，其行为与退出代码1相同。
# 一个脚本的最大运行时间为60秒。在达到这个极限之后 脚本会被SIGKILL终止，并重新执行。

# sentinel notification-script and sentinel reconfig-script are used in order
# to configure scripts that are called to notify the system administrator
# or to reconfigure clients after a failover. The scripts are executed
# with the following rules for error handling:
#
# If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
## If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
#
# If script exits with "2" (or an higher value) the script execution is
# not retried.
#
# If script terminates because it receives a signal the behavior is the same
# as exit code 1.
#
# A script has a maximum running time of 60 seconds. After this limit is
# reached the script is terminated with a SIGKILL and the execution retried.

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
#
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# CLIENTS RECONFIGURATION SCRIPT
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# When the master changed because of a failover a script can be called in
# order to perform application-specific tasks to notify the clients that the
# configuration has changed and the master is at a different address.
#
# The following arguments are passed to the script:
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> is currently always "failover"
# <role> is either "leader" or "observer"
#
# The arguments from-ip, from-port, to-ip, to-port are used to communicate
# the old address of the master and the new address of the elected slave
# (now a master).
#
# This script should be resistant to multiple invocations.
#
# Example:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh


```

### 启动哨兵

```bash
./redis-sentinel /usr/local/etc/redis-sentinel.conf
54480:X 21 Jul 14:58:48.270 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
54480:X 21 Jul 14:58:48.270 # Redis version=4.0.10, bits=64, commit=00000000, modified=0, pid=54480, just started
54480:X 21 Jul 14:58:48.270 # Configuration loaded
54480:X 21 Jul 14:58:48.272 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 4.0.10 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 54480
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

54480:X 21 Jul 14:58:48.274 # Sentinel ID is 6385b361f2e44aac9324feef210f0d4963e20beb
54480:X 21 Jul 14:58:48.274 # +monitor master mymaster 127.0.0.1 6379 quorum 2
54480:X 21 Jul 14:58:48.276 * +slave slave 127.0.0.1:16372 127.0.0.1 16372 @ mymaster 127.0.0.1 6379
54480:X 21 Jul 14:58:48.276 * +slave slave 127.0.0.1:16371 127.0.0.1 16371 @ mymaster 127.0.0.1 6379

```

###  master宕机

宕机后重启，变为slave

```bash
127.0.0.1:6379> shutdown
not connected> exit

54480:X 21 Jul 15:04:07.101 # +sdown master mymaster 127.0.0.1 6379
```



```bash
127.0.0.1:16371> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=16372,state=online,offset=9806,lag=0
master_replid:46b57105038f43a876ffa3fff6b298e2911b778d
master_replid2:ad3f04b7f5f831dbd957a7bba38dd1b942e2242c
master_repl_offset:9806
second_repl_offset:8551
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:9806

55023:X 21 Jul 15:09:16.178 # +monitor master mymaster 127.0.0.1 6379 quorum 1
55023:X 21 Jul 15:09:56.626 # +sdown master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.626 # +odown master mymaster 127.0.0.1 6379 #quorum 1/1
55023:X 21 Jul 15:09:56.626 # +new-epoch 1
55023:X 21 Jul 15:09:56.626 # +try-failover master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.632 # +vote-for-leader 6385b361f2e44aac9324feef210f0d4963e20beb 1
55023:X 21 Jul 15:09:56.632 # +elected-leader master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.632 # +failover-state-select-slave master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.727 # +selected-slave slave 127.0.0.1:16371 127.0.0.1 16371 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.727 * +failover-state-send-slaveof-noone slave 127.0.0.1:16371 127.0.0.1 16371 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:56.803 * +failover-state-wait-promotion slave 127.0.0.1:16371 127.0.0.1 16371 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:57.189 # +promoted-slave slave 127.0.0.1:16371 127.0.0.1 16371 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:57.189 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:57.239 * +slave-reconf-sent slave 127.0.0.1:16372 127.0.0.1 16372 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:58.201 * +slave-reconf-inprog slave 127.0.0.1:16372 127.0.0.1 16372 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:59.217 * +slave-reconf-done slave 127.0.0.1:16372 127.0.0.1 16372 @ mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:59.321 # +failover-end master mymaster 127.0.0.1 6379
55023:X 21 Jul 15:09:59.321 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 16371
55023:X 21 Jul 15:09:59.321 * +slave slave 127.0.0.1:16372 127.0.0.1 16372 @ mymaster 127.0.0.1 16371
55023:X 21 Jul 15:09:59.321 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 16371
55023:X 21 Jul 15:10:29.381 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 16371
```

> 哨兵模式优点：
>
> - 哨兵集群，基于主从复制模式，所有的主从配置的有点它都有
> - 主从可以切换，故障可以转移，系统可用性更好
> - 是主从模式的升级，master shutdown后，自动切换slave为master

> 缺点：
>
> - redis不好在线看扩容，集群的容量一旦到达上限，在线扩容就十分麻烦。
> - 实现哨兵模式的配置麻烦，里面有很多选择

## 缓存穿透 雪崩



### 缓存穿透

#### 概念

client查询一个数据，redis作为缓存数据，查找时redis内没有该数据，于是直接向持久层数据库进行查询，发现持久层数据库中也没有，本次查询失败；当用户很多的时候，都去直接请求持久层数据库，造成持久层数据库压力变大。

#### 解决方案

##### 布隆过滤器

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储。在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力

##### 缓存空对象

当存储层不命中后，即使返回一个空对象也将其缓存到redis，同时设计一个过期时间，之后访问就会从缓存中读取空对象。

问题：

- 占用内存（很多空对象都要存储到redis中）
- 数据一致性问题

### 缓存击穿

#### 概述

一个key非常热点，高并发集中的对此热点进行访问，这个key在过期的一瞬间，持续的大量查询请求就直接穿过缓存去请求数据库，导致数据库瞬间压力变大

#### 解决方案

- 设置热点key永不过期
- 加互斥锁
  - 分布式锁，保证每个key只有一个线程去查询后端服务，其他线程等待。
  - 这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大

### 缓存雪崩

在某一个时间段，缓存集中过期失效。

比如某些key的缓存时间统一在一个时间过期（为应对12点的秒杀，11点将缓存写入redis，设置的有效期为2小时，13点时缓存到期但是仍有大量的请求，过期的一瞬间对服务器的数据层产生较大的压力。

另一种情况就是缓存服务器宕机或者断网，会对数据库层造成极大的压力。

#### 解决方案

##### redis高可用

集群操作，保证redis能正常运行

##### 限流降级

##### 数据预热

热点数据预先访问一遍，加载到缓存中，并且设置不同的过期时间。

