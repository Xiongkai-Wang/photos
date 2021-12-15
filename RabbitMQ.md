## RabbitMQ



### MQ简介

- What？ **MQ(message queue)**，本质是个队列，FIFO 先入先出，队列中存放的内容是 message 。还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不用依赖其他服务。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-intro.png" style="zoom:33%;" />

- Why？

  - **流量消峰**：如果订单系统最多能处理一万次订单，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，可以取消这个限制，把一秒内下的订单分散成一段时间来处理，有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。
  - **应用解耦**：电商应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。
  - **异步处理**：有些服务间调用是异步的，例如A调用B，B需要花费很长时间执行，但是A需要知道B什么时候可以执行完。之前一般有两种方式，1--A每过一段时间去调用B的查询api查询；2--或者A提供一个callback api，B执行完之后调用 api 通知A服务。这两种方式都不是很优雅，需要服务之间提供耦合的功能。而使用消息队列，可以很方便解决这个问题，A调用B服务后，只需要监听B处理完成的消息，当B处理完成后，会发送一条消息给MQ，MQ 会将此消息转发给A服务。同样B服务也不用做这些操作。 A服务还能及时的得到异步处理成功的消息。

- 常见几种MQ的比较

  - **ActiveMQ**：

    - 单机吞吐量万级，时效性毫秒ms级，可用性高，基于主从架构实现高可用性，较低的概率丢失数据
    - 官方社区现在对 ActiveMQ 5.x 维护越来越少，高吞吐量场景较少使用

  - **Kafka**：

    - 最大的优点就是吞吐量高，单机写入系统吞吐量约在百万条/秒。时效性 ms 级可用性非常高，kafka 是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用。有优秀的第三方Kafka Web管理界面 Kafka-Manager。在日志领域比较成熟，被多家公司和多个开源项目使用。功能支持较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用
    - Kafka 单机超过 64 个队列/分区，Load 会发生明显的飙高现象，队列越多，load 越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试。支持消息顺序， 但是一台代理宕机后，就会产生消息乱序。

  - **RocketMQ**：

    - 单机吞吐量十万级，可用性非常高。分布式架构，消息可以做到0丢失。MQ 功能较为完善，而且分布式结构扩展性好。支持 10 亿级别的消息堆积，不会因为堆积导致性能下降。
    - 支持的客户端语言不多，目前是 java 及 c++，其中 c++不成熟。社区活跃度一般，没有在 MQ 核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码

  - **RabbitMQ**：

    - 由于 erlang 语言的高并发特性，性能较好。吞吐量到万级，MQ 功能比较完备，健壮、稳定、易用、跨平台、支持多种语言 如:Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript等，支持 AJAX 文档齐全。开源提供的管理界面非常棒，用起来很好用。社区活跃度高，更新频率相当高
    - 商业版需要收费

  - **总结**：Kafka适用于大数据，适合产生大量数据的互联网服务的数据收集业务，如果有日志采集功能， 肯定是首选kafka。RocketMQ适用于金融互联网领域对于可靠性要求很高的场景，比如电商里面的订单扣款；RoketMQ 在淘宝双11已经经历了多次考验。RabbitMQ结合 erlang 语言本身的并发优势，性能好时效性微秒级，社区活跃度也比较高，管理界面用起来十分方便，中小型公司优先选择功能比较完备的 RabbitMQ。

    

### RabbitMQ简介

- What？RabbitMQ 是一个消息中间件: 它接受并转发消息，它是在 AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。包含四大核心概念：

  - **生产者Producer**：产生数据发送消息的程序

  - **交换机Exchange**：是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

  - **队列Queue**：是 RabbitMQ 内部使用的一种数据结构，消息存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据

  - **消费者Consumer**：消费者大多时候是一个等待接收消息的程序。同一个应用程序既可以是生产者又是可以是消费者

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-rabbitmq.png" style="zoom:33%;" />

