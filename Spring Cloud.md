## Spring Cloud



### 微服务介绍

- **软件架构演进**：软件架构的发展经历了从单体架构、垂直架构、分布式架构、SOA架构到微服务架构的过程。

- **单体架构**：Web应用程序发展的早期，将所有的功能模块打包到一起并放在一个web容器中运行，所有功能模块使用同一个数据库。简单，开发部署都很方便，小型项目首选

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-Monolithic.png" style="zoom:50%;" />

  - 项目启动慢、可靠性差、可伸缩性差、扩展性和可维护性差、性能低

- **垂直架构**：垂直架构是指将单体架构中的多个模块拆分为多个独立的项目，形成**多个独立的单体架构**。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-vertical.png" style="zoom:50%;" />

  - 重复功能太多

- **分布式架构**：是指在垂直架构的基础上，将公共业务模块抽取出来，作为独立的服务，供其他调用者消费（**实现RPC**），以实现服务的共享和重用。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-distributed.png" style="zoom:50%;" />

  - 服务提供方一旦产生变更，所有消费方都需要变更

- **SOA架构**：Service-Oriented Architecture，面向服务的架构，是一种**面向服务**的架构，基于分布式架构，它将不同业务功能按服务进行拆分，并通过这些服务之间定义良好的接口和协议联系起来。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-soa.png" style="zoom:50%;" />

  - 基于SOA的架构思想，将重复公用的功能抽取为组件，以服务的方式向各各系统提供服务。
  - 将ESB作为系统与服务之间通信的桥梁。ESB(Enterparise Servce Bus) 企业服务总线，服务中介。主要是提供了一个服务与服务之间的交互。ESB 包含的功能如:负载均衡，流量控制，加密处理，服务的监控 ，异常处理，监控告急等等。

- **微服务架构**：基于SOA架构的思想，为了满足移动互联网对大型项目及多客户端的需求，**对服务层进行细粒度的拆分**，所拆分的每个服务只完成某个特定的业务功能，服务的粒度很小，所以称为微服务架构。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-microservice.png" style="zoom:50%;" />

  - 拆分粒度更小、服务更独立、耦合度更低
  - 架构非常复杂，运维、监控、部署难度提高

- **微服务结构**：

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-micro-archi.png" style="zoom:33%;" />

  - 注册中心：注册并维护远程服务及服务提供者的地址，供服务消费者发现和调用，为保证可用性，通常基于分布式 kv 存储器来实现。如Eureka、Zookeeper
  - 配置中心：把项目中各种配置、各种参数、各种开关，全部都放到一个集中的地方进行统一管理，并提供一套标准的接口。当各个服务需要获取配置的时候，就来配置中心的接口拉取。
  - 服务网关：所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理非业务功能。介于客户端与微服务之间的网关层，可以理解为门卫的角色，以确保服务提供者对客户端的透明，这一层可以进行反向路由、安全认证、灰度发布、日志监控等前置动作；
  - 服务监控与服务追踪：对服务消费者与提供者之间的调用情况进行监控和数据展示；记录对每个请求的微服务调用完整链路，以便进行问题定位和故障分析；
  - 服务治理：服务治理就是通过一系列的手段来保证在各种意外情况下，服务调用仍然能够正常进行，这些手段包括熔断、隔离、限流、降级、负载均衡等。 



### Spring Cloud简介

- **What?** 是目前国内使用最广泛的微服务框架。Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

  - SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-intro.png" style="zoom:33%;" />

- **SpringCloud体系**：

  - 虽然 Spring Cloud 提供了非常强大的功能，但是它并不提供所有的实现，而是通过 Spring Cloud Common子项目，定义了统一的抽象 API。不同厂商结合其自身的中间件，提供自己的 Spring Cloud 套件，例如说：

    - Netflix 结合自己的 Eureka、Ribbon、Hystrix 等开源中间件，实现了 Spring Cloud Netflix。目前最为流行。

    - Alibaba 结合自己的 Nacos、Dubbo、Sentinel 等开源中间件，实现了Spring Cloud Alibaba。

      <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-overview.png" style="zoom:33%;" />

