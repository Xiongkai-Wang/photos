## Zookeeper



### Zookeeper简介

- **What？**Zookeeper 是一个**分布式协调服务**的开源框架。主要用来解决分布式集群中应用系统的一致性问题。ZooKeeper 本质上是一个分布式的小文件存储系统。提供基于类似于文件系统的目录树方式的数据存储，并且可以对树中的节点进行有效管理。从而用来维护和监控你存储的数据的状态变化。通过监控这些数据状态的变化，可以达到基于数据的集群管理。

  - ZooKeeper 数据模型的结构是树形结构，每个节点称做一个 ZNode。每一个ZNode默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。

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

- 下载：https://zookeeper.apache.org/releases.html 选择指定版本下载，如apache-zookeeper-3.5.7-bin.tar.gz

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

  - Server id（sid）：服务器ID，即myid文件中的编号。初始化启动时就是编号越大在选择算法中的权重越大
  - zxid (ZooKeeper Transaction ID)：服务器中存放的数据的事务ID，值越大说明数据越新，在选举算法中数据越新权重越大
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

  

### 常用命令

- 创建节点：

  ```shell
  create -e /tmp001 firstNode 
  create [-s] [-e] path data acl
  # -s或-e 分别指定节点特性:顺序或临时节点。默认为永久节点
  # acl 用来进行权限控制
  ```

- 读取节点：

  ```shell
  ls [-s] [-w] path  # 列出指定节点下的所有子节点(第一级的所有子节点), 
  # -s 表示列出节点属性信息; -w 监听子节点变化
  ls / # [tmp001, zookeeper]
  
  get [-s] [-w] path  # 查看指定节点的数据内容，
  # -s 表示列出节点属性信息；-w 监听节点内容变化
  get /tmp001 # firstNode
  
  stat path # 查看节点属性信息
  ```

- 更新节点：

  ```shell
  set path data 
  ```

- 删除节点:

  ```shell
  delete path  # 若存在子节点，那么无法删除该节点，需要先删除子节点
  deleteall path # 清空，可以递归删除节点
  ```

- quota 对节点增加限制

  ```shell
  setquota -n|-b val path # 对节点增加限制
  # n:表示子节点的最大个数,b:表示数据值的最大长度,val:子节点最大个数或数据值的最大长度
  listquota path # 列出指定节点的 quota
  delquota [-n|-b] path # 删除指定节点的quota
  ```

- 查看历史：

  ```shell
  history # 列出命令历史, 且含有一个命令编号
  redo 1 # 重新执行指定命令编号的历史命
  ```

  

### 数据模型

- **What?** ZooKeeper 的数据模型，在结构上和标准文件系统的非常相似，拥有一个层 次的命名空间，都是采用树形层次结构，ZooKeeper 树中的每个节点被称为Znode

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-nodes.png" style="zoom:70%;" />

- **节点类型**：节点的类型在创建时即被确定，并且不能改变

  - **临时节点和永久节点**：

    - 临时节点的生命周期**依赖于创建节点的会话**（client和server的一次连接)。一旦会话结束，临时节点将被自动删除。临时节点不允许拥有子节点
    - 永久节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。

  - **节点的序列化特性**：创建节点的时候指定序列化 `-s` ，该 Znode 的名字后会自动追加一个不断增加的序列号。序列号对于此节点的父节点来说是唯一 的，这样便会**记录每个子节点创建的先后顺序**

    ```shell
    [zk: localhost:2181(CONNECTED) 31] create -s /seqParent/request fromClient01
    Created /seqParent/request0000000002
    [zk: localhost:2181(CONNECTED) 32] create -s /seqParent/request fromClient02
    Created /seqParent/request0000000003
    [zk: localhost:2181(CONNECTED) 33] create -s /seqParent/request fromClient03
    Created /seqParent/request0000000004
    [zk: localhost:2181(CONNECTED) 34] get /seqParent/request
    org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /seqParent/request
    [zk: localhost:2181(CONNECTED) 35] ls /seqParent
    [request0000000002, request0000000003, request0000000004]
    ```

  - 所以一共会有**四类节点**：

    - PERSISTENT :永久节点
    - EPHEMERAL :临时节点
    - PERSISTENT_SEQUENTIAL :永久节点、序列化
    - EPHEMERAL_SEQUENTIAL :临时节点、序列化

