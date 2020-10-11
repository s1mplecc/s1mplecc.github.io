---
title: Redis 五种数据类型
date: 2018-03-16T08:27:36.000Z
tags: ['Redis']
categories: [Concepts]
---
## Preface

> 本文介绍 Redis 的五种数据类型，并结合`redis-cli`和 Koa 应用演示如何与 Redis 的五种数据类型进行 CRUD 操作，在文章最后还有关于**过期时间设置**的介绍

- 代码已上传至 [GitHub](https://github.com/s1mplecc/koa-redis-example)
- Node 框架使用 Koa2，与 Redis 的操作都使用 Restful API 的形式展现，使用 Postman 测试接口
- 使用的依赖为`ioredis`，执行`npm i ioredis`即可安装依赖。之前有尝试`noderedis`，但是在服务器上部署运行有连接不上 Redis 的 Bug 出现

## Data Type

> Redis 通常被称为**数据结构服务器**，因为它提供了五种数据类型用于数据存储，包括 **String** 字符串、**Hash** 哈希、**List** 列表、**Set** 集合、**Zset** 有序集合

### String

> Redis String 是 Redis 最基本的类型，用于保存字符串类型的 Key-Value 键值对

- `redis-cli`

    ```
    127.0.0.1:6379> SET string:s1 hello
    OK
    127.0.0.1:6379> GET string:s1
    "hello"
    127.0.0.1:6379> DEL string:s1
    (integer) 1
    ```
    
- `contollers/string.js`

    ```js
    const router = require('koa-router')()
    const redis = require('../config/redis')

    const setString = async (ctx) => {
      ctx.body = await redis.set('string:s1', 'hello world')
    }

    const getString = async (ctx) => {
      ctx.body = await redis.get('string:s1')
    }

    const delString = async (ctx) => {
      ctx.body = await redis.del('string:s1')
    }

    router.post('/string', setString)
    router.get('/string', getString)
    router.delete('/string', delString)

    module.exports = router
    ```
    
### Hash

> Redis Hash 是一个 String 类型的 Field-Value 映射表，类似于 Java 的 HashMap，特别适合用于**存储对象**。

- `redis-cli`

    ```
    127.0.0.1:6379> HMSET hash:hash1 name "jack" age 10 score 90
    OK
    127.0.0.1:6379> HGET hash:hash1 name
    "jack"
    127.0.0.1:6379> HGETALL hash:hash1
    1) "name"
    2) "jack"
    3) "age"
    4) "10"
    5) "score"
    6) "90"
    127.0.0.1:6379> HDEL hash:hash1 name age
    (integer) 2
    127.0.0.1:6379> HGETALL hash:hash1
    1) "score"
    2) "90"
    127.0.0.1:6379> DEL hash:hash1
    (integer) 1
    ```
    需要特别注意的是，`HGETALL key`命令用于获取键的映射列表，`HGETALL key field`用于获取某个字段的值，`HDEL key field`用于删除键的某个字段，`DEL key`才是删除整个键

- `contollers/hash.js`

    ```js
    const setHash = async (ctx) => {
      const student = {
        name: 'Jack',
        age: 18,
        score: 90
      }
      ctx.body = await redis.hmset('hash:hash1', student)
    }

    const getHashAll = async (ctx) => {
      ctx.body = await redis.hgetall('hash:hash1')
    }

    const getHashByField = async (ctx) => {
      ctx.body = await redis.hget('hash:hash1', ctx.params.field)
    }

    const delHash = async (ctx) => {
      ctx.body = await redis.del('hash:hash1')
    }

    const delHashByField = async (ctx) => {
      ctx.body = await redis.hdel('hash:hash1', ctx.params.field)
    }

    router.post('/hash', setHash)
    router.get('/hash', getHashAll)
    router.get('/hash/:field', getHashByField)
    router.delete('/hash', delHash)
    router.delete('/hash/:field', delHashByField)
    ```

### List

> Redis 列表是简单的字符串列表，**按照插入顺序排序**。你可以添加一个元素到列表的头部（左边）或者尾部（右边），列表中的**元素可以重复**
 
- `redis-cli`

    ```
    127.0.0.1:6379> LPUSH list:list1 hello
    (integer) 1
    127.0.0.1:6379> LPUSH list:list1 world my friend
    (integer) 4
    127.0.0.1:6379> RPUSH list:list1 right push
    (integer) 6
    127.0.0.1:6379> LLEN list:list1
    (integer) 6
    127.0.0.1:6379> LRANGE list:list1 0 5
    1) "friend"
    2) "my"
    3) "world"
    4) "hello"
    5) "right"
    6) "push"
    127.0.0.1:6379> LPOP list:list1
    "friend"
    127.0.0.1:6379> RPOP list:list1
    "push"
    127.0.0.1:6379> LINDEX list:list1 2
    "hello"
    ```
    `LPUSH`添加到列表头部（Left Push），`RPUSH`添加到列表头部（Right Push）。相应的，`LPOP`和`RPOP`即从列表头和列表尾移除一个元素并获取该值。
    
- `LREM`根据参数`count`的值，移除列表中与参数 value 相等的元素

    ```
    count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
    count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
    count = 0 : 移除表中所有与 VALUE 相等的值。
    ```
    语法如下：`LREM key count value`
    ```
    127.0.0.1:6379> LRANGE list:list1 0 100
    1) "my"
    2) "world"
    3) "hello"
    4) "right"
    5) "hello"
    6) "foo"
    7) "hello"
    127.0.0.1:6379> LREM list:list1 -2 hello
    (integer) 2
    127.0.0.1:6379> LRANGE list:list1 0 100
    1) "my"
    2) "world"
    3) "hello"
    4) "right"
    5) "foo"
    ```
    
- `contollers/list.js`

    ```js
    const pushListAndGetAll = async (ctx) => {
      for (let i = 0; i < 5; i++) {
        redis.lpush('list:list1', i)
      }
      for (let i = 5; i < 10; i++) {
        redis.rpush('list:list1', i)
      }
      const length = await redis.llen('list:list1')
      console.log('length', length)
      ctx.body = await redis.lrange('list:list1', 0, length)
    }

    const popList = async (ctx) => {
      const lPopValue = await redis.lpop('list:list1')
      const rPopValue = await redis.rpop('list:list1')
      ctx.body = { lPopValue, rPopValue }
    }

    const deleteListByValue = async (ctx) => {
      ctx.body = await redis.lrem('list:list1', 0, ctx.params.value)
    }

    router.post('/list', pushListAndGetAll)
    router.delete('/list', popList)
    router.delete('/list/:value', deleteListByValue)
    ```

### Set 

> Redis 的 Set 是 String 类型的**无序集合**。集合是通过哈希表实现的，所以添加、删除、查找的复杂度都是 O(1)

- `redis-cli`

    ```
    127.0.0.1:6379> SADD set:set1 hello
    (integer) 1
    127.0.0.1:6379> SADD set:set1 my friend hello world
    (integer) 3
    127.0.0.1:6379> SMEMBERS set:set1
    1) "hello"
    2) "world"
    3) "friend"
    4) "my"
    127.0.0.1:6379> SREM set:set1 my
    (integer) 1
    127.0.0.1:6379> SPOP set:set1
    "hello"
    127.0.0.1:6379> SMEMBERS set:set1
    1) "world"
    2) "friend"
    ```
    `SMEMBERS`返回集合中的所有成员，`SPOP`移除并返回集合中的一个随机元素，`SREM`移除集合中一个或多个成员

- `contollers/set.js`

    ```js
    const addMembersAndGetAll = async (ctx) => {
      redis.sadd('set:set1', 'hello', 'world', 'my', 'friend', 'hello')
      ctx.body = await redis.smembers('set:set1')
    }

    const popRandomMember = async (ctx) => {
      ctx.body = await redis.spop('set:set1')
    }

    const removeMemberByValue = async (ctx) => {
      ctx.body = await redis.srem('set:set1', ctx.params.value)
    }

    router.post('/set', addMembersAndGetAll)
    router.delete('/set/random', popRandomMember)
    router.delete('/set/:value', removeMemberByValue)
    ```
    
### Zset

> Redis Zset，即有序集合（sorted set），和集合一样也是 String 类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个 Double 类型的**分数** (**Score**)。Redis 正是通过分数来为集合中的成员进行从小到大的排序。有序集合的成员是唯一的,但分数却可以重复。

- `redis-cli`

    ```
    127.0.0.1:6379> ZADD zset:zset1 1 a
    (integer) 1
    127.0.0.1:6379> ZADD zset:zset1 2 b 3 c 4 d 5 e
    (integer) 4
    127.0.0.1:6379> ZRANGE zset:zset1 0 4
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "e"
    127.0.0.1:6379> ZREM zset:zset1 c
    (integer) 1
    127.0.0.1:6379> ZRANK zset:zset1 d
    (integer) 2
    127.0.0.1:6379> ZSCORE zset:zset1 d
    "4"
    ```
    `ZREM`移除有序集合中的一个或多个成员，`ZRANK key member`返回有序集合中指定成员的索引，`ZSCORE`返回有序集中，成员的分数值

- `contollers/set.js`
    
    ```js
    const addMembersAndGetAll = async (ctx) => {
      const students = [['Jack', 99], ['John', 90], ['Tony', 88], ['Helen', 92]]
      students.forEach((item) => {
        redis.zadd('zset:zset1', item[1], item[0])
      })
      const count = await redis.zcard('zset:zset1')
      ctx.body = await redis.zrange('zset:zset1', 0, count)
    }

    const removeMemberByValue = async (ctx) => {
      ctx.body = await redis.zrem('zset:zset1', ctx.params.value)
    }

    router.post('/zset', addMembersAndGetAll)
    router.delete('/zset/:value', removeMemberByValue)
    ```
    
## Expire

> 在使用 Redis 的时候遇到了需要设置过期时间的情况，需要注意的是：**Redis 只能针对最顶级的 KEY，也就是五种数据结构设置过期时间**，而不能针对譬如 Hash 中的某一条映射设置。在设计数据存储类型时需要考虑到这点
 
- 设置过期时间使用`EXPIRE`关键字，时间单位为秒(s)，时间到后，整个 hash:hash1 键将被删除

    ```
    127.0.0.1:6379> EXPIRE hash:hash1 60
    (integer) 1
    ```
    
- 可以使用`TTL`查看 KEY 的剩余过期时间，`TTL`为 -1 表示未设置过期时间

    ```
    127.0.0.1:6379> TTL hash:hash1
    (integer) 46
    ```

## References

- [Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)
- [Redis 过期时间](http://www.redis.cn/commands/expire.html)