- Spring Cloud与Springboot的**版本匹配**

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-boot-version.png" style="zoom:33%;" />



### 注册中心--Eureka

- **What?** Eureka是注册中心

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-eureka.png" style="zoom:50%;" />

- 工作原理

  - 服务提供者启动时向eureka注册自己的信息，eureka保存提供者信息，消费者根据服务名称向eureka拉取提供者信息
  - 服务消费者需要调用服务时，利用负载均衡算法，从服务列表中挑选一个
  - 服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态，eureka会更新记录服务列表信息，心跳不正常会被剔除

- 搭建**EurekaServer**：

  - 创建项目，引入`spring-cloud-starter-netflix-eureka-server`的依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    ```

  - 在启动类中添加`@EnableEurekaServer`注解

  - 在application.yml配置文件中，添加下面的配置

    ```xml
    server:
    		port:10086
    spring: 
    		application:
    				name: eurekaserver 
    eureka:
    		client: 
    				service-url:
    						defaultZone: http://127.0.0.1:10086/eureka/
    ```

- 在**提供者**和**消费者**中注册：

  - 引入`spring-cloud-starter-netflix-eureka-client`的依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```

  - 在application.yml配置文件中，添加下面的配置

    ```xml
    spring: 
    		application:
    				name: providerServiceName # or comsumerServiceName
    eureka:
    		client: 
    				service-url:
    						defaultZone: http://127.0.0.1:10086/eureka/
    ```

  

### 负载均衡--Ribbon

- **What?** : 是Netflix开发的一个负载均衡组件，它在服务体系中起着重要作用，比如远程调用的组件Feign就集成、封装了Ribbonn这个组件。Ribbon是一个客户端负载均衡器，它赋予了客户端一些支配HTTP与TCP行为的能力。

- **工作流程**：

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-ribbon-principle.png" style="zoom:50%;" />

- **负载均衡策略**：不同的Rule都是实现`IRule`接口

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-ribbon-rules.png" style="zoom:50%;" />

- **如何配置**：

  - 配置文件方式：在consumer的`application.yml`文件中。

    - 针对某一个服务的配置。
    - 直观，方便，无需重新打包发布， 但是无法做全局配置

    ```xml
    userservice:
    	ribbon:
    		NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    ```

  - 代码方式：在consumer的启动类中定义一个新的`IRule`。

    - 是全局的配置，调用的所有服务都会使用
    - 配置灵活，但修改时需要重新打包发布

    ```java
    @Bean
    public IRule randomRule(){ 
      return new RandomRule();
    }
    ```

- **懒加载与饥饿加载**：

  - 懒加载：即第一次访问时才会去创建负载均衡，第一次的请求时间会很长。默认加载方式

  - 饥饿加载：在项目启动时就会创建

    ```xml
    ribbon:
    		eager-load:
    				enable: true
    				clients: 
    						- userservice
    						- orderservice
    ```



### 注册和配置中心--Nacos

- **What?**  Nacos是Alibaba的产品，现在是SpringCloud中的一个组件。相比Eureka功能更加丰富，受欢迎程度越来越高。Nacos 致力于帮助发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助快速实现动态服务发现、服务配置、服务元数据及流量管理。

- 下载与安装：需要java1.8+和maven3.2+的环境。https://nacos.io/zh-cn/docs/quick-start.html

  ```shell
  # 下载
  git clone https://github.com/alibaba/nacos.git
  cd nacos/
  # maven 安装编译
  mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U 
  # 
  cd distribution/target/nacos-server-$version/nacos/bin
  
  # 启动和关闭服务器 (Linux和MacOS)
  sh startup.sh -m standalone # standalone表示非集群默认
  sh shutdown.sh
  
  # Nacos默认使用8848，可以在conf目录下的application.properties文件中修改
  # 启动后打开：http://localhost:8848/nacos/ 可以看到nacos控制台，用户密码默认为nacos，可在配置文件中修改
  ```

