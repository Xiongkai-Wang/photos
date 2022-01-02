## Elasticache



###  Elasticache Introduction

- Elasticache是AWS提供的分布式内存对象缓存系统。

  - 在内存中存储关键数据实现低延迟访问

  - 避免过多低效率的磁盘IO，通过缓存查询数据提高网络应用的性能

  - 提供了Memcached和Redis两种常用缓存

    

### Elasticache Security

- Amazon为Redis集群的不同部署形式提供了不同的安全解决方案。

  - Elaticache Access Pattern -- Same VPC, Same Region

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-01.png" style="zoom:33%;" />

    - 直接在Redis中Security Group设置规则来允许EC2的连接

  - Elaticache Access Pattern -- Different VPC, Same Region

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-02.png" style="zoom:40%;" />

    - 用VPC Peering实现两个不同VPC之间的连接

  - Elaticache Access Pattern -- Different VPC, Different Region

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-03.png" style="zoom:40%;" />

    - 创建Transit VPC,  应用和Redis都用Transit VPC实现连接

  - Elaticache Access Pattern -- Corporate Datacenter (在同一Datacenter的应用集群访问同一Redis集群)

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-04.png" style="zoom:40%;" />

    - 通过VPN或Direct Connect来连接VPC和Datacenter



### Elasticache Session Store

- session store：从用户登陆时开始一个会话，直到超过设定时间或者登出，这段时间的连接需要保存会话信息。session保存在服务器端。

  - 在分布式系统中，如果通过负载均衡前后请求达到的服务器不一样，则会话信息的保存出现问题。

  - session replication：复制会话策略，即一个用户访问了一次就把session复制到集群中所有的服务器。---效率低

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-non-sticky.png" style="zoom:50%;" />

  - sticky session：粘性会话策略是指一个用户访问了一次后，同一个session周期内，所有的请求都定向到这个服务器。---负载不平衡

- 使用Elasticache：将session保存在缓存中，服务器集群共享该缓存

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/cache-elasticacheStore.png" style="zoom:50%;" />