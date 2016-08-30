---
title: redis-hash使用
date: 2016-08-30 17:50:32
tags:
- redis
categories:
- 架构/软件
---
#### 一、推荐阅读
1. [http://stackoverflow.com/questions/16375188/redis-strings-vs-redis-hashes-to-represent-json-efficiency](http://stackoverflow.com/questions/16375188/redis-strings-vs-redis-hashes-to-represent-json-efficiency)
2. [http://instagram-engineering.tumblr.com/post/12202313862/storing-hundreds-of-millions-of-simple-key-value](http://instagram-engineering.tumblr.com/post/12202313862/storing-hundreds-of-millions-of-simple-key-value)
3. [http://www.wzxue.com/redis核心解读-类型系统解构/](http://www.wzxue.com/redis%E6%A0%B8%E5%BF%83%E8%A7%A3%E8%AF%BB-%E7%B1%BB%E5%9E%8B%E7%B3%BB%E7%BB%9F%E8%A7%A3%E6%9E%84/)

#### 二、文章1 重点摘要
有多种方式，可以将一个对象数组存储于redis
1.将单个对象JSON ENCODE，然后以string的方式存储。如:
```shell
INCR id:users //redis key "id:users"存储了用户数量
SET user:1 '{"name":"Fred","age":25}' //json encode后以user:id为key存储
SADD users 1 //将用户id添加到redis set中
```
通常这是最优的一个方法，但是不适用于仅仅需对象中个别field的场景。
优点：每个对象有独立的key，json解析速度足够快，特别适用于当需要使用对象中大多数field的场景。
缺点：不适用于仅仅需对象中个别field的场景，比较慢，相应的应该使用方式2

2.将对象存储于hash，类似db table
```shell
INCR id:users //redis key "id:users"存储了用户数量
HMSET user:1 name "Fred" age 25
SADD users 1 //将用户id添加到redis set中
```
这也是一个好的方法。
优点：每个对象有独立的key，特别适用于仅仅需对象中个别field的场景。
缺点：不适用于大多数field的场景，比较慢，相应的应该使用方式1

3.将单个对象JSON ENCODE，以用户id为field，json数组作为值，存储于hash中. 如：
```shell
INCR id:users //redis key "id:users"存储了用户数量
HMSET users 1 '{"name":"Fred","age":25}' //以用户id为field，json数组作为值，存储于hash中
```
这并不是一个好的方法
优点：减少了key的数量，将数据都整合在hash的field-value对中
缺点：a.显然的缺点，是无法设置TTL；
	b.当对象数量比较大（大于hash-max-ziplist-entries配置参数的值）时，占用的内存空间和方式1基本一样。

4.以对象的各个field为key存储
```shell
INCR id:users
SET user:1:name "Fred"
SET user:1:age 25
SADD users 1
```
这并不是一个好的方法，key过多，对key的命名造成污染；占用内存大；不应该被选择，除非一个对象中的各个field需要不同的TTL

方法1,2非常的相似，看场景选择。方法3,除非你非常在意redis中key的数量，且不需要TTL

#### 三、文章2 重点摘要
其服务器hash-max-ziplist-entries参数的值为1000，意思也就是说当hash中field的个数小于1000时以REDIS_ENCODING_ZIPLIST存储，否则使用REDIS_ENCODING_HT存储
```shell
HSET "mediabucket:1156" "1155316" "939"
HSET "mediabucket:1155" "1155315" "935"
```
1155316为ID，939为ID对应的某个值，1156 (1155316 / 1000 = 1156):
```shell
HGET "mediabucket:1156" "1155316"
> "939"
```
#### 四、文章3 重点摘要
Redis类型物理存储解构
\#define REDIS_ENCODING_RAW 0     /\* Raw representation \*/
\#define REDIS_ENCODING_INT 1     /\* Encoded as integer \*/
\#define REDIS_ENCODING_HT 2      /\* Encoded as hash table \*/
\#define REDIS_ENCODING_ZIPMAP 3  /\* Encoded as zipmap \*/
\#define REDIS_ENCODING_LINKEDLIST 4 /\* Encoded as regular linked list \*/
\#define REDIS_ENCODING_ZIPLIST 5 /\* Encoded as ziplist \*/
\#define REDIS_ENCODING_INTSET 6  /\* Encoded as intset \*/
\#define REDIS_ENCODING_SKIPLIST 7  /\* Encoded as skiplist \*/

Redis内部物理表示类型的编码方式目前有8种:
string类型: REDIS_ENCODING_RAW， REDIS_ENCODING_INT
list类型: REDIS_ENCODING_ZIPLIST, REDIS_ENCODING_LINKEDLIST
set类型: REDIS_ENCODING_HT, REDIS_ENCODING_INTSET
zset类型: REDIS_ENCODING_ZIPLIST, REDIS_ENCODING_SKIPLIST
hash类型: REDIS_ENCODING_ZIPLIST, REDIS_ENCODING_HT

文章说明了文章1,2中方法的原理