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
  
  # 先启动namesrv
  nohup sh bin/mqnamesrv &
  # 验证namesrv是否启动成功
  $ tail -f ~/logs/rocketmqlogs/namesrv.log  # ctrl+c退出日志显示
  
  # 启动broker
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



### 工作原理

- 首先要启动NameServer，NameServer起来后监听端口，等待Broker、Producer、Consumer连上来，相当于一个路由控制中心。
- 然后启动Broker，跟所有的NameServer保持长连接，注册信息，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储Topic的信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。
- Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。（收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic）
- Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。



### 消息发送与接收

- 创建maven工程，在pom.xml文件中添加依赖

  ```xml
  <dependency>
      <groupId>org.apache.rocketmq</groupId>
      <artifactId>rocketmq-client</artifactId>
      <version>4.9.2</version>
  </dependency>
  ```

- 三种简单的消息发送：

  - 发送**同步消息**：同步发送是指消息发送方发出一条消息后，会在收到服务端同步响应之后才发下一条消息的通讯方式。使用的比较广泛，比如：重要的消息通知，短信通知，短信营销系统等。
  - 发送**异步消息**：异步发送是指发送方发出一条消息后，不等服务端返回响应，接着发送下一条消息的通讯方式。通常用在对响应时间敏感的业务场景，即发送端不能容忍长时间地等待Broker的响应，例如，您视频上传后通知启动转码服务，转码完成后通知推送转码结果等。
  - **单向发送**消息：发送方只负责发送消息，不等待服务端返回响应且没有回调函数触发，即只发送请求不等待应答。主要用在不特别关心发送结果的场景如日志发送

- 发送同步消息

  ```java
  public class SyncProducer {
      public static void main(String[] args) throws Exception {
          // 实例化消息生产者Producer
          DefaultMQProducer producer = new DefaultMQProducer("MyProducerGroup01");
          // 设置NameServer的地址
          producer.setNamesrvAddr("localhost:9876");
          // 启动Producer实例
          producer.start();
          for (int i = 0; i < 100; i++) {
              // 创建消息，并指定Topic，Tag和消息体
              Message msg = new Message("TopicTest",
                      "TagA" ,
                      ("Hello Sync " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
              );
              // 发送消息到一个Broker
              SendResult sendResult = producer.send(msg);
              // 通过sendResult返回消息是否成功送达
              System.out.printf("%s%n", sendResult); // %s字符串类型占位符，%n 换行符
          }
          // 如果不再发送消息，关闭Producer实例。
          producer.shutdown();
      }
  }
  ```

- 发送异步消息

  ```java
  public class AsyncProducer {
      public static void main(String[] args) throws Exception {
          // 实例化消息生产者Producer
          DefaultMQProducer producer = new DefaultMQProducer("MyProducerGroup01");
          // 设置NameServer的地址
          producer.setNamesrvAddr("localhost:9876");
          // 启动Producer实例
          producer.start();
          producer.setRetryTimesWhenSendAsyncFailed(0);
  
          int messageCount = 100;
          // 根据消息数量实例化倒计时计算器: 需要所有消息都有回应以后才能关闭
          final CountDownLatch2 countDownLatch = new CountDownLatch2(messageCount); 
          for (int i = 0; i < messageCount; i++) {
              final int index = i;
              // 创建消息，并指定Topic，Tag, key和消息体
              Message message = new Message("TopicTest", "TagA", "OrderID188",
                      "Hello Async".getBytes(RemotingHelper.DEFAULT_CHARSET)
              );
              // SendCallback接收异步返回结果的回调
              producer.send(message, new SendCallback() {
                  public void onSuccess(SendResult sendResult) {
                      countDownLatch.countDown(); // 处理完一条消息倒数
                      System.out.printf("%-10d OK %s %n", index, sendResult.getMsgId());
                  }
                  public void onException(Throwable exception) {
                      countDownLatch.countDown(); // 处理完一条消息倒数
                      System.out.printf("%-10d Exception %s %n", index, exception);
                      exception.printStackTrace();
                  }
              });
          }
          // 等待5秒，等待异步发送结果。
          countDownLatch.await(5, TimeUnit.SECONDS);
          // 如果不再发送消息，关闭Producer实例。
          producer.shutdown();
      }
  }
  ```