- 在Spring Cloud项目中配置Nacos：

  - 在项目中添加Alibaba依赖：

    ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId> 
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>		
      	<version>2.2.6.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
    ```

  - 在各个服务中添加Nacos服务注册依赖: （如有，注释服务中的eureka依赖）

    ```xml
    <!-- nacos客户端依赖 --> 
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> 
    </dependency>
    ```

  - 修改各个服务中的配置文件`application.yml`

    ```xml
    spring:
    	cloud:
    		nacos:
    			server-addr: localhost:8848
    ```

- **Nacos作为注册中心**：

  - ***Nacos服务分级存储模型***：

    ​	<img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-nacos.png" style="zoom:50%;" />

    - 一级是服务，例如userservice；二级是集群，例如上海集群；三级是实例，例如上海集群的某台部署了userservice的服务器

    - 服务跨集群调用问题：服务调用尽可能选择本地集群的服务，跨集群调用延迟较高；本地集群不可访问时，再去访问其它集群

    - 配置服务集群属性：修改`application.yml`

      ```xml
      spring:
      	cloud:
      		nacos:
      			server-addr: localhost:8848
      			discovery:
      				cluster-name: Shanghai
      ```

  - ***Nacos负载均衡***：

    - 当我们配置了服务的集群属性后，可以使用Nacos负载均衡策略

      ```xml
      userservice:
      	ribbon:
      		NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
      ```

    - 该策略：尽可能选择本地集群的服务，确定了可用实例列表后，再采用随机负载均衡挑选实例

  - ***Nacos环境隔离***：Nacos中服务存储和数据存储的最外层都是一个名为namespace的东西，用来做最外层隔离

    ​	<img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-nacos-namespace.png" style="zoom:50%;" />

    - 可以在在Nacos控制台可以创建namespace，用来隔离不同环境，比如开发生产和测试环境等。每个namespace都有唯一id

    - 在服务中配置namespace：服务设置namespace时要写id而不是名称。只有在同一namespace才可以访问到该服务。

      ```xml
      spring:
      	cloud:
      		nacos:
      			server-addr: localhost:8848
      			discovery:
      				cluster-name: Shanghai
      				namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 
      				ephemeral: false
      ```

  - ***Nacos与Eureka的比较***：

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-nacos-eureka.png" style="zoom:50%;" />

    - 相同点：都支持服务注册和服务拉取；都支持服务提供者心跳方式做健康检测
    - 不同点：
      - Nacos支持服务端主动检测提供者状态:临时实例采用心跳模式，非临时实例采用主动检测模式（临时节点配置如上ephemeral）
      - Nacos临时实例心跳不正常会被剔除，非临时实例则不会被剔除
      - Nacos支持服务列表变更的消息推送模式，服务列表更新更及时

- **Nacos作为配置中心**：

  - ***配置Nacos配置中心***：

    - 引入Nacos配置管理依赖

      ```xml
      <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId> 
      </dependency>
      ```

    - 在服务中添加一个另一个文件-bootstrap.yml，这个文件是引导文件，优先级高于 application.yml。

      <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-nacos-configLoad.png" style="zoom:50%;" />

      ```xml
      spring: 
      	application:
      		name: userservice 
      	profiles:
      		active: dev 
      	cloud:
      		nacos:
      			server-addr: localhost:8848 
      			config:
      				file-extension: yaml 
      ```

  - ***统一配置的使用***：

    - 新建配置：在Nacos console中新建

      ​	<img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-nacos-config-new.png" style="zoom:50%;" />

      - DataID创建规则：springApplicationName-profileActive.fileExtension 例如:userservice-dev.yaml

    - 服务引入Nacos配置：

      ```java
      @RestController 
      @RequestMapping("/user") 
      @RefreshScope
      public class UserController {
      	// 用Value注解注入nacos中的配置属性 
        @Value("${pattern.dateformat}") 
        private String dateformat;
      	// 编写controller，通过日期格式化器来格式化现在时间并返回 
        @GetMapping("now")
      	public String now(){
      		return LocalDate.now().format(DateTimeFormatter.ofPattern(dateformat, Locale.CHINA));
      	}
      }

    - 配置自动刷新：

      - 方式一：在@Value注入的变量所在类上添加注解@RefreshScope

      - 方式二：使用@ConfigurationProperties注解

        ```java
        @Component
        @Data
        @ConfigurationProperties(prefix = "pattern") 
        public class PatternProperties {
        	private String dateformat;
        }
        ```

  - **多环境共享配置**：

    - 当DataID为springApplicationName.fileExtension的格式，比如userservice.yaml。那么这个配置会被所有环境的userservice服务加载。
    - 优先级：非共享配置 > 共享配置 