- Terminologies in RabbitMQ

  - **Connection**: publisher/consumer 和 broker 之间的TCP连接

  - **Channel**: 若每一次访问MQ都建立一个TCP Connection，消息量大时开销将是巨大的，效率也较低。Channel是在 connection内部建立的逻辑连接，channel复用了 Connection 的 TCP 连接。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-channel.png" style="zoom:33%;" />

  - **Virtual host**: 出于多租户和安全因素设计的考虑，在RabbitMQ server上可以创建多个虚拟的message broker，又叫做virtual hosts (vhosts)。每一个vhost本质上是一个mini-rabbitmq server，分别管理各自的exchange等。

  - **Broker**: 接收和分发消息的应用，RabbitMQ Server就是Message Broker

  - **Exchange**: message 到达 broker 的第一站，根据分发规则，匹配查询表中的routing key，分发消息到 queue 中去。常用的类型有:direct (point-to-point), topic (publish-subscribe) and fanout(multicast)

  - **Queue**: 消息最终被送到这里等待 consumer 取走

  - **Binding**: 就是将一个特定的Exchange和一个特定的Queue绑定起来。Exchange和Queue的绑定可以是多对多的关系

### RabbitMQ安装

- 官网地址：https://www.rabbitmq.com/download.html

- 将下载的文件erlang.rpm和rabbitmq.rpm上传到/usr/local/software 目录下

- 安装文件

  ```shell
  rpm -ivh erlang-21.3-1.el7.x86_64.rpm
  yum install socat -y
  rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm # -i install -vh 显示进度
  # docker
  docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management # --rm关闭容器后直接删除容器
  ```

- 常用命令

  ```shell
  chkconfig rabbitmq-server on # 添加开机启动 RabbitMQ 服务
  /sbin/service rabbitmq-server start # 启动服务
  /sbin/service rabbitmq-server status # 查看服务状态
  /sbin/service rabbitmq-server stop  # 停止服务
  
  rabbitmq-plugins enable rabbitmq_management # 开启 web 管理插件
  # 添加新的用户
  rabbitmqctl add_user admin password
  rabbitmqctl set_user_tags admin administrator # 设置角色
  rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*" # 设置用户权限，-p表示指定一个vhost，后面三个表示配置、写、读的权限
  rabbitmqctl list_users 
  
  # 重置
  rabbitmqctl stop_app # 先关闭
  rabbitmqctl reset
  rabbitmqctl start_app 
  ```




### MQ Application -- Hello World

- 最简单的情况，只有一个生产者和一个消费者。“ P”是我们的生产者，“ C”是我们的消费者。中间的框是一个队列-RabbitMQ 代表使用者保留的消息缓冲区。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-hello.png" style="zoom:50%;" />

- 引入java依赖：在创建的maven工程中pom文件中加入

  ```xml
  <dependencies>
      <dependency>
          <groupId>com.rabbitmq</groupId>
          <artifactId>amqp-client</artifactId>
          <version>5.8.0</version>
      </dependency>
      <dependency>
          <groupId>commons-io</groupId>
          <artifactId>commons-io</artifactId>
          <version>2.6</version>
      </dependency>
  </dependencies>
  ```

- 生产者

  ```java
  public class Producer {
      private final static String QUEUE_NAME = "hello";
      public static void main(String[] args) throws Exception {
          //创建一个连接工厂
          ConnectionFactory factory = new ConnectionFactory();
          factory.setHost("172.20.10.2");
          factory.setUsername("admin");
          factory.setPassword("password");
          //channel 实现了自动 close 接口 自动关闭 不需要显示关闭
          try(Connection connection = factory.newConnection();
              Channel channel = connection.createChannel()) {
              /**
               * 声明一个队列
               * 1.队列名称
               * 2.durable:队列里面的消息是否持久化，默认false消息存储在内存中
               * 3.exclusive:是否排他.如果一个队列声明为排他队列，该队列对首次声明它的连接可见，并在连接断开时自动删除，
               * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
               * 5.其他参数
               */
              channel.queueDeclare(QUEUE_NAME,false,false,false,null);
              String message="hello world";
              /**
               * 发送一个消息
               * 1.发送到那个交换机,""表示默认
               * 2.路由的key是哪个
               * 3.其他的参数信息,如消息持久化
               * 4.发送消息的消息体，字节流
               * */
              channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
              System.out.println("message sent");
          }
      }
  }
  
  ```