- **节点属性**：可以通过`stat path` ; `ls -s path`;  `get -s path`; 查看

  ```shell
  [zk: localhost:2181(CONNECTED) 9] stat /zookeeper
  cZxid = 0x0  # *c表示create，znode第一次被创建的事务zxid
  ctime = Thu Jan 01 08:00:00 CST 1970  # *znode被创建的时间戳，从 1970 年开始
  mZxid = 0x0 # *Znode被最新修改的事务id，每次对znode的修改都会更新
  mtime = Thu Jan 01 08:00:00 CST 1970 # *znode最后一次修改的时间戳
  pZxid = 0x0 # 最后更新的子节点zxid
  cversion = -1 # znode 子节点变化号，znode 子节点修改次数
  dataVersion = 0 # *znode 数据变化号
  aclVersion = 0 # znode 访问控制列表的变化号
  ephemeralOwner = 0x0  # 若为临时节点，则是znode拥有者的sessionID。若非临时节点则是0。
  dataLength = 0 # *znode 的数据长度
  numChildren = 2 # *znode 子节点数量
  ```



### 监听器

- What? ZooKeeper引入了 **Watcher 机制**来提供**分布式数据发布/订阅**功能。客户端注册监听它关心的目录节点，当目录节点发生变化时(数据改变、节点删除、子节点增加删除)，ZooKeeper 会通知客户端。监听机制让ZooKeeper保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序。

- 特点：

  - **一次性触发**：事件发生触发监听，一个watcher event就会被发送到设置监听的客户端，这种效果是一次性的。后续再次发生同样的事件，不会再次触发，需要重新注册。
  - **事件封装**：ZooKeeper 使用 WatchedEvent 对象来封装服务端事件并传递。对象包含3个属性：通知状态(keeperState)，事件类型(EventType)和节点路径(path)
  - **event 异步发送**：WatchedEvent从服务端发送到客户端是异步的
  - **先注册再触发**：必须客户端先去服务端注册监听，事件发生才会触发监听，通知给客户端

- 监听内容：

  ```shell
  # 监听节点数据的变化
  get -w path
  # 监听子节点增减的变化
  ls -w path
  
  # [watch]方式已经废除
  get /tmp001 watch
  'get path [watch]' has been deprecated. Please use 'get [-s] [-w] path' instead.
  ```

- 案例：

  ```shell
  # 先在2182客户端中注册监听
  [zk: localhost:2182(CONNECTED) 1] get -w /tmp002
  null
  # 2181客户端中修改节点内容
  [zk: localhost:2181(CONNECTED) 40] set /tmp002 change
  # 2182客户端得到修改内容的通知
  [zk: localhost:2182(CONNECTED) 2] 
  WATCHER::
  
  WatchedEvent state:SyncConnected type:NodeDataChanged path:/tmp002
  ```

  

### Java API

- **What？** Zookeeper的Java API主要包括两个类

  - `org.apache.zookeeper.Zookeeper`  客户端主类，负责建立与zookeeper集群的会话， 并提供方法进行操作。
  - `org.apache.zookeeper.Watcher` 监听器接口，其定义了事件通知相关的逻辑， 包含 KeeperState 和 EventType 两个枚举类，分别代表了通知状态和事件类型， 同时定义了事件的回调方法:`process(WatchedEvent event)`

- 引入依赖

  ```xml
  <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.5.7</version>
  </dependency>
  ```

- 基本使用

  ```java
  public class TestZK {
      @Test
      public void testZk() throws Exception{
          String connection = "localhost:2181,localhost:2182,localhost:2183";
          Integer sessionTimeout = 20000;
          // 创建客户端对象，连接 zookeeper 服务器
          ZooKeeper zkClient = new ZooKeeper(connection, sessionTimeout, new Watcher() {
            	@Override
              public void process(WatchedEvent watchedEvent) {
                  System.out.println("通知状态: " + watchedEvent.getState());
                  System.out.println("通知类型: " + watchedEvent.getType());
                  System.out.println("节点路径: " + watchedEvent.getPath());
              }
          });
  
          // 查看根节点的子节点, 相当于ls
          System.out.println("-------------");
          System.out.println("根节点的子节点：" + zkClient.getChildren("/", false)); 
          if (zkClient.exists("/java", false) != null) { // 不存在返回null，存在则返回节点属性信息
              zkClient.delete("/java", -1); // version=-1表示可以match任何version
          }
  
          // 创建节点: 路径，数据，acl权限控制列表，节点类型
          zkClient.create("/java", "java".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
  
          // 创建子节点
          zkClient.create("/java/child01", "child01".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
          zkClient.create("/java/child02", "child02".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
  
          // 取出子节点列表, false表示不监听
          System.out.println("java节点的子节点：" + zkClient.getChildren("/java", false));
  
          // 查看节点数据并监听： 路径，是否watch，stat节点状态(null表示不输出节点属性信息)
          System.out.println("/java节点的数据为：" + new String(zkClient.getData("/java", true, null)));
  
          // 设置节点数据, version=-1表示可以match任何version
          System.out.println("************");
          zkClient.setData("/java", "new java".getBytes(), -1);
  
          // 关闭客户端
          System.out.println("============");
          zkClient.close();
      }
  }
  ```

  