### 统一网关--Gateway

- **What？** 网关是一个处于应用程序或服务之前的系统，用来管理授权、访问控制和流量限制等，这样服务就被网关保护起来，对所有的调用者透明。因此，隐藏在网关后面的业务系统就可以专注于创建和管理服务，而不用去处理这些策略性的基础设施。

  - Spring Cloud Gateway是Spring官方开发的网关，旨在为微服务架构提供一种简单而有效的统一的API路由管理方式。Spring Cloud Gateway作为Spring Cloud生态系中的网关，目标是替代ZUUL，其不仅提供统一的路由方式，并且基于Filter链的方式提供网关基本的功能，例如：安全，监控/埋点，和限流等。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-gateway-.png" style="zoom:50%;" />

- 网关的**职能**：

  - 对用户请求做身份认证、权限校验：确保请求来源的安全性
  - 将用户请求路由到微服务，并实现负载均衡
  - 对用户请求做限流：限制超过系统能力后的请求

- **Gateway与Zuul的比较**：在SpringCloud中网关的实现包括Gateway和Zuul

  - SpringCloud中所集成的Zuul1.0版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。servlet是一个简单的网络IO模型，当请求进入servlet container时，servlet container就会为其绑定一个线程，在并发不高的场景下这种模型是适用的，但是一旦并发上升，线程数量就会上涨，严重影响请求的处理时间。
    - 虽然Zuul 2.0开始使用了Netty，并且有了大规模Zuul 2.0集群部署的成熟案例。但是，SpringCloud官方已经没有集成该版本的计划了。
  - 而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程的实现，底层使用的是Netty，Netty是目前业界认可的高性能的通信框架，具备更好的性能。

- **Gateway中的核心概念**；

  - **Route(路由)**：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由，目标URI会被访问。
  - **Predicate(断言)**：断言用来匹配来自http请求的任何内容，如：请求头和请求参数。断言的类型是一个ServerWebExchange。
  - **Filter(过滤器)**：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前后对请求进行修改。

- **搭建网关服务**

  - 创建新的module，引入SpringCloudGateway的依赖和nacos的服务发现依赖

    ```xml
     <!--网关依赖--> 
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId> 
    </dependency>
    <!--nacos服务发现依赖-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> 
    </dependency>
    ```

  - 编写路由配置及nacos地址

    ```yml
    server:
    	port: 10010 # 网关端口
    spring: 
    	application:
    		name: gateway # 服务名称 
    	cloud:
    		nacos:
    			server-addr: localhost:8848 # nacos地址
    		gateway:
    			routes: # 网关路由配置
    				- id: user-service # 路由id，自定义，只要唯一即可
    					# uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
    					uri: lb://userservice # 路由的目标地址 lb指负载均衡，后面跟服务名称 
    					predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
    						- Path=/user/** # Path表示按照路径匹配，只要以/user/开头就符合要求
    					filters: # 过滤器
    						- AddRequestHeader=X-Request-red, blue # 添加请求头
    						- SaveSession # 保存session
    			default-filters: # 默认过滤器，会对所有的路由请求都生效
    				- RedirectTo=302, https://acme.org
    ```

- Gateway中的**路由断言**：

  - 在配置文件中写的断言规则只是字符串，这些字符串会被**路由断言工厂Route Predicate Factory**读取并处理，转变为路由判断的条件。

  - SpringCloudGateway中的11种断言工厂：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-gateway-predicate.png" style="zoom:50%;" />