- 消费者

  ```java
  public class Consumer {
      private final static String QUEUE_NAME = "hello";
  
      public static void main(String[] args) throws Exception {
          ConnectionFactory factory = new ConnectionFactory();
          factory.setHost("172.20.10.2");
          factory.setUsername("admin");
          factory.setPassword("password");
          Connection connection = factory.newConnection();
          Channel channel = connection.createChannel();
          System.out.println("等待接收消息....");
          //推送的消息如何进行消费的接口回调
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message= new String(delivery.getBody());
              System.out.println(message);
          };
          //取消消费的一个回调接口 如在消费的时候队列被删除掉了
          CancelCallback cancelCallback = (consumerTag) -> {
              System.out.println("消息消费被中断");
          };
          /**
           * 消费者消费消息
           * 1.消费哪个队列
           * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
           * 3.成功消费回调
           * 4.未成功消费回调
           */
          channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
      }
  
  }
  ```
  
  

### MQ Application -- Work Queue

- **Work Queue工作队列**：用来将耗时的任务分发给多个消费者（工作者）。有了工作队列，我们就可以将具体的工作放到后面去做，将工作封装为一个消息，发送到队列中，一个工作进程就可以取出消息并完成工作。如果启动了多个工作进程，那么工作就可以在多个进程间共享。这个概念也即我们说的异步，在项目中，有时候一个简单的Web请求，后台要做一系统的操作，这时候，如果后台执行完成之后再给前台返回消息将会导致浏览器页面等待从而出现假死状态。因此，通常的做法是，在这个Http请求到后台，后台获取到正确的参数等信息后立即给前台返回一个成功标志，然后后台异步地进行后续的操作。<img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-workqueue.png" style="zoom:50%;" />

- 抽出channel生产过程到工具类

  ```java
  public class RabbitMqUtils {
      // 得到一个连接的 channel
      public static Channel getChannel() throws Exception{
          // 创建一个连接工厂
          ConnectionFactory factory = new ConnectionFactory();
          factory.setHost("172.20.10.2");
          factory.setUsername("admin");
          factory.setPassword("password");
          Connection connection = factory.newConnection();
          Channel channel = connection.createChannel();
          return channel;
      }
  }
  ```

- 生产者

  ```java
  public class Sender {
      private static final String QUEUE_NAME="Work Queue";
      public static void main(String[] args) throws Exception {
          try(Channel channel = RabbitMqUtils.getChannel();) {
              channel.queueDeclare(QUEUE_NAME,false,false,false,null);
              //从控制台当中接受信息
              Scanner scanner = new Scanner(System.in);
              // 模拟不断发送消息
              while (scanner.hasNext()){
                  String message = scanner.next();
                  channel.basicPublish("", QUEUE_NAME,null, message.getBytes());
                  System.out.println("message sent: "+message);
              }
          }
      }
  }
  ```

- 消费者

  ```java
  public class Worker1 {
      private static final String QUEUE_NAME="Work Queue";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          // 推送的消息如何进行消费的接口回调
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String receivedMessage = new String(delivery.getBody());
              System.out.println("message received: " + receivedMessage);
          };
          // 取消消费的一个回调接口 如在消费的时候队列被删除掉了
          CancelCallback cancelCallback = (consumerTag) -> {
              System.out.println(consumerTag + " message consuming is stopped...");
          };
          
        	System.out.println("Worker1 waiting for message......");
          channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
      }
  }
  ```

  

### 消息应答

- **What?**  Message Acknowledgment 消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。

- **Why？**为了保证消息在发送过程中不丢失。没有消息应答，RabbitMQ 一旦向消费者传递了一条消息，便立即将该消 息标记为删除。但是消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，就会丢失正在处理的消息。

- **应答方式**

  - 自动应答：消息发送后立即被认为已经传送成功。这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用

  - 手动应答：在接收到消息并且处理该消息之后应答。

    ```java
    channel.basicAck(deliveryTag, true)
    // deliveryTag:
    // multiple: true 代表批量应答 channel 上未应答的消息, flase只会应答tag=8的消息 5,6,7 这三个消息依然不会被确认收到消息应答 (比如说channel上有传送消息 5,6,7,8)
    ```

