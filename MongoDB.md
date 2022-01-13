## MongoDB



### NoSQL与SQL

- **What？**NoSQL (“Non  SQL” or “Not only SQL”) 数据库是在2000年代后期开发的，其重点是可伸缩性、快速查询、允许频繁更改应用程序，并使开发人员的编程更简单。

  - 使用SQL(结构化查询语言)访问的**关系数据库**开发于20世纪70年代，其重点是减少数据复制，因为存储比开发人员的时间成本高得多。SQL数据库往往具有严格、复杂的表格模式，通常需要昂贵的垂直扩展。

- **NoSQL 与 SQL 的比较**

  - SQL易于理解、易于维护（社区成熟，服务稳定，数据稳定，高一致性 ，读写实时）、使用灵活（SQL 语言功能丰富，可以在表间进行 JOIN）；
    - 但是高并发读写性能较弱 （为了保证高一致性加锁造成读写性能的牺牲）、表结构 (Schema) 改动成本高（修改表结构时会造成部分数据库服务不可用）、水平扩展较难 （需要解决跨服务器 JOIN，分布式事务等问题）
  - NoSQL读写性能高、水平扩展简单、存储格式多样、低廉的成本（开源）
    - 一致性支持较弱：多数 NoSQL 数据库倾向于牺牲一致性来增强可用性、不支持Join、大多数不支持事务（MangoDB可以）

- **NoSQL数据库的分类：**

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/MongoDB-nosql.png" style="zoom:33%;" />

  - 键值存储：memcached、Redis
  - 面向文档的数据库：MongoDB、CouchDB
  - 面向列的数据库：Cassandra、HBase 
  - 图数据库：Neo4j、Titan、HyperGraphDB

- **NoSQL 与 SQL如何选择？**

  - **是否需要ACID**：关系型数据库支持 ACID (Atomicity, Consistency, Isolation, Duration) 即原子性，一致性，隔离性和持续性。 相对而言，NoSQL 采用更宽松的模型 BASE (Basically Available, Soft state, Eventual Consistency) 即基本可用，软状态和最终一致性。**对于需要保证 ACID 的应用，我们可以优先考虑 SQL。反之则可以优先考虑 NoSQL**
  - **结构化数据or非结构化数据**：有时候我们的应用包含着非常结构化的数据，可以简单地放入一个二维表中；有时候我们的应用需要存储一个键值对，一个图 ，一篇文章。前者我们可以采用 SQL 或者 NoSQL，而后者我们优先考虑 NoSQL 并且选择合适类型的 NoSQL 数据库存储数据。
  - **扩展性**：**NoSQL** 数据库的**横向扩展性 (Horizontal Sacaling)** 一般来说要**好于关系型数据库**。而关系型数据库，传统上它的主要扩展方式是**纵向扩展性 (Veritical Scaling)** 即采用更大更强的机器。



### MangoDB简介

- **What？**MongoDB是一个开源、高性能、无模式的文档型数据库，当初的设计就是用于简化开发和方便扩展，是NoSQL数据库产品中的一种。它支持的数据结构非常松散，是一种类似于JSON的格式叫BSON，所以它既可以存储比较复杂的数据类型，又相当的灵活。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/MongoDB-%20structure.png" style="zoom:50%;" />

- **数据结构**：

  - BSON(Binary Serialized Document Format)是一种类json的一种二进制形式的存储格式，简称Binary JSON。BSON和JSON一样，支持内嵌的文档对象和数组对象。

  - MongoDB的最小存储单位就是文档(document)对象。

  - BSON支持比JSON更丰富的数据类型，如Date和BinData类型。在MongoDB中 所有的字段名称都是字符串。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/MongoDB-dataTypes.png" style="zoom:50%;" />

- **MongoDB的主要特点**：

  - 高性能、高可用性、高扩展性
  - 丰富的查询支持（MongoDB支持丰富的查询语言，支持数据聚合、文本搜索和地理空间查询等。）
  - 动态模式：文档模型支持可变的数据模式，不要求每个文档都具有完全相同的结构，例如在同一个文档中支持同一个字段拥有不同的数据类型，对很多异构数据场景支持非常好。

  

### MongoDB安装与启动

- MongoDB支持Linux、MacOS、Windows系统：https://docs.mongoing.com/install-mongodb/install-mongodb-community-edition

- 下面以MacOS为例安装

  ```shell
  # 进入你想下载MongoDB的目录，如/usr/local
  cd /usr/local 
  # 下载指定版本的压缩包
  sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.9.tgz
  # 解压
  sudo tar -zxvf mongodb-osx-ssl-x86_64-4.0.9.tgz
  # 重命名为 mongodb 目录
  sudo mv mongodb-osx-x86_64-4.0.9/ mongodb
  
  # 可以给MongoDB配置环境
  export PATH=/usr/local/mongodb/bin:$PATH
  #新建两个目录存放 MongoDB的数据和日志，并确保当前用户对以上两个目录有读写的权限
  sudo mkdir -p /usr/local/var/mongodb
  sudo mkdir -p /usr/local/var/log/mongodb
  sudo chown xiongkai /usr/local/var/mongodb
  sudo chown xiongkai /usr/local/var/log/mongodb
  
  # 以后台的方式启动MongoDB
  mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork
  # --dbpath 设置数据存放目录 --logpath 设置日志存放目录 --fork 在后台运行
  
  # 打开MongoDB shell
  cd /usr/local/mongodb/bin 
  ./mongo
  ```



### 常用命令

- 创建数据库

  ```shell
  show dbs # 查看所有数据库
  use runoob # 创建数据库
  db # 查看当前使用的数据库
  ```

- 创建集合

  ```shell
  db.createCollection(name, options)
  # capped: 如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。
  # size:为固定集合指定一个最大值，单位为Byte。
  # max: 指定固定集合中包含文档的最大数量。
  db.createCollection("mycol", { capped : true, size : 6142800, max : 10000 } )
  ```

- 插入文档

  ```shell
  db.COLLECTION_NAME.insert(document) 
  db.col.insert({title: 'MongoDB', 
      description: 'MongoDB: Nosql Database',
      url: 'http://docs.mongoing.com',
      tags: ['mongodb', 'database', 'NoSQL'],
  })
  ```

  