- Gateway中的**过滤器**：

  - GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理。Gateway中提供31种过滤器。

    >AddRequestHeader 给当前请求添加一个请求头
    >
    >RemoveRequestHeader 移除请求中的一个请求头
    >
    >AddResponseHeader 给响应结果中添加一个响应头
    >
    >RequestRateLimiter  限制请求的流量
    >
    >SaveSession 保存session
    >
    >RedirectTo 重定向

  - **默认过滤器**：作用是设置统一的过滤器，会对该服务所有的路由请求都生效

    - 设置的位置与routes平级

  - **全局过滤器** ：全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。 区别在于GatewayFilter通过配置定义，处理逻辑是固定的。而GlobalFilter的逻辑可以自己写代码实现。 定义方式是实现GlobalFilter接口。

    ```java
    @Order(-1)
    @Component
    public class AuthorizeFilter implements GlobalFilter {
    		@Override
    		public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) { 
          	// 1.获取请求参数
    				MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
    				// 2.获取authorization参数
    				String auth = params.getFirst("authorization");
    				// 3.校验
            if ("authorizationCode".equals(auth)) {
            		// 放行
            		return chain.filter(exchange);
            }
            // 4.拦截
            // 4.1.禁止访问
    				exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN); 
          	// 4.2.结束处理
          	return exchange.getResponse().setComplete();
    		} 
    }
    ```

  - 过滤器的优先级：

    - 请求进入网关会碰到三类过滤器:当前路由的GatewayFilter、DefaultFilter、GlobalFilter。请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链中，排序后依次执行每个过滤器

      <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-gateway-filter.png" style="zoom:50%;" />

    - 每一个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前**

      - GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值
      - 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增
      - 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。



### Http客户端--Feign

- **What?**  Feign是一个声明式的http客户端，其作用就是帮助我们优雅的实现http请求的发送，实现服务间的远程调用。Spring Cloud OpenFeign是基于Netflix feign实现，整合了Spring Cloud Ribbon和Spring Cloud Hystrix。Spring Cloud还对Feign进行了增强，使Feign支持了Spring MVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。

- **如何使用Feign**：

  - 引入依赖：

    ```xml
     <dependency> 
       <groupId>org.springframework.cloud</groupId> 
       <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    ```

  - 在消费者启动类中添加注解开启Feign的功能，

    ```java
    @EnableFeignClients
    @SpringbootApplication
    public class OrderApplication {
      	public static void main(String[] args) {
          	SpringApplication.run(OrderApplication.class, args)
        }
    }
    ```

  - 编写Feign客户端,基于SpringMVC的注解来声明远程调用的信息

    ```java
    @FeignClient("userservice")
    public interface UserClient {
        @RequestMapping(method = RequestMethod.GET, value = "/users")
        List<User> getUsers();
    
        @RequestMapping(method = RequestMethod.GET, value = "/users/{userId}")
        User findById(@PathVariable("userId") Long userId);
    }
    // 服务名称:userservice；请求方式:GET；请求路径:/user/{id}；请求参数:Long id；返回值类型:User
    ```

  - Orderservice远程调用userservice

    ```java
    @Autowired
    private  Userclient userClient;
    
    public Order queryOrder(Long orderID) {
      	Order order = orderMapper.findByID(orderID);
      	// 远程调用userservice
      	User user =  userClient.findById(order.getUserID);
      	// 将user的内容封装到order中去
      	order.setUser(user);
      	return order;
    }
    ```

- **自定义Feign日志**：**feign.Logger.Level**包含四种层级NONE、BASIC、HEADERS、FULL

  - 方式一：配置文件配置

    ```yaml
    feign: 
    	client:
    		config:
    			default: # 用default就是全局配置，
    				loggerLevel: FULL # 日志级别
    				
    feign: 
    	client:
    		config:
    			userservice: # 写服务名称，则是针对某个微服务的配置
    				loggerLevel: FULL # 日志级别
    ```

  - 方式二：java代码配置

    ```java
    public class FeignClientConfiguration { 
      	@Bean
    		public Logger.Level feignLogLevel(){
    				return Logger.Level.BASIC; 
        }
    }
    // 然后在启动类中的@EnableFeignClients添加参数：全局配置、局部配置分别如下
    // @EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class) 
    // @EnableFeignClient(value = "userservice", configuration = FeignClientConfiguration.class) 
    ```








### 微服务保护--Sentinel

- **What?** 随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 是阿里巴巴开源的面向分布式服务架构的流量控制组件，主要以流量为切入点，从流量控制、熔断降级等多个维度来帮助您保障微服务的稳定性。

