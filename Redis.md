### **Redis**持久化操作



#### **RDB-redis database**

- What? 在指定的时间间隔内将内存中的数据集快照（dump.rdb）写入磁盘，也就是行话讲的 Snapshot 快照，它恢复时是将快照文件直接读到内存里 。

- 原理： Redis 会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何 IO 操作的，确保了极高的性能。但是最后一次持久化后的数据可能丢失。 

- 具体操作：

  - 在 redis.conf 中配置持久化文件名称，默认为 dump.rdb； 默认位置为 Redis 启动时命令行所在的目录下 ；

    ```sh
    # 在redis.conf中设置持久化规则
    save 30 3 # 在30秒内至少有3次发生写的操作
    
    # 其他设置
    stop-writes-on-bgsave-error yes # 当 Redis 无法写入磁盘的话，直接关掉 Redis 的写操作
    rdbcompression yes # 对于存储到磁盘中的快照，设置是否进行压缩存储。如果yes，redis会采用 LZF算法进行压缩
    rdbchecksum yes # 存储快照后，还可以让redis使用CRC64算法来进行数据校验,但是这样做会增加大约 10%的性能消耗
    ```

  - 如何备份和共享：先通过config get dir 查询rdb文件的目录 ，再将*.rdb 的文件拷贝到别的地方，cp dump2.rdb dump.rdb 数据迁移。

#### **AOF-append only file**

- What? 以日志的形式来记录每个写操作(增量保存)，将 Redis 执行过的所有写指令记录下 来(读操作不记录)， 只许追加文件但不可以改写文件，配置之后，redis 启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一 次以完成数据的恢复工作 。

- 具体操作：

  - AOF默认不开启；可以在 redis.conf 中修改默认的 **appendonly** no，改为 yes开启。

  - 配置持久化文件名称，默认为 **appendonly.aof**，AOF 文件的保存路径，同 RDB 的路径一致。

  - AOF 和 RDB 同时开启 ，系统默认取 AOF 的数据

  - 恢复：如遇到 AOF 文件损坏，通过/usr/local/bin/**redis-check-aof--fix appendonly.aof** 进行恢复。

  - AOF 同步频率设置：

    ```shell
    appendfsync always  # 每次 Redis 的写入都会立刻记入日志
    appendfsync everysec # 每秒记录日志
    appendfsync no redis # 不主动进行同步，把同步时机交给操作系统 
    ```

- Rewrite压缩：

  - What？AOF 采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当 AOF 文件的大小超过所设定的阈值时，Redis 就会启动 AOF 文件的内容压缩， 只保留可以恢复数据的最小指令集。可以使用命令手动开始重写

    ```shell
    bgrewriteaof 
    ```

  - 过程： AOF 文件持续增长而过大时，会 fork 出一条新进程来将文件重写(也是先写临时文件最 后再 rename)，redis4.0 版本后的重写，是指上就是把 rdb 的快照，以二级制的形式附 在新的 aof 头部，作为已有的历史数据，替换掉原来的流水账操 

  - 自动配置，何时触发重写？重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的， 因此设定 Redis 要满足一定条件才会进行重写 

    ```shell
    auto-aof-rewrite-min-size 64M # default
    auto-aof-rewrite-percentage 100% # 设置重写的基准值，文件达到100%时开始重写(文件是原来重写后文件的2倍时触发）
    ```

#### RDB与AOF比较

- RDB：
  - 适合大规模的数据恢复；对数据完整性和一致性要求不高更适合使用；节省磁盘空间；恢复速度快 
  - 但是：虽然 Redis 在 fork 时使用了写时拷贝技术,但是如果数据庞大时还是比较消 耗性能；在备份周期在一定间隔时间做一次备份，所以如果 Redis 意外 down 掉的话， 就会丢失最后一次快照后的所有修改
- AOF：
  - 备份机制更稳健，丢失数据概率更低；可读的日志文本，通过操作AOF稳健，可以处理误操作
  - 但是： 比起RDB占用更多的磁盘空间；恢复备份速度要慢；每次读写都同步的话，有一定的性能压力

- 官方推荐同时开启。如果对数据不敏感，可以选单独用 RDB。
  - 在master-salve机制下：因为 RDB 文件只用作后备用途，建议只在 Slave 上持久化 RDB 文件，而且只要 15 分钟备份一次就够了，只保留 save 900 1 这条规则

### **Redis**主从复制

- What? 主机数据更新后根据配置和策略，自动同步到备用机的 master/slaver 机制，**Master** 以写为主，**Slave** 以读为主

  <img src="/Users/xiongkai/Desktop/应用.png" alt="应用" style="zoom:50%;" />

- Why？ 读写分离，性能扩展（减轻主机负担，高并发场景）；容灾快速恢复。

- 主从复制原理：Slave 启动成功连接到 master 后会发送一个 sync 命令 ；Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master 将传送整个数据文件到 slave,以完成一次完全同步；全量复制:而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中 ；增量复制:Master 继续将新的所有收集到的修改命令依次传给 slave,完成同步；

- 具体操作：以 一主两从 为例

  - 以配置文件的方式启动多个**redis** 服务器：**redis6379**，**redis6380**，**redis6381**

    ```shell
    # 配置文件格式:
    include /myredis/redis.conf   # 引入配置文件，以此为基础修改下面的配置
    pidfile /var/run/redis_6379.pid  # 配置不同的pidfile
    port 6379  # 配置不同端口
    dbfilename dump6379.rdb  # 配置不同的持久化dump文件
    
    # 命令：
    redis-server redis6379.conf
    redis-server redis6380.conf
    redis-server redis6381.conf
    
    # 打印主从复制的相关信息
    info replication 
    ```

  - 将**redis6380**，**redis6381**设置为**masterredis6379**的**slave** 

    ```shell
    slaveof 127.0.0.1 6379 # 在从机命令行中运行
    ```

- 薪火相传 ：上一个 Slave 可以是下一个 slave 的 Master，Slave 同样可以接收其他 slaves 的连接和同步请求，那么该 slave 作为了链条中下一个的 master, 可以有效减轻 master 的写压力,去中心化降低风险。 

- 反客为主：当一个 master 宕机后，后面的 slave 可以立刻升为 master，其后面的 slave 不用做任何修改。用如下命令手动将从机变为主机  

  ```shell
  slaveof no one 
  ```

- 哨兵模式（sentinel）反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库 转换为主库 。

  - 启动哨兵模式：

    ```shell
    redis-sentinel /myredis/sentinel.conf 
    ```

  - sentinel.conf 文件内容：

    ```shell
    sentinel monitor mymaster 127.0.0.1 6379 1
    # 其中 mymaster 为监控对象的服务器自定义名称， 1 为至少有多少个哨兵同意迁移的数量
    ```

  - 如何确定新的**master**？ 

    - 优先级在 redis.conf 中默认:replica-priority 100，值越小优先级越高
    - 选择偏移量大的，偏移量是指获得原主机数据最全的
    - 选择runid小的。每个 redis 实例启动后都会随机生成一个 40 位的 runid