- 消息自动重新入队：如果消费者由于某些原因失去连接(连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队

- 具体操作：

  ```java
  public class Worker {
      private static final String QUEUE_NAME="Message Acknowledgment";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          // 推送的消息如何进行消费的接口回调
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String receivedMessage = new String(delivery.getBody());
              System.out.println("message received: " + receivedMessage);
            	// 手动应答
            	channel.basicAck(delivery.getEnvelope().getDeliveryTag(), true)
          };
          // 取消消费的一个回调接口 如在消费的时候队列被删除掉了
          CancelCallback cancelCallback = (consumerTag) -> {
              System.out.println(consumerTag + " message consuming is stopped...");
          };
          
        	System.out.println("Worker1 waiting for message......");
        	boolean autoAck = false;
          channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
      }
  }
  ```

  

### 队列和消息持久化

- **Why?**  消息应答是保证任务不丢失，而持久化是保障当 RabbitMQ 服务停掉以后消息生产者发送过来的消息不丢失。这需要队列和消息都标记为持久化

- **队列持久化**：

  ```java
  // 1.队列名称 2.队列里面的消息是否持久化 3.该队列是否只供一个消费者进行消费 4.是否自动删除 5.其他参数
  boolean durable = true;
  channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
  ```

- **消息持久化**：

  ```java
  String message = "hello";
  // 1.交换机 2.路由key 3.消息持久化 4.message内容
  channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
  ```

- 消费者预取值: 定义（队列想消费者传输消息）通道上允许的未确认消息的最大数量

  ```java
  int prefetchCount = 4;
  channel.basicQos(prefetchCount);
  // 增加预取将提高向消费者传递消息的速度;但是已传递但尚未处理的消息的数量也会增加，增加了消费者内存消耗
  ```

  

### 发布确认（Publisher Comfirms）

- **What？**：生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID，一旦消息被投递到所有匹配的队列之后，broker 就会发送一个包含ID的确认给生产者，这就使得生产者知道消息已经正确到达目的队列。

- **单独发布消息**：发布一个消息之后只有它被确认发布，后续的消息才能继续发布。

  ```java
  private static final Integer MESSAGE_COUNT =  10;
  public static void publishMessageIndividually() throws Exception { 
    try (Channel channel = RabbitMqUtils.getChannel()) {
      String queueName = UUID.randomUUID().toString(); 
      channel.queueDeclare(queueName, false, false, false, null); 
      //开启发布确认
      channel.confirmSelect();
      long begin = System.currentTimeMillis(); 
      for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        channel.basicPublish("", queueName, null, message.getBytes()); 
        //等待确认，服务端返回false或超时未返回，生产者可以消息重发
        boolean flag = channel.waitForConfirms();
        if(flag){
          System.out.println("消息发送成功"); 
        }
      }
      long end = System.currentTimeMillis();
      System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息,耗时" + (end - begin) +
  "ms"); }
  }
  ```

- **批量发布消息**: 发布一批消息然后一起确认，可以提高吞吐量

  ```java
  public static void publishMessageBatch() throws Exception { 
    try (Channel channel = RabbitMqUtils.getChannel()) {
      String queueName = UUID.randomUUID().toString(); 
      channel.queueDeclare(queueName, false, false, false, null); 
      channel.confirmSelect(); //开启发布确认
      int batchSize = 100; // 批量大小
      int outstandingMessageCount = 0; //未确认消息个数
      long begin = System.currentTimeMillis(); 
      for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        channel.basicPublish("", queueName, null, message.getBytes());
        outstandingMessageCount++;
        if (outstandingMessageCount == batchSize) {
          channel.waitForConfirms();
          outstandingMessageCount = 0;
        }
      }
      long end = System.currentTimeMillis();
      System.out.println("发布" + MESSAGE_COUNT + "个批量确认消息,耗时" + (end - begin) +
  "ms"); 
    }
  }
  ```

- **异步发布消息**：利用回调函数来达到消息可靠性传递的，这个中间件也是通过函数回调来保证是否投递成功

  ```java
  public static void publishMessageAsync() throws Exception { 
    try (Channel channel = RabbitMqUtils.getChannel()) {
      String queueName = UUID.randomUUID().toString(); 
      channel.queueDeclare(queueName, false, false, false, null); 
      channel.confirmSelect();
      
      // 消息确认成功的回调函数 1.消息标记 2.是否为批量确认
      ConfirmCallback ackCallback = (deliveryTag, multiple) -> { 
        System.out.println("发布的消息已确认，序列号:"+deliveryTag); 
      };
      // 消息确认失败的回调函数
      ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
        System.out.println("发布的消息未被确认，序列号:"+deliveryTag); 
      };
      //添加一个异步确认的监听器，监听哪些成功哪些失败: 1.确认收到消息的回调 2.未收到消息的回调
      channel.addConfirmListener(ackCallback, nackCallback); 
      
      long begin = System.currentTimeMillis();
      for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = "msg" + i; 
        channel.basicPublish("", queueName, null, message.getBytes()); 
      }
      long end = System.currentTimeMillis();
      System.out.println("发布" + MESSAGE_COUNT + "个异步确认消息,耗时" + (end - begin) + "ms");
    } 
  }
  ```

- 比较: 

  - 单独发布消息是同步等待确认，简单，但是吞吐量有限
  - 批量发布消息也是同步等待确认，吞吐量提升，但是一旦出现问题很难推断出是哪一条
  - 异步发布消息达到最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些



### 交换机 Exchange

- **What？**RabbitMQ 中，生产者生产的消息从不会直接发送到队列，相反，生产者只能将消息发送到交换机，交换机工作的内容非常简单，一方面它接收来自生产者的消息，另一方面将它们推入队列。交换机必须确切知道如何处理收到的消息。是应该把这些消 息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们。这就的由交换机的类型来决定。

- **类别**：扇出(fanout)、直接(direct)、主题(topic)、标题(headers) 。空字符串表示默认或无名称交换机。

  ```java
  // review
  //声明队列：1.队列名称 2.队列是否持久化 3.该队列是否排他 4.是否自动删除 5.其他参数
  channel.queueDeclare(QUEUE_NAME,false,false,false,null);
  //发送消息：1.发送到的交换机 2.路由key 3.其他参数信息(如发布的消息持久化) 4.消息体：字节流
  channel.basicPublish("", routingKey, null, message.getBytes());
  //消费消息：1.消费哪个队列 2.是否自动应答 3.消费成功的回调 4.消费未成功的回调 
  channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
  ```

- 绑定 Binding：是 exchange 和 queue 之间的桥梁，它告诉我们 exchange 和那个队列进行了绑定关系。
- **Fanout**：类型非常简单，是将接收到的所有消息广播到它绑定的所有队列中。
- **Direct**: Fanout 只能进行无意识的广播，Direct规定消息只去到它绑定的routingKey队列中去。
- **Topics**: 在Direct的基础上进一步对routingKey做出规定，实现更加灵活的方式。



### Publish/Subscribe

- What? 简单的发布订阅模式，利用fanout交换机将接收到的所有消息广播到它绑定的所有队列中。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-pub%3Asub.png" style="zoom:50%;" />

- 生产者

  ```java
  public class EmitLog {
      private static final String EXCHANGE_NAME = "logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
        	 // 声明交换机，指定交换机类型
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
          Scanner scanner = new Scanner(System.in);
          System.out.println("Please input message...");
          while (scanner.hasNext()) {
              String message = scanner.nextLine();
              channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
              System.out.println("Produced Message: " + message);
          }
      }
  }
  ```

- 消费者1

  ```java
  public class ReceiveLog01 {
      private static final String EXCHANGE_NAME = "logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
          // 生成一个临时队列
          String queue = channel.queueDeclare().getQueue();
          // 绑定交换机与队列，routingKey为空字符串
          channel.queueBind(queue, EXCHANGE_NAME, "");
          System.out.println("Waiting for message...");
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message = new String(delivery.getBody(), "UTF-8");
            	// 写入磁盘
              File file = new File("/Users/xiongkai/IdeaProjects/RabbitMQ/mq_info.txt");
              FileUtils.writeStringToFile(file,message,"UTF-8");
              System.out.println("write " + message + " into file successfully");
          };
          CancelCallback cancelCallback = consumerTag -> {};
          channel.basicConsume(queue, true, deliverCallback, cancelCallback);
      }
  }
  ```

- 消费者2

  ```java
  DeliverCallback deliverCallback = (consumerTag, delivery) -> {
      String message = new String(delivery.getBody(), "UTF-8");
      System.out.println("print message: " + message);
  };
  ```

  

  

### Routing

- What? 路由模式，选择性地接收消息。利用direct交换机实现消息只发送到它绑定的 routingKey 队列中去。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-routing.png" style="zoom:50%;" />

- 生产者

  ```java
  public class EmitLogDirect {
      private static final String EXCHANGE_NAME = "direct_logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
          Map<String, String> routingKey = new HashMap<>();
          routingKey.put("info", "normal information");
          routingKey.put("warning", "warning information");
          routingKey.put("error", "error information");
          routingKey.put("debug", "debug information");
  
          for (Map.Entry<String, String> entry: routingKey.entrySet()) {
              String key = entry.getKey();
              String message = entry.getValue();
              channel.basicPublish(EXCHANGE_NAME, key, null, message.getBytes(StandardCharsets.UTF_8));
              System.out.println("Produced Message: " + message);
          }
      }
  }
  ```

- 消费者1

  ```java
  public class ReceiveLogDirect01 {
      private static final String EXCHANGE_NAME = "direct_logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
          String queue = "directQueue01";
          channel.queueDeclare(queue, false, false, false, null);
          // 绑定指定的routingKey
          channel.queueBind(queue, EXCHANGE_NAME, "error");
          System.out.println("Waiting for message....");
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message = new String(delivery.getBody(), "UTF-8");
              message = message + "binding key: " + delivery.getEnvelope().getRoutingKey();
              // 将error信息写到磁盘
              File file = new File("/Users/xiongkai/IdeaProjects/RabbitMQ/error_info.txt");
              FileUtils.writeStringToFile(file,message,"UTF-8");
              System.out.println("write error into file successfully");
          };
          CancelCallback cancelCallback = consumerTag -> {};
          channel.basicConsume(queue, true, deliverCallback, cancelCallback);
      }
  }
  ```

- 消费者2

  ```java
  String queue = "directQueue02";
  // 绑定多个指定的routingKey
  channel.queueBind(queue, EXCHANGE_NAME, "info");
  channel.queueBind(queue, EXCHANGE_NAME, "warning");
  ```

  

### Topics

- **What？**发送到类型是 topic 交换机的消息的 routingKey 不能随意写，必须满足一定的要求，它必须是一个单词列表，以点号分隔开。*可以代替一个单词，#可以替代零个或多个单词。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-topics.png" style="zoom:50%;" />

- Examples:

  >Queue1: 
  >
  >​	 	\*.orange.\* 中间带 orange 带 3 个单词的字符串 
  >
  >Queue2: 
  >
  >​		\*.\*.rabbit 最后一个单词是 rabbit 的 3 个单词;  
  >
  >​		lazy.#第一个单词是 lazy 的多个单词
  >
  >quick.orange.rabbit  被队列 Q1 Q2 接收到
  >
  >lazy.orange.elephant 被队列 Q1Q2 接收到
  >
  >quick.orange.fox 被队列 Q1 接收到
  >
  >lazy.brown.fox  被队列 Q2 接收到
  >
  >lazy.pink.rabbit 虽然满足两个绑定但只被队列 Q2 接收一次
  >
  >quick.brown.fox 不匹配任何绑定不会被任何队列接收到会被丢弃
  >
  >quick.orange.male.rabbit 是四个单词,不匹配任何绑定会被丢弃
  >
  >lazy.orange.male.rabbit  四个单词但匹配Q2

- 生产者

  ```java
  public class EmitLogTopic {
      private static final String EXCHANGE_NAME = "topic_logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
          Map<String, String> routingKey = new HashMap<>();
          routingKey.put("quick.orange.rabbit", " 队列 Q1Q2 接收到");
          routingKey.put("lazy.orange.elephant", " 队列 Q1Q2 接收到");
          routingKey.put("quick.orange.fox", "队列 Q1 接收到");
          routingKey.put("lazy.brown.fox", " 队列 Q2 接收到");
          routingKey.put("quick.brown.fox", " 不匹配任何绑定不会被任何队列接收到会被丢弃");
          routingKey.put("lazy.orange.male.rabbit", " 四个单词 匹配Q2");
          routingKey.put("quick.orange.male.rabbit", " 四个单词不匹配任何绑定会被丢弃");
        
          for (Map.Entry<String, String> entry: routingKey.entrySet()) {
              String key = entry.getKey();
              String message = entry.getValue();
              channel.basicPublish(EXCHANGE_NAME, key, null, message.getBytes(StandardCharsets.UTF_8));
              System.out.println("Produced Message: " + message);
          }
      }
  }
  ```

- 消费者1

  ```java
  public class ReceiveLogTopic01 {
      private static final String EXCHANGE_NAME = "topic_logs";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
          String queue = "topicQueue01";
          channel.queueDeclare(queue, false, false, false, null);
          // 绑定指定topic的routingKey
          channel.queueBind(queue, EXCHANGE_NAME, "*.orange.*");
          System.out.println("Waiting for message....");
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message = new String(delivery.getBody(), "UTF-8");
              System.out.println(queue + message + " binding key: " + delivery.getEnvelope().getRoutingKey());
          };
          CancelCallback cancelCallback = consumerTag -> {};
          channel.basicConsume(queue, true, deliverCallback, cancelCallback);
      }
  }
  ```

- 消费者2

  ```java
  String queue = "topicQueue02";
  // 绑定指定topic的routingKey
  channel.queueBind(queue, EXCHANGE_NAME, "*.*.rabbit");
  channel.queueBind(queue, EXCHANGE_NAME, "lazy.#");      
  ```

  



### 死信队列

- **What？**  死信Dead Letter，顾名思义就是无法被消费的消息。某些时候由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信，死信队列就是接收这样的消息。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-deadLetter.png" style="zoom:33%;" />

- **死信来源**：消息TTL（Time To Live）过期；队列达到最大长度；消息被拒绝并且requeue=false

- 生产者：

  ```java
  public class Producer {
      private static final String NORMAL_EXCHANGE = "normal_exchange";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
          for (int i = 1; i < 11; i++) {
              String message = "message" + i;
              channel.basicPublish(NORMAL_EXCHANGE, "normal", null, message.getBytes(StandardCharsets.UTF_8));
              System.out.println("Produced " + message);
          }
      }
  }
  ```

- 普通消费者：

  ```java
  public class NormalConsumer {
      //普通交换机名称
      private static final String NORMAL_EXCHANGE = "normal_exchange";
      //死信交换机名称
      private static final String DEAD_EXCHANGE = "dead_exchange";
  
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          //声明死信和普通交换机 类型为 direct
          channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
          channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
  
          //声明死信队列
          String deadQueue = "deadQueue";
          channel.queueDeclare(deadQueue, false, false, false, null);
          channel.queueBind(deadQueue, DEAD_EXCHANGE, "dead"); //死信队列绑定死信交换机
  
          //正常队列绑定死信队列信息
          Map<String, Object> params = new HashMap<>();
          params.put("x-dead-letter-exchange", DEAD_EXCHANGE); //正常队列设置死信交换机 参数固定
          params.put("x-dead-letter-routing-key", "dead"); //正常队列设置死信routingKey 参数固定
          String normalQueue = "normalQueue";
          channel.queueDeclare(normalQueue, false, false, false, params);
          channel.queueBind(normalQueue, NORMAL_EXCHANGE, "normal");
  
          System.out.println("waiting for messages.....");
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message = new String(delivery.getBody(), "UTF-8");
              // 模拟普通消费者拒绝消费某个信息，从而让死信消费者消费
              if ("message5".equals(message)) {
                  System.out.println("Normal Consumer received " + message + " but refused");
                  channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
              } else {
                  System.out.println("Normal Consumer receive message: " + message);
              }
          };
          CancelCallback cancelCallback = consumerTag -> {};
          channel.basicConsume(normalQueue, false, deliverCallback,cancelCallback);
      }
  }
  ```

- 死信消费者：

  ```java
  public class DeadLetterConsumer {
      //死信交换机名称
      private static final String DEAD_EXCHANGE = "dead_exchange";
      public static void main(String[] args) throws Exception {
          Channel channel = RabbitMqUtils.getChannel();
          channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
          String deadQueue = "deadQueue";
          channel.queueDeclare(deadQueue, false, false, false, null);
          channel.queueBind(deadQueue, DEAD_EXCHANGE, "dead");
          System.out.println("waiting for messages.....");
          DeliverCallback deliverCallback = (consumerTag, delivery) -> {
              String message = new String(delivery.getBody(), "UTF-8");
              System.out.println("Dead Consumer receive dead letter: " + message);
          };
          CancelCallback cancelCallback = consumerTag -> {};
          channel.basicConsume(deadQueue, true, deliverCallback, cancelCallback);
      }
  }
  ```

  
