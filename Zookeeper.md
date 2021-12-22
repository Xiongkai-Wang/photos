## Zookeeper



### Zookeeper简介

- **What？**Zookeeper 是一个**分布式协调服务**的开源框架。主要用来解决分布式集群中应用系统的一致性问题。ZooKeeper 本质上是一个分布式的小文件存储系统。提供基于类似于文件系统的目录树方式的数据存储，并且可以对树中的节点进行有效管理。从而用来维护和监控你存储的数据的状态变化。通过监控这些数据状态的变化，可以达到基于数据的集群管理。

  - ZooKeeper 数据模型的结构是树形结构，每个节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。

- 特点：![](https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-overview.png)

  - Zookeeper:一个领导者(Leader)，多个跟随者(Follower)组成的集群。集群中只要有**半数**以上节点存活，Zookeeper集群就能正常服务。Zookeeper适合安装奇数台服务器。
  - **全局数据一致**:每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
  - 更新请求**顺序执行**，来自同一个Client的更新请求按其发送顺序依次执行
  - **数据更新原子性**，一次数据更新要么成功，要么失败
  - **实时性**，在一定时间范围内，Client能读到最新数据

- 应用：

  - **服务注册与发现**：在微服务中，服务提供方把服务注册到Zookeeper中心如图中的Member服务，但是每个应用可能拆分成多个服务对应不同的IP地址，Zookeeper注册中心可以动态感知到服务节点的变化。服务消费方Order需要调用提供方Member提供的服务时，从zookeeper中获取提供方的调用地址列表，然后进行调用。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-serviceRegistry.jpg" style="zoom:50%;" />

  - **命名服务**：在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址、提供者等信息。比如说一些分布式服务框架中的服务地址列表。通过调用 Zookeeper 提供的创建节点的 API，能够很容易创建一个全局唯一的 path，这个 path 就可以作为一个名称。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-namingService.png"  style="zoom:50%;" />

  - **统一配置管理**：发布者将数据发布到 Zookeeper节点上，供订阅者动态获取数据，实现配置信息的集中式管理和动态更新。可将配置信息写入ZooKeeper上的一个Znode。各个客户端服务器监听这个Znode。一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-configCenter.png" style="zoom: 33%;" />

  - **分布式锁**：根据Zookeeper有序临时节点的特性，每个进程对应连接一个有序临时节点（进程1对应节点/znode/00000001）。每个进程监听对应的上一个节点的变化。编号最小的节点对应的进程获得锁，可以操作资源。当进程1完成业务后，删除对应的子节点00000001，释放锁。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-distributedLock.jpg" style="zoom:70%;" />



### Zookeeper集群安装与启动

- 下载：https://zookeeper.apache.org/releases.html 选择指定版本下载，如apache-zookeeper-3.5.7- bin.tar.gz

- 集群搭建：以三台为例

  ```shell
  # 解压到指定文件夹
  tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C ./zookeeper # 得到 apache-zookeeper-3.5.7-bin 目录
  
  # 创建3个文件夹分别代表3个zookeeper服务器的dataDir
  mkdir zkData1
  mkdir zkData2
  mkdir zkData3
  # 分别在三个dataDir中创建myid文件，并在myid文件中指定服务器标识
  echo 1 > ./zkData1/myid
  echo 2 > ./zkData2/myid
  echo 3 > ./zkData3/myid
  
  # 在apache-zookeeper-3.5.7-bin目录的conf下创建3个不同的配置文件用于启动3个server
  zoo1.cfg   zoo2.cfg   zoo3.cfg
  
  # 分别启动3台zookeeper server
  cd ./apache-zookeeper-3.5.7-bi
  ./bin/zkServer.sh start ./conf/zoo1.cfg 
  ./bin/zkServer.sh start ./conf/zoo2.cfg
  ./bin/zkServer.sh start ./conf/zoo3.cfg 
  
  # 查看server状态
  ./bin/zkServer.sh status ./conf/zoo1.cfg 
  # 会有 Mode: follower/Leader 的信息
  
  # 连接客户端，端口号为2181,2182,2183
  ./bin/zkCli.sh -server localhost:2181
  ```

  ```shell
  # zoo1.cfg：zookeeper的配置文件
  tickTime=2000 # 通信心跳时间，Zookeeper服务器与客户端心跳时间
  initLimit=10 # LF初始通信时限，Leader和Follower初始连接时能容忍的最多心跳数
  syncLimit=5 # LF同步通信时限，Leader和Follower之间通信时间如果超过syncLimit * tickTime，Leader认为Follwer死 掉，从服务器列表中删除Follwer
  # 指定数据存储位置 zkData1,zkData2,zkData2
  dataDir=/Users/xiongkai/docker/zookeeper/zkData1
  # 指定客户端连接的端口 2181,2182,2183
  clientPort=2181
  # 集群配置：指定集群中服务器
  # server.X 服务器表示，为myid文件中数字
  # 端口1：用于数据同步;端口2：用于leader选举
  server.1=localhost:2880:3881
  server.2=localhost:2882:3883
  server.3=localhost:2884:3885
  ```

  

### 选举机制

- 基本概念：

  - Server id（sid）：服务器ID，即myid文件中的编号。大初始化启动时就是编号越大在选择算法中的权重越
  - Zxid：事务ID：服务器中存放的数据的事务ID，值越大说明数据越新，在选举算法中数据越新权重越大
  - Epoch：逻辑时钟，也叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的，每投完一次票这个数据就会增加
  - 选举状态
    - LOOKING，竞选状态。
    - FOLLOWING，随从状态，同步leader状态，参与投票。
    - OBSERVING，观察状态，同步leader状态，不参与投票
    - LEADING，领导者状态

- 第一次启动：集群投票每个服务器都会首先投自己一票，然后将投票的信息发送给其他服务器。

  - server1启动：发起一次选举，服务器1投自己一票，此时服务器1票数一票，不够半数以上（2票），选举无法完成。服务器1状态保持为LOOKING
  - server2启动：发起一次选举，服务器1和2分别投自己一票，此时服务器1发现服务器2的id比自己大，更改选票投给服务器2。此时服务器2的票数已经超过半数，当选Leader
  - server3启动，发起一次选举，此时服务器1，2已经不是LOOKING 状态，不会更改选票信息。交换选票信息结果：服务器2为2票，服务器3为1票。此时服务器3服从多数，更改选票信息为服务器2。状态为Following

- 非第一次启动：

  - 状态变更。Leader 故障后，余下的Follower服务器都会将自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程
  - 第一次投票，每台机器都会将票投给自己；
  - 接着每台机器都会将自己的投票发给其他机器，如果发现其他机器的zxid比自己大，那么就需要改投票重新投一次。比如server1 收到了三张票，发现server2的xzid为102，pk一下发现自己输了，后面果断改投票选server2。

  

### 客户端常用命令

