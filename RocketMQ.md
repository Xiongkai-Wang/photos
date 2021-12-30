## RocketMQ



### RocketMQ介绍

- **RocketMQ**是一款阿里巴巴开源的消息中间件。是使用Java语言开发的一款MQ产品，单机吞吐量达到十万级，经过数年阿里双11的考验，性能与稳定性非常高。2017 年 9 月 25 日，Apache 宣布 RocketMQ 孵化成为 Apache 顶级项目( TLP )，成为国内首个互联网中间件在 Apache 上的顶级项目。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq-intro.png" style="zoom:50%;" />

- **MQ协议**：一般情况下MQ的实现是要遵循一些常规性协议的，

  - 常见的协议包括**JMS**（Java Messaging Service，Java消息服务）、**STOMP**（Streaming Text Orientated Message Protocol，面向流文本的消息协议）、**AMQP**（Advanced Message Queuing Protocol，高级消息队列协议）、**MQTT**（Message Queuing Telemetry Transport，消息队列遥测传输）
  - **ActiveMQ**是JMS与STOMP的典型实现，**RabbitMQ**是AMQP的典型实现。而**RocketMQ**与Kafka都是使用**自研协议**。所以会有差别。

  

### 基本概念

- **Message**：消息是指消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题Topic。
- **Topic**：Topic表示一类消息的集合，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。一个生产者可以同时发送多种Topic的消息; 而一个消费者只对某种特定的Topic感兴趣，即只可以订阅和消费一种Topic的消息。
- **Tag**：为消息设置的标签，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。
  - 可以说Topic是消息的一级分类，Tag是消息的二级分类。
- **Queue**：队列是存储消息的物理实体。一个Topic中可以包含多个Queue，Queue也被称为一个Topic中消息的分区(Partition)。一个Topic的Queue中的消息只能被同一个消费者组中的一个消费者消费，不允许同一个消费者组中的多个消费者同时消费。
- **MessageId & Key**：RocketMQ中每个消息拥有唯一的MessageId，且可以携带具有业务标识的Key，以方便对消息的查询。MessageId有两个: 在生产者send()消息时会自动生成一个`MessageId(msgId)`， 当消息到达Broker后，Broker也会自动生成一个`MessageId(offsetMsgId)`。`msgId`、`offsetMsgId`与`key`都称为**消息标识**。
  - `msgId`: 由producer端生成，其生成规则为: producerIp + 进程pid + MessageClientIDSetter类的ClassLoader的hashCode +当前时间 + AutomicInteger自增计数器
  - `offsetMsgId`:  由broker端生成，其生成规则为:brokerIp + 物理分区的offset(Queue中的偏移量)
  - `key`:  由用户指定的业务相关的唯一标识



### 系统架构

- **Producer**：消息发布的角色，支持分布式集群方式部署。Producer通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递。RocketMQ中的Producer是以生产者组(Producer Group)的形式出现的。生产者组是同一类生产者的集合，这类Producer发送相同Topic类型的消息，可以是一个或者多个Topic。

- **Consumer**：消息消费的角色，支持分布式集群方式部署。Consumer会从Broker服务器中获取到消息，并对消息进行相关业务处理。RocketMQ中的消息消费者是以消费者组(Consumer Group)的形式出现的。消费者组是同一类消费者的集合，这类Consumer消费的是同一个Topic类型的消息。从客户端的角度而言提供了两种消费形式：拉取式消费、推动式消费；从消费者组消费方式来说，有集群消费（Clustering）和广播消费（Broadcasting）

  - **拉取式消费**（Pull Consumer）：Consumer消费的一种类型，客户端通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由客户端控制。
  - **推动式消费**（Push Consumer）：Consumer消费的一种类型，该模式下Broker收到数据后会主动推送给消费端，该消费模式一般实时性较高。
  - **集群消费**（Clustering）：相同Consumer Group的每个Consumer实例平均分摊消息。
  - **广播消费**（Broadcasting）：相同Consumer Group的每个Consumer实例都接收全量的消息。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq-architecture.png" style="zoom:50%;" />

