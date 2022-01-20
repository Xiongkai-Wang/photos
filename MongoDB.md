## MongoDB



### NoSQL与SQL

- **What？**NoSQL (“Non  SQL” or “Not only SQL”) 数据库是在2000年代后期开发的，其重点是可伸缩性、快速查询、允许频繁更改应用程序，并使开发人员的编程更简单。

  - 使用SQL(结构化查询语言)访问的**关系数据库**开发于20世纪70年代，其重点是**减少数据复制**，因为存储比开发人员的时间成本高得多。SQL数据库往往具有严格、复杂的表格模式，通常需要昂贵的垂直扩展。

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



### MongoDB简介

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

- 下面以MacOS为例安装,Linux类似

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



### 基本常用命令

- **数据库**操作

  - 数据库命名规范：
    - 不能是空字符串；
    - 不得含有``' '``(空格)、`.`、`$`、`/`、`\`和`\0` (空字符)；
    - 全小写；
    - 最多64字节
  - 默认自带数据库
    - admin: 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
    - local: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
    - config: 当MongoDB用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

  ```shell
  show dbs # (or databases) 查看所有数据库 
  use runoob # 选择和创建数据库
  db # 查看当前使用的数据库
  
  db.dropDatabase() # 删除数据库
  ```

- **集合**操作

  - 集合命名规范：
    - 不能是空字符串；
    - 不能含有`\0`(空字符)
    - 不能以``system.``开头，这是为系统集合保留的前缀
    - 用户创建的集合名字不能含有保留字符；除非你要访问这种系统创建的集合，否则千万不要在名字里出现`$`。

  ```shell
  show collections #  (or tables) 查看当前库中的集合
  db.createCollection(name, options)  # 创建集合
  	## capped: 如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。
  	## size:为固定集合指定一个最大值，单位为Byte。
  	## max: 指定固定集合中包含文档的最大数量。
  db.createCollection("mycol", { capped : true, size : 6142800, max : 10000 } )
  
  db.collectionName.drop() # 删除集合
  ```

- **文档**操作

  - 注意：
    - key的命名规范：不能含有`\0`(空字符)；``.``和`$`有特别的意义，只有在特定环境下才能使用。
    - 文档中的键/值对是有序的；MongoDB区分类型和大小写；MongoDB的文档不能有重复的键；以下划线`_`开头的键是保留的(不是严格要求的)。
    - mongo中的数字，默认情况下是double类型，如果要存整型，必须使用函数``NumberInt(num)``，否则取出来就有问题了；插入当前日期使用new Date()；插入的数据没有指定 _id ，会自动生成主键值
    - 批量插入时，如果某条数据插入失败，将会终止插入，但已经插入成功的数据不会回滚掉。/usr/local/mongodb/bin

  ```shell
  # 插入一条文档（插入文档时，如果该collection不存在时，会自动创建）
  db.collectionName.insert(<document>, { writeConcern: <document>, ordered: <boolean>}) 
  	## writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
  	## ordered：指定是否按顺序写入，默认true，按顺序写入。
  db.col.insert({title: 'MongoDB', 
      description: 'MongoDB: Nosql Database',
      url: 'http://docs.mongoing.com',
      tags: ['mongodb', 'database', 'NoSQL']})
  
  # 批量插入文档
  db.col.insertMany([
  {title: 'MongoDB', description: 'MongoDB: Nosql Database',url: 'http://docs.mongoing.com', tags: ['mongodb', 'database', 'NoSQL'], likenum:50},
  {title: 'Redis', description: 'Redis: Nosql Database',url: 'http://docs.redis.com', tags: ['redis', 'database', 'NoSQL'], likenum:150},
  {title: 'MySQL', description: 'MySQL: Relational Database',url: 'http://docs.mysql.com', tags: ['mysql', 'database', 'SQL'], likenum:300}
  ]);
  
  # 文档的基本查询
  db.collectionName.find(query, projection)
   ## query ：可选，使用查询操作符指定查询条件
   ## projection ：可选，投影操作符指定返回的键。
   
  db.col.find()  # 查询所有
  db.col.find({title:"Redis"}, {title:1, url:1}) # 查询title为redis的数据，返回title和url字段
  
  # 文档的更新
  db.collectionName.update(query, update, options)
   ## update: $set 和 $inc (对原有值增加) 如 {$inc:{likenum:NumberInt(1)}}
   ## upsert：可选，默认是false，如果不存在update的记录，是否插入新文档, true为插入
   ## multi: 可选，默认是false,只更新第一条记录，如果这个参数为true,按条件查出来多条记录全部更新。
  
  db.col.update({title:"Redis"}, {$set:{url:'http://org.redis.com'}}, {multi:true})
  
  # 删除文档 
  db.collectionName.remove(query, {justOne: <boolean>})
   ## justOne 可选,如果设为 true 或 1，则只删除一个文档，默认false，删除所有匹配条件的文档。
  db.col.remove({}) # 删除全部数据
  db.col.remove({title:"MySQL"}) # 删除title为MySQL的数据
  ```

  

### 文档的高级查询

- 条件操作符

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/MongoDB-query.png" style="zoom:50%;" />

- 文档的统计查询 `count()`

  ```shell
  db.collectionName.count(query)
   
  db.col.count() # 统计col集合的所有的记录数
  db.col.count({likenum:{$gt:100}}) # 统计为likenum大于100的记录条数
  ```

- 文档的分页列表查询 `limit()`  `skip()`

  ```shell
  db.collectionName.find().limit(num) # 返回指定条数的记录,默认值20
  db.collectionName.find().skip(num) # 接受一个数字参数作为跳过的记录条数(前N个不要),默认值是0
  
  db.col.find().skip(0).limit(2) # 第一页
  db.col.find().skip(2).limit(2) # 第二页
  db.col.find().skip(4).limit(2) # 第三页
  ```

- 文档的排序查询 `sort()`

  ```shell
  db.collectionName.find().sort({key:1}) # 1为升序 -1为降序
  
  db.col.find().sort({likenum:-1}) # 对likenum进行升序排列
  ```

- 正则的复杂条件查询 

  ```shell
  db.collectionName.find({key:/正则表达式/})
  
  db.col.find({description:/Nosql/})
  ```

- 包含查询 `$in`

  ```shell
  db.col.find({title:{$in:["MongoDB","Redis"]}})
  ```

- 条件连接查询 `$and` `$or`

  ```shell
  $and:[{query1},{query2},{query3}]
  $or:[{query1},{query2},{query3}]
  
  db.col.find({$or:[ {title:"MongoDB"} ,{likenum:{$gt:200} }]})
  ```

  

### 索引

- **What？**Index是特殊的数据结构（MongoDB索引使用B-树数据结构），它以易于遍历的形式存储集合数据集的一小部分。索引存储特定字段或一组字段的值，按字段值排序。索引项的排序支持有效的高效的查询操作。此外，MongoDB还可以使用索引中的排序返回排序结果。如果没有索引，MongoDB必须执行全集合扫描，即扫描集合中的每个文档，以选择与查询语句匹配的文档，查询效率非常低。

- **分类**

  - **单字段索引(Single Field Index)**：在文档的单个字段上创建用户定义的升序/降序索引，称为单字段索引
  - **复合索引(Compound Index)**：支持自定义的多个字段索引。复合索引中列出的字段顺序具有重要意义。例如，如果复合索引由{ userid: 1, score: -1 }组成，则索引首先按userid正序排序，然后 在每个userid的值内，再在按score倒序排序。
  - **地理空间索引(Geospatial Index)**：支持对地理空间坐标数据的有效查询。可以应用在O2O（OnlineToOffline）的场景，比如『查找附近的美食』、『查找某个区域内的车站』等。
  - **文本索引(Text Index)**：MongoDB提供了一种文本索引类型，支持在集合中搜索字符串内容。能解决快速文本查找的需求，比如有一个博客文章集合，需要根据博客的内容来快速查找。
  - **哈希索引(Hashed Index)**：是指按照某个字段的hash值来建立索引，目前主要用于MongoDB Sharded Cluster的Hash分片。

- **索引基本操作**

  ```shell
  # 查看索引
  db.collectionName.getIndexes()
  
  # 创建索引
  db.collectionName.createIndex(keys, options)
  	# keys：  {字段:1或-1}表示升序或者是降序
  	# name: 索引的名称。若未指定，连接索引的字段名和排序顺序生成一个索引名称。‘
  	# v：索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。
  	# expireAfterSeconds：过期时间
  db.col.createIndex({title:1,likenum:-1})
  
  # 移除索引
   db.collectionName.dropIndex(index) # 可以通过索引名称或索引规范文档指定索引。
   db.collection.dropIndexes() # 移除所有索引
  ```

- **覆盖索引查询**: 

  - 覆盖查询满足两个条件：所有的查询字段是索引的一部分，所有的查询返回字段是索引的一部分。由于所有出现在查询中的字段是索引的一部分， MongoDB 无需在整个数据文档中检索匹配查询条件和返回使用相同索引的查询结果。因为索引存在于RAM中，从索引中获取数据比通过扫描文档读取数据要快得多。

  ```shell
  db.col.createIndex({title:1}) # 先创建索引
  db.col.find({title:{$in:["MongoDB","Redis"]}}) # 覆盖索引查询查询
  
  # 使用explain查询分析函数查看索引是否起作用
  db.col.find({title:{$in:["MongoDB","Redis"]}}).explain()
  # "stage" : "COLLSCAN" 表示全集合扫描
  # "stage" : "IXSCAN" 说明是基于索引的扫描
  ```

  

### MongoDB Java-driver

- 环境配置：在maven工程中添加依赖

  ```xml
  <dependencies>
          <dependency>
              <groupId>org.mongodb</groupId>
              <artifactId>mongo-java-driver</artifactId>
              <version>3.9.1</version>
          </dependency>
  </dependencies>
  ```

- 具体操作：

  ```java
  public class MongoTest {
      public static void main(String[] args) {
          try {
              MongoClient mongoClient = new MongoClient("127.0.0.1", 27017);
              // 连接数据库，你需要指定数据库名称，如果指定的数据库不存在，mongo会自动创建数据库。
              MongoDatabase database = mongoClient.getDatabase("test02");
              System.out.println("Connect to database successfully");
  
              // 创建集合
              database.createCollection("test");
              System.out.println("create collection successfully");
              // 选择集合
              MongoCollection<Document> test = database.getCollection("test");
              System.out.println("select collection successfully");
  
              //插入文档
              /**
               * 1. 创建文档 org.bson.Document 参数为key-value的格式
               * 2. 创建文档集合List<Document>
               * 3. 将文档集合插入数据库集合中 mongoCollection.insertMany(List<Document>)
               *     插入单个文档可以用 mongoCollection.insertOne(Document)
               * */
              Document document01 = new Document("title", "MongoDB").
                      append("description", "nosql database").
                      append("likes", 50).
                      append("by", "Cindy");
              Document document02 = new Document("title", "MySQL").
                      append("description", "relational database").
                      append("likes", 150).
                      append("by", "Bob");
              List<Document> documents = new ArrayList<Document>();
              documents.add(document01);
              documents.add(document02);
              test.insertMany(documents);
              System.out.println("insert documents successfully");
  
              //检索所有文档
              /**
               * 1. 获取迭代器FindIterable<Document>
               * 2. 获取游标MongoCursor<Document>
               * 3. 通过游标遍历检索出的文档集合
               * */
              FindIterable<Document> iterable = test.find();
              MongoCursor<Document> cursor = iterable.iterator();
              while (cursor.hasNext()) {
                  System.out.println(cursor.next());
              }
  
              //更新文档   将文档中likes=150的文档修改为likes=200
              test.updateMany(Filters.eq("likes", 100), new Document("$set",new Document("likes",200)));
              //删除符合条件的文档
              test.deleteMany(Filters.eq("likes", 200));
              System.out.println("update and delete ");
  
              FindIterable<Document> iterable1 = test.find();
              MongoCursor<Document> cursor1 = iterable1.iterator();
              while (cursor1.hasNext()) {
                  System.out.println(cursor1.next());
              }
  
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```

  

### MongoDB复制

- **What？**MongoDB复制（副本集）是将数据同步在多个服务器的过程。复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。复制还允许您从硬件故障和服务中断中恢复数据。与主从模式类似。

- **复制原理**：

  - mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。
  - 主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

- 具体操作

  ```shell
  # 通过指定 --replSet 选项来启动mongoDB
  mongod --port "PORT" --dbpath "YOUR_DB_DATA_PATH" --replSet "REPLICA_SET_INSTANCE_NAME"
  
  # 打开mongo shell
  rs.initiate() # 使用命令启动一个新的副本集
  rs.conf() # 查看副本集的配置
  rs.status() # 查看副本集状态
  
  rs.add(HOST_NAME:PORT) # 添加副本集的成员
  # 注意：只能通过主节点将Mongo服务器添加到副本集中，判断当前运行的Mongo服务器是否为主节点可以使用命令db.isMaster() 
  ```

  

### MongoDB分片

- **What？**Mongodb存在的另一种集群用于横向扩展，分片（Sharding），可以满足MongoDB数据量大量增长的需求。当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

  - 纵向扩展：或者说垂直扩展，意味着增加单个服务器的容量，例如使用更强大的CPU，添加更多RAM或增加存储空间量。

  - 横向扩展：划分系统数据集并加载多个服务器，添加其他服务器以根据需要增加容量。

    <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/MongoDB-sharding.png" style="zoom:50%;" />

- **分片集群的组成**：

  - **Shard**： 用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障
  - **Config Server:** mongod实例，存储了整个 ClusterMetadata。
  - **Query Routers:** 前端路由，是客户端应用程序和分片集群之间的接口。

- 具体实现：

  ```shell
  # 1 启动多个Shard Server, 如s0 s1
  mkdir /usr/local/mongodb/shard/s0
  ./mongod --port 27020 --dbpath=/usr/local/mongodb/shard/s0 --logpath=/usr/local/mongodb/shard/log/s0.log --logappend --fork
  
  # 2 启动Config Server
  mkdir -p /usr/local/mongodb/shard/config
  ./mongod --port 27100 --dbpath=/usr/local/mongodb/shard/config --logpath=/usr/local/mongodb/shard/log/config.log --logappend --fork
  
  # 3 启动Router
  ./mongos --port 40000 --configdb localhost:27100 --fork --logpath=/usr/local/mongodb/shard/log/route.log --chunkSize 500
  # chunkSize这一项是用来指定chunk的大小的，单位是MB，默认大小为200MB.
  
  # 配置Sharding: 需要登陆到Router服务器的shell中去配置
  ./mongo admin --port 40000
  # 登陆到shell中后
  db.runCommand({ addshard:"localhost:27020" })
  db.runCommand({ enablesharding:"test" })  #设置分片存储的数据库
  db.runCommand({ shardcollection: "test.log", key: { id:1} }) # 分片的集合以及分片规则
  ```

  