### 分布式锁java实现

- **What？**比如说"进程 1"在使用该资源的时候，会先去获得锁，"进程 1"获得锁以后会对该资源保持独占，这样其他进程就无法访问该资源，"进程 1"用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，我们就能保证了分布式系统中多个进程能够有序的 访问该临界资源。那么我们把这个分布式环境下的这个锁叫作分布式锁。

- **Zookeeper实现分布式锁**

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/zookeeper-lock.png" style="zoom:50%;" />

```java
public class TestLock {
    public static void main(String[] args) throws Exception{
        final DistributedLock lock1 = new DistributedLock();
        final DistributedLock lock2 = new DistributedLock();
        // 构造两个线程模拟分别用不同的客户端访问同一个资源
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock1.lock();
                    System.out.println("Thread1 got lock");
                    Thread.sleep(1000); // 模拟业务处理
                    lock1.unlock();
                    System.out.println("Thread1 released lock");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock2.lock();
                    System.out.println("Thread2 got lock");
                    Thread.sleep(1000);
                    lock2.unlock();
                    System.out.println("Thread2 released lock");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
        thread1.start();
        thread2.start();
    }
}
```

```java
public class DistributedLock {
    private String connection = "localhost:2181,localhost:2182,localhost:2183";
    private Integer sessionTimeout = 20000;
    private ZooKeeper zk;
    private String rootNode = "locks";
    private String subNode = "seq";
    private String waitPath; // 监听的节点路径，即排序前一位的节点
		private String currentNode;  // 当前节点路径，即该客户端代表的访问请求
    private CountDownLatch connectLatch = new CountDownLatch(1);
    private CountDownLatch waitLatch = new CountDownLatch(1); 
    
    public DistributedLock() throws IOException, InterruptedException, KeeperException {
         // 连接zookeeper
         zk = new ZooKeeper(connection, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                // 连接建立时, 打开 latch, 唤醒 wait 在该 latch 上的线程
                if (event.getState() == Event.KeeperState.SyncConnected) {
                    connectLatch.countDown();
                }
                //  发生了节点删除事件，如果是waitPath（即释放锁的是该节点监视的节点），则可以唤醒waitLatch的线程了
                if (event.getType() == Event.EventType.NodeDeleted &&
                        event.getPath().equals(waitPath)) {
                    waitLatch.countDown();
                }
            }
        });
        // 等待连接建立
        connectLatch.await(); // 当countDown到0时，就会结束等待
        // 如果是第一个到达的客户端，需要创建/lock根节点
        Stat stat = zk.exists('/' + rootNode, false);
        if (stat == null) {
            System.out.println("lock根节点不存在");
            zk.create("/" + rootNode, "lock".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    }

    public void lock() {
        try {
            // 当该客户端连接到服务器，在根节点下创建临时顺序节点，返回值为创建的节点路径
            currentNode = zk.create("/" + rootNode + "/" + subNode, null,
                    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
            Thread.sleep(10);
            // 获取root节点下所有排队的节点
            List<String> childrenNodes = zk.getChildren("/" + rootNode, false);
            if (childrenNodes.size() == 1){
                // 列表中只有一个子节点, 该client获得锁
                return;
            } else {
                //对根节点下的所有临时顺序节点进行从小到大排序
                Collections.sort(childrenNodes);
                //当前节点名称
                String thisNode = currentNode.substring(("/" + rootNode + "/").length());
                //获取当前节点的位置
                int index = childrenNodes.indexOf(thisNode);

                if (index == -1) {
                    System.out.println("Exception");
                } else if (index == 0) {
                    // 排名最前，可以获得锁
                    return;
                } else {
                    // 获得排名比currentNode前 1 位的节点，并监视该节点
                    this.waitPath = "/" + rootNode + "/" + childrenNodes.get(index - 1);
                    zk.getData(waitPath, true, new Stat());   
                    // 进入等待状态
                    waitLatch.await(); // 当countDown到0时，就会结束等待
                    return;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void unlock() {
        try {
            zk.delete(currentNode, -1);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- **CountDownLatch**:  Java并发编程中的常用类。CountDownLatch主要有两个方法：countDown()和await()。countDown()方法用于使计数器减一，其一般是执行任务的线程调用，await()方法则使调用该方法的当前线程处于等待状态，其一般是主线程调用。构造CountDownLatch时传入一个整数n，在这个整数“倒数”到0之前，主线程需要等待在门口，而这个“倒数”过程则是由各个执行线程驱动的，每个线程执行完一个任务“倒数”一次。总的来说，CountDownLatch的作用就是等待其他的线程都执行完任务，然后主线程才继续往下执行。

  

  

  