- **雪崩问题**：微服务调用链路中的某个服务故障，引起整个链路中的所有微服务都不可用，这就是雪崩。在微服务架构中，一个业务往往需要调用多个微服务去完成，如果突然该服务链路上的某个服务因为故障或者高流量并发阻塞不可用，整个链路阻塞，甚至需要调用该链路服务的其他链路也会阻塞，导致系统整体不可用。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-sentinel-snow.png" style="zoom:33%;" />

  - 解决雪崩问题的方法：
    - **超时处理**: 设定超时时间，请求超过一定时间没有响应就返回错误信息，不会无休止等待
    - **线程隔离**：限定每个业务能使用的线程数，避免耗尽整个容器的资源
    - **熔断降级**: 由断路器统计业务执行的异常比例，如果超出阈值则会熔断该业务，拦截访问该业务的一切请求。
    - **流量控制**: 限制业务访问的QPS，避免服务因流量的突增而故障。

- Sentinel分为两个部分：

  - **核心库**（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。引入依赖即可

  - **控制台**（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

  - 安装Sentinel控制台：

    ```shell
    # 1-官方下载jar包：https://github.com/alibaba/Sentinel/releases
    
    # 2-直接运行jar包
    java -Dserver.port=8080 -Dsentinel.dashboard.auth.username=sentinel -Dsentinel.dashboard.auth.password=123456 -jar sentinel-dashboard.jar
    # 参数：控制台接口、控制台用户名和密码
    ```

- 微服务整合Sentinel：

  - 在某个服务中引入sentinel依赖

    ```xml
    <dependency>
    	<groupId>com.alibaba.cloud</groupId> 
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    ```

  - 配置控制台地址:

    ```yaml
    spring: 
    	cloud:
    		sentinel: 
    			transport:
    				port: 8719
    				dashboard: localhost:8080
    ```

- 核心概念：

  - **簇点链路**：就是项目内的调用链路，链路中被监控的每个接口就是一个资源。默认情况下sentinel会监控SpringMVC 的每一个端点(Endpoint)，即Controller中的映射方法。如果要标记其它方法，需要利用@SentinelResource注解；同时需要添加如下配置

    ```yaml
    spring: 
    	cloud:
    		sentinel:
    			web-context-unify: false 
    ```

  - **QPS**：Queries  Per Second 每秒查询率

- **Sentinel限流规则**:

  - **What？**通过限制访问该服务资源的流量来保护该资源的状态

  - **三种流控模式**：

    - **直接**：统计当前资源的请求，触发阈值时对当前资源直接限流，也是默认的模式

    - **关联**：统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流

      - 需要满足两个条件：两个有竞争关系的资源；一个优先级较高，一个优先级较低
      - 比如用户支付时需要修改订单状态，同时用户要查询订单。查询和修改操作会争抢数据库锁，产生竞争。业务需求是优先支付和更新订单的业务，因此当修改订单业务触发阈值时，需要对查询订单业务限流。

    - **链路**：统计从指定链路访问到本资源的请求，触发阈值时，对指定链路限流。是对请求来源的限流

      - 比如有查询订单和创建订单业务，两者都需要查询商品。针对从查询订单进入到查询商品的请求统 计，并设置限流。

        <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-sentinel-query.png" style="zoom:50%;" />

  - **流控效果**：

    - 快速失败: 达到阈值后，新的请求会被立即拒绝并抛出FlowException异常。是默认的处理方式。
    - warm up: 预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一个较小值逐渐增加到最大阈值。
      - 是应对服务冷启动的一种方案。请求阈值初始值是 threshold / coldFactor，持续指定时长后， 逐渐提高到threshold值。而coldFactor的默认值是3。
    - 排队等待: 当请求超过QPS阈值时，快速失败和warm up 会拒绝新的请求并抛出异常。而排队等待则是让所有请求进入一个队列中 ，然后按照阈值允许的时间间隔依次执行。后来的请求必须等待前面执行完成，如果请求预期的等待时间超出最大时 长，则会被拒绝。

  - **热点参数限流**：之前的限流是统计访问某个资源的所有请求，判断是否超过QPS阈值。而热点参数限流是分别统计参数值相同的请求， 判断是否超过QPS阈值。

    - 对某个这个资源的索引参数做统计，每一统计窗口时长相同参数值的请求数不能超过 threshold

      <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-sentinel-hot.png" style="zoom:50%;" />