- 单向发送消息

  ```java
  public class OnewayProducer {
      public static void main(String[] args) throws Exception{
          // 实例化消息生产者Producer
          DefaultMQProducer producer = new DefaultMQProducer("MyProducerGroup01");
          // 设置NameServer的地址
          producer.setNamesrvAddr("localhost:9876");
          // 启动Producer实例
          producer.start();
          for (int i = 0; i < 100; i++) {
              // 创建消息，并指定Topic，Tag和消息体
              Message msg = new Message("TopicTest", "TagA",
                      ("Hello OneWay " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
              );
              // 发送单向消息，没有任何返回结果
              producer.sendOneway(msg);
  
          }
          // 如果不再发送消息，关闭Producer实例。
          producer.shutdown();
      }
  }
  ```

- 消费者消费

  ```java
  public class Consumer {
      public static void main(String[] args) throws InterruptedException, MQClientException {
          // 实例化消费者
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("MyConsumerGroup01");
          // 设置NameServer的地址
          consumer.setNamesrvAddr("localhost:9876");
          // 订阅Topic，以及Tag来过滤需要消费的消息
          consumer.subscribe("TopicTest", "*");
          // 注册回调实现类来处理从broker拉取回来的消息
          consumer.registerMessageListener(new MessageListenerConcurrently() {
              @Override
              public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                  System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                  // 标记该消息已经被成功消费
                  return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
              }
          });
          // 启动消费者实例
          consumer.start();
          System.out.printf("Consumer Started.%n");
      }
  ```

  

### 集群搭建

- **单Master模式**：这是最简单但也是最危险的模式，一旦broker服务器重启或宕机，整个服务将不可用。 只建议在本地测试和开发可以选择这种模式。 

- **多master模式**：该模式是指所有节点都是master主节点（比如2个或3个主节点），没有slave从节点的模式。 

  - 配置简单，性能高；一个master节点的宕机或者重启（维护）对应用程序没有影响；当磁盘配置为RAID10时，消息不会丢失，因为RAID10磁盘非常可靠，即使机器不可恢复；、
  - 单台机器宕机时，本机未消费的消息，直到机器恢复后才会订阅，影响消息实时性

- **多Master多Slave模式-异步复制**：每个主节点配置多个从节点，多对主从。HA采用异步复制，主节点和从节点之间有短消息延迟（毫秒）

  - 即使磁盘损坏，也不会丢失极少的消息，不影响消息的实时性能；当主节点宕机时，消费者仍然可以消费从节点的消息，这个过程对应用本身是透明的，不需要人为干预；性能几乎与多Master模式一样高
  - 主节点宕机、磁盘损坏时，会丢失少量消息

- **多Master多Slave模式-同步双写**:每个master节点配置多个slave节点，有多对Master-Slave。HA采用同步双写，即只有消息成功写入到主节点并复制到多个从节点，才会返回成功响应给应用程序

  - 数据和服务都没有单点故障；在master节点关闭的情况下，消息也没有延迟；服务可用性和数据可用性非常高
  - 这种模式下的性能略低于异步复制模式（大约低 10%）；发送单条消息的RT略高，目前版本，master节点宕机后，slave节点无法自动切换到master