- **Name Server**：是一个Broker与Topic路由的注册中心，支持Broker的动态注册与发现。主要包括两个功能：

  - **Broker管理**：接受Broker集群的注册信息并且保存下来作为路由信息的基本数据；提供心跳检测机制，检查Broker是否还存活。
  - **路由信息管理**：每个NameServer中都保存着Broker集群的整个路由信息和用于客户端查询的队列信息。Producer和Conumser通过NameServer可以获取整个Broker集群的路由信息，从而进行消息的投递和消费。
  - NameServer通常也是集群的方式部署，各实例间相互不进行信息通讯。Broker是向每一台NameServer注册自己的路由信息，所以每一个NameServer实例上面都保存一份完整的路由信息。当某个NameServer因某种原因下线了，Broker仍然可以向其它NameServer同步其路由信息，Producer,Consumer仍然可以动态感知Broker的路由的信息。

- **Broker**：Broker主要负责消息的存储、投递和查询以及服务高可用保证。在Broker集群中实现了每个节点的主从集群。为了实现这些功能，Broker包含了以下几个重要子模块：

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq_brokerServer.png" style="zoom:50%;" />

  - Remoting Module：整个Broker的实体，负责处理来自clients端的请求。

  - Client Manager：负责管理客户端(Producer/Consumer)和维护Consumer的Topic订阅信息

  - Store Service：提供方便简单的API接口处理消息存储到物理硬盘和查询功能。

  - HA Service：高可用服务，提供Master Broker 和 Slave Broker之间的数据同步功能。

  - Index Service：根据特定的Message key对投递到Broker的消息进行索引服务，以提供消息的快速查询。

    

### 安装与启动

- 下载：https://rocketmq.apache.org/dowloading/releases/

- 支持Linux/Unix/MacOS系统，同时支持Windows（安装启动方式不一样），需要JDK1.8，Maven

  ```shell
  # 在下载rocketmq-all-4.9.2-source-release.zip后，解压
  unzip rocketmq-all-4.9.2-source-release.zip
  cd rocketmq-all-4.9.2/ 
  mvn -Prelease-all -DskipTests clean install -U # 编译，因为需要下载许多依赖，耗时较多
  cd distribution/target/rocketmq-4.9.2/rocketmq-4.9.2
  ```

- 启动

  ```shell
  # 修改配置: -Xms初始化堆大小,-Xmx最大堆内存大小,
  vi bin/runserver.sh
  vi bin/runbroker.sh
  JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g"
  JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
  
  # 第一步启动namesrv
  nohup sh bin/mqnamesrv &
  # 验证namesrv是否启动成功
  $ tail -f ~/logs/rocketmqlogs/namesrv.log  # ctrl+c退出日志显示
  
  # 第一步先启动broker
  $ nohup sh bin/mqbroker -n localhost:9876 &
  # 验证broker是否启动成功, 比如, broker的ip是192.168.1.2 然后名字是broker-a
  $ tail -f ~/logs/rocketmqlogs/Broker.log 
  ```

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq-install.png" style="zoom:50%;" />

- 测试发送和接收消息

  ```shell
  # 1.设置环境变量,告知客户端name server地址
  export NAMESRV_ADDR=localhost:9876
  # 2.使用安装包的Example作为客户端发送消息
  sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
  > SendResult [sendStatus=SEND_OK, msgId=2408845202900028B10ADF0D6...
  
  # 1.设置环境变量
  export NAMESRV_ADDR=localhost:9876
  # 2.接收消息
  sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
  > Consumer Started.
  > ConsumeMessageThread_5 Receive New Messages: [MessageExt [brokerName=wxks-Mac.local,...
  ```

- 关闭

  ```shell
  # 1.关闭NameServer
  sh bin/mqshutdown namesrv
  # 2.关闭Broker
  sh bin/mqshutdown broker
  ```





