## Dubbo



###  Dubbo简介

- **What?** 是一款**高性能**、**轻量级**的开源**RPC**框架。简单来说 Dubbo 是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，以及分布式系统的服务治理方案。

- **RPC (Remote Procedure Call)** ：远程过程调用。是一种进程间通信方式，相对于本地过程调用。它允许程序调用另一个地址空间(通常是共享网络的另一台机器上)的过程或函数， 而不用程序员显式编码这个远程调用的细节。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/Dubbo-RPC.png" style="zoom:33%;" />

- **Dubbo主要功能**：

  - **透明化的远程方法调用**：就像调用本地方法一样调用远程方法，只需简单配置，没有任何API侵入。
  - **服务自动注册与发现**：不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者。
    - Dubbo 提供的是一种 Client-Based 的服务发现机制，通常还需要部署额外的第三方注册中心组件来协调服务发现过程，如常用的Zookeeper、Nacos、Consul。Dubbo推荐使用zookeeper作为服务中心。
  - **软负载均衡及容错机制**：可在内网替代F5等硬件负载均衡器，降低成本，减少单点。

- **Dubbo架构**：

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/Dubbo-architecture.png" alt="Dubbo-architecture" style="zoom:33%;" />

  - Provider: 暴露服务的服务提供方。Container: 服务运行的容器。
  - Consumer: 调用远程服务的服务消费方。
  - Registry: 服务注册与发现的注册中心。
  - Monitor: 统计服务的调用次调和调用时间的监控中心。

- **工作原理**：

  - 服务容器负责启动、加载、运行服务提供者。服务提供者在启动时，向注册中心注册自己提供的服务。
  - 服务消费者在启动时，向注册中心订阅自己所需的服务。
  - 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将推送变更数据给消费者。
  - 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
  - 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。



### Dubbo环境搭建

- 需要安装并启动启动Zookeeper作为服务器

  ```shell
  # 详见zookeeper讲解
  cd YourPathToZookeeper/apache-zookeeper-3.5.7-bin
  ./bin/zkServer.sh start
  ./bin/zkServer.sh status # 查看状态
  ```

- 下载安装dubbo-admin:  https://github.com/apache/dubbo-admin

  - dubbo-admin是为了让用户更好的管理监控dubbo服务，官方提供的一个可视化的监控程序。

  ```shell
  # 在自定义文件夹
  git clone https://github.com/apache/dubbo-admin.git
  
  # 在application.properties中修改registry地址，默认为127.0.0.1:2181
  vim ./dubbo-admin/dubbo-admin-server/src/main/resources/application.properties
  
  # build
  cd ./dubbo-admin # 修改后返回到dubbo-admin目录中
  mvn clean package -Dmaven.test.skip=true
  
  # 启动
  mvn --projects dubbo-admin-server spring-boot:run
  or
  cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.4.0.jar
  
  # 查看http://localhost:8080，默认username和password都是root
  ```

  

### 搭建一个Dubbo服务

- 问题概述：一个分布式项目的两个模块：用户模块和订单模块部署在不同的服务器中，在初始化订单时需要先获取用户收货地址，而用户收货地址是在用户模块的，因此出现需求：在订单模块远程调用用户模块的功能。

  - 订单模块：为Consumer，需要远程调用Provider的某个函数
  - 用户模块：为Provider，提供给Conusmer服务

- **分包**：当服务的消费者和提供者越来越多，将服务相关的实体类和接口等内容作为公共部分抽取成一个公共模块，提供者和消费者可以使用maven导入公共模块的jar包，从而使用公共模块的实体类、接口等。

- 公共接口-Dubbo-interface：

  ```java
  public class UserAddress implements Serializable {
      private Integer id;
      private String userAddress;
      private String userId;
      private String phoneNum;
      private String isDefault;
    	...
  }
  
  public interface UserService {
      public List<UserAddress> getUserAddressList(String userId);
  }
  ```

- Provider和Conusmer项目导入依赖

  ```xml
  <!--  引入Dubbo公共接口    -->
  <dependency>
    <groupId>com.xiongkai</groupId>
    <artifactId>Dubbo-interface</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
  <!--  引入Dubbo -->
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.6.2</version>
  </dependency>
  <!--  引入Zookeeper客户端 -->
  <dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.12.0</version>
  </dependency>
  ```

  

- Dubbo-Provider：具体实现接口

  ```java
  @Service
  public class UserServiceImpl implements UserService{
      // 服务提供者，实现接口提供服务
      public List<UserAddress> getUserAddressList(String userId) {
          UserAddress userAddress01 = new UserAddress(1, "Xi'an", "1001", "15511223344", "true");
          UserAddress userAddress02 = new UserAddress(2, "Changsha", "1001", "13311223344", "false");
          List<UserAddress> list = new ArrayList<UserAddress>();
          list.add(userAddress01);
          list.add(userAddress02);
          return list;
      }
  }
  
  // spring配置文件中实现dubbo连接
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
         http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
      <!-- 当前应用的名字   -->
      <dubbo:application name="dubbo-provider" ></dubbo:application>
      <!--指定注册中心的地址 -->
      <dubbo:registry address="zookeeper://127.0.0.1:2181" />
      <!--使用dubbo协议，将服务暴露在20880端口 -->
      <dubbo:protocol name="dubbo" port="28888" />
      <!-- 指定需要暴露的服务 interface表示接口位置，ref表示实现接口的位置-->
      <dubbo:service interface="com.xiongkai.dubbo.service.UserService"
                     ref="userServiceImpl" />
      <bean id="userServiceImpl" class="com.xiongkai.dubbo.service.UserServiceImpl"></bean>
  </beans>
        
  // 加载spring配置，提供服务
  public static class MainApplication {
          public static void main(String[] args) throws Exception{
              // 加载spring配置
              ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-provider.xml");
              context.start();
              System.in.read();
          }
      }
  ```

- Dubbo-Conusmer：

  ```java
  @Service
  public class OrderService {
      @Autowired
      UserService userService;
  
      // orderService初始化订单时需要首先查询用户的所有地址并返回，因此需要调用userService的相关功能
      public void initOrder(String userId){
          List<UserAddress> userAddressList = userService.getUserAddressList(userId);
          for (UserAddress userAddress: userAddressList) {
              System.out.println(userAddress.getUserAddress());
          }
      }
  }
  
  // spring配置文件中实现dubbo连接,指定需要代理的服务
   <context:component-scan base-package="com.xiongkai.dubbo.service"></context:component-scan>
  <dubbo:application name="dubbo-consumer"></dubbo:application>
  <!--指定注册中心的地址 -->
  <dubbo:registry address="zookeeper://127.0.0.1:2181" />
  <!-- 生成远程服务代理，可以和本地bean一样使用userService -->
  <dubbo:reference id="userService" interface="com.xiongkai.dubbo.service.UserService"></dubbo:reference>
  
  public class MainApplication {
      public static void main(String[] args) throws IOException {
        	// 加载spring配置
          ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-consumer.xml");
          OrderService service = context.getBean(OrderService.class);
          service.initOrder("1");
          System.in.read();
      }
  }
  ```

  

  