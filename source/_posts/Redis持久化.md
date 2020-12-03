---
title: Redis 持久化
date: 2018-03-16T08:25:31.000Z
tags: ['Redis']
categories: [Concepts]
---
## Preface

> Redis 不仅可以基于内存进行高速的数据读写，还提供了两种持久化方式，即 **RDB**(or **Redis Database**) 和 **AOF**(or **Append Only File**)，本文主要介绍这两种持久化方式的原理及区别

## RDB

> RDB 持久化可以在指定的时间间隔内生成数据集的**快照**(snapshot)

- 我们可以在配置文件`/etc/redis/redis.conf`中查看 RDB 持久化的配置

    ```
    ################################ SNAPSHOTTING  ################################           
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

    save 900 1
    save 300 10
    save 60 10000
    ```
    格式如下`save <seconds> <changes>`，比如`save 900 1`就表示 **900秒内如果有超过一个键被修改就自动保存一次数据集**

- 另外两个重要信息就是快照写入的文件名以及文件夹路径

    ```
    # The filename where to dump the DB
    dbfilename dump.rdb
    
    # The working directory.
    #
    # The DB will be written inside this directory, with the filename specified
    # above using the 'dbfilename' configuration directive.
    #
    # The Append Only File will also be created inside this directory.
    #
    # Note that you must specify a directory here, not a file name.
    dir /var/lib/redis
    ```
    可以看到快照的文件名是`dump.rdb`，保存路径`/var/lib/redis`，**RDB 和 AOF 的持久化文件都保存在这个目录下**

- 另外也可以在`redis-cli`中查看配置

    ```
    127.0.0.1:6379> CONFIG GET save
    1) "save"
    2) "900 1 300 10 60 10000"
    ```
    可以通过调用`SAVE`或`BGSAVE` (background save)，手动进行数据集保存
    ```
    127.0.0.1:6379> SAVE
    OK
    ```
    
### RDB 的特性

- RDB 是一个非常紧凑的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于备份

- RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 **`fork`出一个子进程**，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作

- **RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快**

- 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然你可以通过设置来控制保存 RDB 文件的频率。但是，因为 RDB 文件需要保存整个数据集的状态，所以它并不是一个轻松的操作。因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下，一旦发生故障停机，你就可能会丢失好几分钟的数据

- 每次保存 RDB 的时候，Redis 都要`fork`出一个子进程，并由子进程来进行实际的持久化工作。**在数据集比较庞大时，`fork`可能会非常耗时**，造成服务器在某某毫秒内停止处理客户端；如果数据集非常巨大，并且 CPU 时间非常紧张的话，甚至可能会停止长达整整一秒。虽然 AOF 重写也需要进行`fork`，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失
    
## AOF

> AOF 持久化记录服务器执行的所有**写操作命令**，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被 **追加(append)** 到文件的末尾。 Redis 还可以在后台对 AOF 文件进行 **重写(rewrite)** ，使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小

- 在配置文件中查看 AOF 设置

    ```
    ############################## APPEND ONLY MODE ###############################

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

    appendonly no

    # The name of the append only file (default: "appendonly.aof")

    appendfilename "appendonly.aof"

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
    # If unsure, use "everysec".

    # appendfsync always
    appendfsync everysec
    # appendfsync no
    ```
    
- 默认`appendonly no`，需要改为`yes`开启 AOF 功能，修改后重启`redis-server`，发现`/var/lib/redis`多了个`appendonly.aof`文件，即以 Redis 协议格式保存的一条条命令

    ```
    $ ls /var/lib/redis
    appendonly.aof  dump.rdb
    ```
    
- 我们打开`redis-cli`执行一些写操作

    ```
    127.0.0.1:6379> set foo "foooo"
    OK
    127.0.0.1:6379> set bar "barrr"
    OK
    127.0.0.1:6379> keys *
    1) "foo"
    2) "bar"
    ```
    查看`appendonly.aof`，文件可读性很强，`*3`表示命令由 3 个短语构成，`$3`表示短语长度为 3，第一句`SELECT 0`表示选中第一个数据库
    ```
    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    set
    $3
    foo
    $5
    foooo
    *3
    $3
    set
    $3
    bar
    $5
    barrr
    ```
    
- 我们将上述文件 13 ~ 19 行删除，重启`redis-server`，发现第二个键已被我们人工删除。

    ```
    127.0.0.1:6379> keys *
    1) "foo"
    ```

> 实际上，Redis 服务器在重启时会读入 AOF 文件，按顺序执行每条命令来还原数据集

### AOF 的特性

- 从配置文件的注释也可以看出 **AOF 持久化比 Redis 持久化更耐用(durability)**。可以设置不同的 fsync 策略

    ```
    # appendfsync always    // 每次执行写入命令时 fsync
    appendfsync everysec    // 每秒钟一次 fsync
    # appendfsync no    // 无 fsync
    ```
    默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据
  
- 当 Redis 启动时，如果 RDB 持久化和 AOF 持久化都被打开了，那么程序会优先使用 AOF 文件来恢复数据集，因为 **AOF 文件所保存的数据通常是最完整的**
  
- AOF 文件有序地保存了对数据库执行的所有写入操作，可读性很高，可用于分析、操作回滚。举个例子，如果你不小心执行了`FLUSHALL`命令， 但只要 AOF 文件未被重写，那么只要停止服务器，移除 AOF 文件末尾的 `FLUSHALL`命令，并重启 Redis ，就可以将数据集恢复到`FLUSHALL`执行之前的状态

- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写，**重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合**。也可以通过`BGREWRITEAOF`命令手动重写 AOF 文件，因此我们可以做个测试，一直`SET`和`DEL`某个 key，在重写后发现 AOF 文件中多余的步骤已被删除

- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。在一般情况下，`appendfsync everysec`性能依然非常高，因为它是性能和数据安全性之间的平衡点。`appendfsync always`策略在实际使用中非常慢，频繁调用 fsync 注定了这种策略不可能快得起来

## Conclusion

> 一般来说，如果想达到足以媲美 PostgreSQL 的数据安全性，你应该**同时使用两种持久化功能**。有很多用户都只使用 AOF 持久化，但并不推荐这种方式: 因为定时生成 RDB 快照非常便于进行数据库备份，并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快。
> 实际上，官方也提出未来可能会将 AOF 和 RDB 整合成单个持久化模型。


## References

- [Redis Persistence --- 官网](https://redis.io/topics/persistence)
- [Redis持久化 --- 中文文档](http://doc.redisfans.com/topic/persistence.html)