- **搭建多Master多Slave模式-同步双写的集群**：以两个master broker、每个master一个slave为例

  ```shell
  ## cd ./rocketmq-all-4.9.2/distribution/target/rocketmq-4.9.2/rocketmq-4.9.2
  ### 先启动nameServer 在不同的主机上都要启动
  $ nohup sh bin/mqnamesrv &
  
  ### 启动多个broker
  # 假设在A机器上启动第一个Master，NameServer的IP和端口
  $ nohup sh bin/mqbroker -n localhost:9876 -c ./conf/2m-2s-sync/broker-a.properties &
   
  # 假设在B机器上启动第二个Master，NameServer的IP和端口
  $ nohup sh bin/mqbroker -n localhost:9876 -c ./conf/2m-2s-sync/broker-b.properties &
   
  # 假设在C机器上启动第一个Slave，NameServer的IP和端口
  $ nohup sh bin/mqbroker -n localhost:9876 -c ./conf/2m-2s-sync/broker-a-s.properties &
   
  # 假设在D机启动第二个Slave，NameServer的IP和端口
  $ nohup sh bin/mqbroker -n localhost:9876 -c ./conf/2m-2s-sync/broker-b-s.properties &
  
  # 通过mqadmin命令查看集群
  ./bin/mqadmin clusterList
  ```

  ```shell
  # broker-.properties配置文件详解
  # Master和Slave是通过指定相同的config命名“brokerName”来配对的，master节点的brokerId必须为0，slave节点的brokerId必须大于0。
  #所属集群名字
  brokerClusterName=rocketmq-cluster
  #broker名字，注意此处不同的配置文件填写的不一样
  brokerName=broker-a #### broker-a  broker-b  broker-b 
  #0 表示 Master，>0 表示 Slave
  brokerId=0  #### 1 0 1
  #nameServer地址，分号分割
  namesrvAddr=localhost:9876
  #在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
  defaultTopicQueueNums=4
  #是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
  autoCreateTopicEnable=true
  #是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
  autoCreateSubscriptionGroup=true
  #Broker 对外服务的监听端口
  listenPort=10911 # 10912  10913  10914
  #删除文件时间点，默认凌晨 4点
  deleteWhen=04
  #文件保留时间，默认 48 小时
  fileReservedTime=120
  #commitLog每个文件的大小默认1G
  mapedFileSizeCommitLog=1073741824
  #ConsumeQueue每个文件默认存30W条，根据业务情况调整
  mapedFileSizeConsumeQueue=300000
  #destroyMapedFileIntervalForcibly=120000
  #redeleteHangedFileInterval=120000
  #检测物理文件磁盘空间
  diskMaxUsedSpaceRatio=88
  #存储路径
  storePathRootDir=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store
  #commitLog 存储路径
  storePathCommitLog=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store/commitlog
  #消费队列存储路径
  storePathConsumeQueue=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store/consumequeue
  #消息索引存储路径
  storePathIndex=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store/index
  # checkpoint文件存储路径
  storeCheckpoint=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store/checkpoint
  # abort文件存储路径
  abortFile=/Users/xiongkai/rocketmq/rocketmq-all-4.9.2/store/abort
  #限制的消息大小
  maxMessageSize=65536
  #flushCommitLogLeastPages=4
  #flushConsumeQueueLeastPages=2
  #flushCommitLogThoroughInterval=10000
  #flushConsumeQueueThoroughInterval=60000
  #Broker的角色： ASYNC_MASTER 异步复制Master SYNC_MASTER 同步双写Master  SLAVE
  brokerRole=SYNC_MASTER    #  SLAVE   SYNC_MASTER  SLAVE
  #刷盘方式：ASYNC_FLUSH 异步刷盘  SYNC_FLUSH 同步刷盘
  flushDiskType=SYNC_FLUSH
  #checkTransactionMessageEnable=false
  #发消息线程池数量
  #sendMessageThreadPoolNums=128
  #拉消息线程池数量
  #pullMessageThreadPoolNums=128
  ```

  

### 延时消息

- What? 延迟消息是指生产者发送消息发送消息后，不能立刻被消费者消费，需要等待指定的时间后才可以被消费。比如，用户下了一个订单之后，需要在指定时间内(例如15分钟)进行支付，如果时间到后还是未付款就取消订单释放库存。

- 生产者

  ```java
  public class ScheduledMessageProducer {
      public static void main(String[] args) throws Exception {
          // 实例化一个生产者来产生延时消息
          DefaultMQProducer producer = new DefaultMQProducer("Producer02");
          producer.setNamesrvAddr("localhost:9876");
          // 启动生产者
          producer.start();
          int totalMessagesToSend = 100;
          for (int i = 0; i < totalMessagesToSend; i++) {
              Message message = new Message("TestTopic02", ("Hello scheduled message " + i).getBytes());
              // 设置延时等级3,这个消息将在10s之后发送(目前只支持固定的几个时间)
              message.setDelayTimeLevel(3);
              // 发送消息
              producer.send(message);
          }
          // 关闭生产者
          System.out.println("sent all message");
          producer.shutdown();
      }
  }
  ```