- **Feign整合Sentinel**: 微服务调用都是通过Feign来实现的，必须整合Feign和Sentinel。

  - 开启Feign的Sentinel功能

    ```yaml
    feign: 
    	sentinel:
    		enabled: true # 开启Feign的Sentinel功能
    ```

  - 给FeignClient编写调用失败后的降级逻辑：定义类，实现FallbackFactory接口

    ```java
    // 指定要返回的远程调用接口
    public class UserClientFallbackFactory implements FallbackFactory<UserClient> { 		
      	@Override
    		public UserClient create(Throwable throwable) {
    			 // 创建远程调用接口UserClient的实现类，实现其中的方法，编写失败降级的处理逻辑
          return new UserClient() { 
            	@Override
    					 public User findById(Long id) { // 记录异常信息
    									log.error("查询用户失败", throwable); 
                 		 // 根据业务需求返回默认的数据，这里是空用户
    									return new User(); }
          }; 
        }
    }
    ```

  - 将UserClientFallbackFactory注册为一个Bean并在UserClient接口中使用UserClientFallbackFactory

    ```java
    @Bean
    public UserClientFallbackFactory userClientFallback(){ 
      	return new UserClientFallbackFactory();
    }
    
    @FeignClient(value = "userservice", fallbackFactory = UserClientFallbackFactory.class) 
    public interface UserClient {
    		@GetMapping("/user/{id}")
    		User findById(@PathVariable("id") Long id); 
    }
    ```

- **Sentinel线程隔离**

  - **What**：通过限制自身的线程数量保护该资源的状态
  - 实现方式：修改QPS为线程数

- **Sentinel熔断降级**

  - **What？**由断路器统计服务调用的异常比例、慢请求比例，如果超出阈值则会熔断该服务。即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-sentinel-circuit.png" style="zoom:50%;" />

  - 断路器熔断三种策略策略：

    - 慢调用比例：业务的响应时长大于指定时长（最大RT）的请求认定为慢调用请求。在指定时间（统计时长）内，如果请求数量超过设定的最小数量（最小请求数），且慢调用比例大于设定的*比例阈值*，则触发熔断，熔断设置的时长。

    - 异常比例和异常数：在指定时间（统计时长）内，如果请求数量超过设定的最小数量（最小请求数），且出现异常比例大于设定的*比例阈值*（或者出现异常的请求数超过设定的数值），则触发熔断，熔断设置的时长。

      ![](https://github.com/Xiongkai-Wang/photos/blob/main/springCloud-sentinel-circuit-breaker.png)

      

- **Sentinel授权规则**：

  - **What?** 授权规则可以对调用方的来源做控制，有白名单和黑名单两种方式。

    - 白名单:来源(origin)在白名单内的调用者允许访问
    - 黑名单:来源(origin)在黑名单内的调用者不允许访问

  - 实现方式：

    - Sentinel通过RequestOriginParser接口的parseOrigin方法来获取请求的来源

      ```java
      // 例如从request中获取一个名为origin的请求头，作为origin的值
      @Component
      public class HeaderOriginParser implements RequestOriginParser { 
        	@Override
      		public String parseOrigin(HttpServletRequest request) { 
            	String origin = request.getHeader("origin"); 	
            	if(StringUtils.isEmpty(origin)){
                return "blank"; 
              }
            return origin; 
          }
      }
      ```

    - 在gateway服务中，利用网关的过滤器添加名为gateway的origin头

      ```yaml
      spring: 
      	cloud:
      		gateway: 
      			default-filters:
      				- AddRequestHeader=origin,gateway # 添加名为origin的请求头，值为gateway
      ```

      - 这样从gateway发出的请求都会有origin的值为gateway的头信息

    - 给资源配置授权规则: 请求的来源指定为gateway。这样就只允许从网关来的请求访问该资源

      <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springCloud-sentinel-auth.png" style="zoom:50%;" />

