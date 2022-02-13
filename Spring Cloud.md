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



### 注册中心：Eureka

- **What?** Eureka是注册中心

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/springcloud-eureka.png" style="zoom:50%;" />

- 工作原理

  - 服务提供者启动时向eureka注册自己的信息，eureka保存提供者信息，消费者根据服务名称向eureka拉取提供者信息
  - 服务消费者需要调用服务时，利用负载均衡算法，从服务列表中挑选一个
  - 服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态，eureka会更新记录服务列表信息，心跳不正常会被剔除

- 搭建**EurekaServer**：

  - 创建项目，引入``spring-cloud-starter-netflix-eureka-server`的依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    ```

  - 在启动类中添加``@EnableEurekaServer`注解

  - 在application.yml配置文件中，添加下面的配置

    ```yaml
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

  - 引入``spring-cloud-starter-netflix-eureka-server`的依赖

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```

  - 在application.yml配置文件中，添加下面的配置

    ```yaml
    spring: 
    		application:
    				name: providerServiceName # or comsumerServiceName
    eureka:
    		client: 
    				service-url:
    						defaultZone: http://127.0.0.1:10086/eureka/
    ```

  