- 消费者

  ```java
  public class ScheduledMessageConsumer {
      public static void main(String[] args) throws Exception {
          // 实例化消费者
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("Consumer02");
          consumer.setNamesrvAddr("localhost:9876");
          // 订阅Topics
          consumer.subscribe("TestTopic02", "*");
          // 注册回调实现类来处理从broker拉取回来的消息
          consumer.registerMessageListener(new MessageListenerConcurrently() {
              @Override
              public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> messages, ConsumeConcurrentlyContext context) {
                  for (MessageExt message : messages) {
                      // Print approximate delay time period
                      System.out.println("Receive message[msgId=" + message.getMsgId() + "] " + (System.currentTimeMillis() - message.getBornTimestamp()) + "ms later");
                  }
                  return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
              }
          });
          // 启动消费者
          consumer.start();
          System.out.println("consumer started \n");
      }
  }
  ```

  

### 过滤消息

- What? 在大多数情况下，TAG是一个简单而有用的设计，其可以来选择您想要的消息。但是限制是一个消息只能有一个标签，这对于复杂的场景可能不起作用。在这种情况下，可以使用SQL表达式筛选消息。SQL特性可以通过发送消息时的属性来进行计算。

- 需要在broker配置文件中 `enablePropertyFilter=true`

- 基本语法：RocketMQ只定义了一些基本语法来支持这个特性。你也可以很容易地扩展它。

  ```shell
  数值比较，比如：>，>=，<，<=，BETWEEN，=；
  字符比较，比如：=，<>，IN；
  IS NULL 或者 IS NOT NULL；
  逻辑符号 AND，OR，NOT；
  # 只有使用push模式的消费者才能用使用SQL92标准的sql语句，接口如下：
  ```

- 生产者：

  ```java
  public class filterProducer {
      public static void main(String[] args) throws Exception{
          DefaultMQProducer producer = new DefaultMQProducer("Producer03");
          producer.setNamesrvAddr("localhost:9876");
          producer.start();
          for (int i = 0; i < 10; i++)  {
              Message msg = new Message("TopicTest03",  "tag",
                      ("Hello RocketMQ " + i + "  ").getBytes(RemotingHelper.DEFAULT_CHARSET)
              );
              // 设置一些属性
              msg.putUserProperty("id", String.valueOf(i));
              SendResult sendResult = producer.send(msg);
              System.out.println(sendResult.getMessageQueue() + "," + String.valueOf(i));
  
          }
          producer.shutdown();
      }
  }
  ```

- 消费者：

  ```java
  public class filterConsumer {
      public static void main(String[] args) throws Exception {
          // 实例化消费者
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("Consumer03");
          consumer.setNamesrvAddr("localhost:9876");
          // 订阅Topic，以及Tag来过滤需要消费的消息
          //consumer.subscribe("TopicTest03", MessageSelector.bySql("id between 0 and 3"));
          consumer.subscribe("TopicTest03", MessageSelector.bySql("id > 1 and id < 6"));
  
          // 注册回调实现类来处理从broker拉取回来的消息
          consumer.registerMessageListener(new MessageListenerConcurrently() {
              @Override
              public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                  for (MessageExt msg : msgs) {
                      System.out.println("consumed："+ new String(msg.getBody()) + msg.getUserProperty("id"));
                  }
                  return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
              }
          });
          // 启动消费者实例
          consumer.start();
          System.out.printf("Consumer Started.%n");
      }
  }
  ```

  

### 顺序消息

- What? 消息有序指的是可以按照消息的发送顺序来消费(FIFO)。RocketMQ既可以严格的保证消息有序，可以分为分区有序或者全局有序。顺序消费的原理: 在默认的情况下消息发送会采取轮询方式把消息发送到不同的queue(分区队列)；而消费消息的时候从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。但是如果控制发送的顺序消息只依次发送到同一个queue中，消费的时候只从这个queue上依次拉取，则就保证了顺序。当发送和消费参与的queue只有一个，则是全局有序；如果多个queue参与，则为分区有序，即相对每个queue，消息都是有序的。

- 在注册消费监听时，使用`new MessageListenerOrderly()` 来保证顺序消费

