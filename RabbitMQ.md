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
               * 2.队列里面的消息是否持久化，默认false消息存储在内存中
               * 3.该队列是否只供一个消费者进行消费，是否进行共享 true表示可以多个消费者消费
               * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
               * 5.其他参数
               */
              channel.queueDeclare(QUEUE_NAME,false,false,false,null);
              String message="hello world";
              /**
               * 发送一个消息
               * 1.发送到那个交换机,""表示默认
               * 2.路由的 key 是哪个
               * 3.其他的参数信息
               * 4.发送消息的消息体，字节流
               * */
              channel.basicPublish("", QUEUE_NAME,null, message.getBytes());
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
           * 3.消费者未成功消费的回调
           */
          channel.basicConsume(QUEUE_NAME,true, deliverCallback, cancelCallback);
      }
  
  }
  ```

  

### MQ Application -- Work Queue

- **Work Queue工作队列**：用来将耗时的任务分发给多个消费者（工作者）。有了工作队列，我们就可以将具体的工作放到后面去做，将工作封装为一个消息，发送到队列中，一个工作进程就可以取出消息并完成工作。如果启动了多个工作进程，那么工作就可以在多个进程间共享。这个概念也即我们说的异步，在项目中，有时候一个简单的Web请求，后台要做一系统的操作，这时候，如果后台执行完成之后再给前台返回消息将会导致浏览器页面等待从而出现假死状态。因此，通常的做法是，在这个Http请求到后台，后台获取到正确的参数等信息后立即给前台返回一个成功标志，然后后台异步地进行后续的操作。<img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/mq-workqueue.png" style="zoom:50%;" />

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

  
