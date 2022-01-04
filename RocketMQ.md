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

  