- 下面用订单进行**分区有序**的示例。一个订单的顺序流程是：创建、付款、推送、完成。订单号相同的消息会被先后发送到同一个队列中，消费时，同一个OrderId获取到的肯定是同一个队列。

- 生产者：需要保证订单号相同的消息在同一个队列

  ```java
  public class Producer {
  
      public static void main(String[] args) throws Exception {
          DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
          producer.setNamesrvAddr("localhost:9876");
          producer.start();
          String[] tags = new String[]{"TagA", "TagC", "TagD"};
  
          // 订单列表
          List<OrderStep> orderList = new Producer().buildOrders();
  
          Date date = new Date();
          SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          String dateStr = sdf.format(date);
          for (int i = 0; i < 10; i++) {
              // 给消息体加个时间前缀
              String body = dateStr + " Hello RocketMQ " + orderList.get(i);
              Message msg = new Message("Topic04", tags[i % tags.length], "KEY" + i, body.getBytes());
  
              SendResult sendResult = producer.send(msg, new MessageQueueSelector() { // 队列选择器
                  @Override
                  public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                      Long id = (Long) arg;  // 根据订单id选择发送的queue
                      long index = id % mqs.size();
                      return mqs.get((int) index);
                  }
              }, orderList.get(i).getOrderId()); // 订单id的参数传给Object arg
  
              System.out.println(String.format("SendResult queueId:%d, body:%s",
                      sendResult.getMessageQueue().getQueueId(),
                      body));
          }
  
          producer.shutdown();
      }
  
      /*订单步骤类*/
      private static class OrderStep {
          private long orderId; // id
          private String desc;  // description
  
          public long getOrderId() {
              return orderId;
          }
  
          public void setOrderId(long orderId) {
              this.orderId = orderId;
          }
  
          public String getDesc() {
              return desc;
          }
  
          public void setDesc(String desc) {
              this.desc = desc;
          }
  
          @Override
          public String toString() {
              return "OrderStep{" +
                      "orderId=" + orderId +
                      ", desc='" + desc + '\'' +
                      '}';
          }
      }
  
      /* 随意生成模拟订单数据 */
      private List<OrderStep> buildOrders() {
        	// 
          List<OrderStep> orderList = new ArrayList<OrderStep>();
  
          OrderStep orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109001L);
          orderDemo.setDesc("创建");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109002L);
          orderDemo.setDesc("创建");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109001L);
          orderDemo.setDesc("付款");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109003L);
          orderDemo.setDesc("创建");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109002L);
          orderDemo.setDesc("付款");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109003L);
          orderDemo.setDesc("付款");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109002L);
          orderDemo.setDesc("完成");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109001L);
          orderDemo.setDesc("推送");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109003L);
          orderDemo.setDesc("完成");
          orderList.add(orderDemo);
  
          orderDemo = new OrderStep();
          orderDemo.setOrderId(20210109001L);
          orderDemo.setDesc("完成");
          orderList.add(orderDemo);
  
          return orderList;
      }
  }
  ```

  

- 消费者：

  ```java
  public class ConsumerInOrder {
  
      public static void main(String[] args) throws Exception {
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_3");
          consumer.setNamesrvAddr("127.0.0.1:9876");
          /* 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费  */
          consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
          consumer.subscribe("Topic04", "TagA || TagC || TagD");
          // 注册回调实现类来处理从broker拉取回来的消息
          consumer.registerMessageListener(new MessageListenerOrderly() {
              Random random = new Random();
              @Override
              public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                  context.setAutoCommit(true);
                  for (MessageExt msg : msgs) {
                      // 可以看到每个queue有唯一的consume线程来消费, 订单对每个queue(分区)有序
                      System.out.println("consumeThread=" + Thread.currentThread().getName() + "queueId=" + msg.getQueueId() + ", content:" + new String(msg.getBody()));
                  }
                  try {
                      //模拟业务逻辑处理
                      TimeUnit.SECONDS.sleep(random.nextInt(10));
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
                  return ConsumeOrderlyStatus.SUCCESS;
              }
          });
          consumer.start();
          System.out.println("Consumer Started.");
      }
  }
  ```

  



### 事务消息

- **What？** 它可以被认为是一个两阶段的提交消息实现，以确保分布式系统中的最终一致性。事务性消息保证了本地事务的执行和消息的发送能够以原子的方式进行。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq-transaction01.png" style="zoom:50%;" />

- RocketMQ中**事务消息的实现流程**：事务消息的大致方案主要分为两个流程：正常事务消息的发送及提交、事务消息的补偿流程。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/rocketmq-transaction.png" style="zoom:50%;" />

  - **事务消息的三种状态**：
    - TransactionStatus.**Commit**Transaction: 提交事务，它允许消费者消费此消息
    - TransactionStatus.**Rollback**Transaction: 回滚事务，它代表该消息将被删除，不允许被消费
    - TransactionStatus.**Unknown**: 中间状态，它代表需要检查消息队列来确定状态。
  - **正常事务消息的发送及提交**：(1) 发送消息（half消息）。(2) 服务端响应消息写入结果。(3) 根据发送结果执行本地事务（如果写入失败，此时half消息对业务不可见，本地逻辑不执行）。(4) 根据本地事务状态执行Commit或者Rollback（Commit操作生成消息索引，消息对消费者可见）
  - **事务消息的补偿流程**：(1) 对没有Commit/Rollback的事务消息（UNKNOWN状态的消息），从服务端发起一次回查 (2) Producer收到回查消息，检查回查消息对应的本地事务的状态 (3) 根据本地事务状态，重新Commit或者Rollback

- **具体实现**：

  - 创建事务的生产者：

    ```java
    public class Producer {
        public static void main(String[] args) throws MQClientException, InterruptedException {
            //创建事务监听器
            TransactionListener transactionListener = new TransactionListenerImpl();
            TransactionMQProducer producer = new TransactionMQProducer("MyGroup");
            producer.setNamesrvAddr("localhost:9876");
            //生产者设置监听器
            producer.setTransactionListener(transactionListener);
            //启动消息生产者
            producer.start();
            String[] tags = new String[]{"TagA", "TagB", "TagC"};
            for (int i = 0; i < 3; i++) {
                try {
                    Message msg = new Message("TransactionTopic", tags[i % tags.length], "KEY" + i,
                            ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                    SendResult sendResult = producer.sendMessageInTransaction(msg, null);
                    System.out.printf("%s%n", sendResult);
                    TimeUnit.SECONDS.sleep(1);
                } catch (MQClientException | UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
            producer.shutdown();
        }
    }
    ```

  - 实现事务的监听接口

    ```java
    public class TransactionListenerImpl implements TransactionListener {
        @Override
        public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
            System.out.println("执行本地事务");
            if (StringUtils.equals("TagA", msg.getTags())) {
                return LocalTransactionState.COMMIT_MESSAGE;
            } else if (StringUtils.equals("TagB", msg.getTags())) {
                return LocalTransactionState.ROLLBACK_MESSAGE;
            } else {
                return LocalTransactionState.UNKNOW;
            }
        }
    
        @Override
        public LocalTransactionState checkLocalTransaction(MessageExt msg) {
            System.out.println("MQ检查消息Tag【"+msg.getTags()+"】的本地事务执行结果");
            return LocalTransactionState.COMMIT_MESSAGE;
        }
    }
    ```

  - 消费者

    ```java
    public class TransactionConsumer {
        public static void main(String[] args) throws MQClientException {
            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("MyGroup11");
            consumer.setNamesrvAddr("localhost:9876");
            consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
            consumer.subscribe("TransactionTopic", "*");
            consumer.registerMessageListener(new MessageListenerOrderly() {
                private Random random = new Random();
                @Override
                public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                    //设置自动提交
                    context.setAutoCommit(true);
                    for (MessageExt msg:msgs){
                        System.out.println(msg+ " , content : "+ new String(msg.getBody()) + new String(msg.getTags()));
                    }
                    try {
                        //模拟业务处理
                        TimeUnit.SECONDS.sleep(random.nextInt(5));
                    }catch (Exception e){
                        e.printStackTrace();
                        return  ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                    }
                    return ConsumeOrderlyStatus.SUCCESS;
                }
            });
            consumer.start();
            System.out.println("consumer started.");
        }
    }
    ```

    
