## ElasticSearch



### Elasticsearch简介

- **Elastic Stack：**也称为 ELK Stack（代指 Elasticsearch、Logstash 和 Kibana），能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。目前 Elastic Stack 包括一系列丰富的轻量型数据采集代理，这些代理统称为 Beats，可用来向 Elasticsearch 发送数据。其中:
  - Elasticsearch是一个分布式的免费开源搜索和分析引擎。
  - Logstash 可用来对数据进行聚合和处理，并将数据发送到 Elasticsearch。Logstash 是一个开源的服务器端数据处理管道，允许您在将数据索引到 Elasticsearch 之前同时从多个来源采集数据，并对数据进行充实和转换。
  - Kibana 是一款适用于 Elasticsearch 的数据可视化和管理工具，可以提供实时的直方图、线形图、饼状图和地图。Kibana 同时还包括诸如 Canvas 和 Elastic Maps 等高级应用程序；Canvas 允许用户基于自身数据创建定制的动态信息图表，而 Elastic Maps 则可用来对地理空间数据进行可视化。
- **Elasticsearch**: 是一个**分布式**的存储、**搜索**和**分析**引擎。适用于包括文本、数字、地理空间、结构化和非结构化数据等在内的所有类型的数据。
  - **为什么**需要Elasticsearch？
    - Elasticsearch对模糊搜索非常擅长（搜索速度很快）。数据库也可以用like之类的关键字实现模糊查询，但是需要知道的是：select * from user where name like '%stu%'这类的查询是不走索引的（在不建立索引优化的情况下），意味着速度不快
    - Elasticsearch原生就支持排序，搜索到的数据可以根据评分过滤掉大部分的，只要返回评分高的给用户。即便给你从数据库根据模糊匹配查出相应的记录了，那往往会返回大量的数据给你，往往你需要的数据量并没有这么多。
    - Elasticsearch能匹配有相关性的记录，没有那么准确的关键字也能搜出相关的结果，用户输入的内容往往并没有这么的精确。这是数据库所没有的功能。
  - **如何工作**：
    - 1-**数据采集**： Elasticsearch 进行索引之前去解析、标准化并充实这些原始数据的过程。（原始数据会从多个来源（包括日志、系统指标和网络应用程序）输入到 Elasticsearch 中）
    - 2- Elasticsearch对数据进行**索引**；数据在索引完成之后，用户便可针对他们的数据运行复杂的查询，并使用聚合来检索自身数据的复杂汇总
    - 3-之后可以利用 Kibana 基于自己的数据创建强大的可视化，分享仪表板，对 Elastic Stack 进行管理
  - **应用场景**：Elasticsearch 在速度和可扩展性方面都表现出色，而且还能够索引多种类型的内容，用途广泛
    - 应用程序搜索、网站搜索、企业搜索
    - 日志处理和分析、基础设施指标和容器监测、应用程序性能监测、安全分析
    - 地理空间数据分析和可视化
    - 业务分析
- **Elasticsearch的特点**：
  - **速度快。**由于 Elasticsearch 是在 Lucene 基础上构建而成的，所以在全文本搜索方面表现十分出色。Elasticsearch 同时还是一个近实时的搜索平台，这意味着从文档索引操作到文档变为可搜索状态之间的延时很短，一般只有一秒。
  - **分布式，高可扩展性**。Elasticsearch 中存储的文档分布在不同的容器中，这些容器称为***分片***，还可以对分片进行复制以提供数据冗余副本，以防发生硬件故障。Elasticsearch 的分布式特性使得它可以扩展至数百台（甚至数千台）服务器，并处理 PB 量级的数据。
  - **功能广泛。**Elasticsearch 有大量强大的内置功能（例如数据汇总和索引生命周期管理），可以方便用户更加高效地存储和搜索数据。
  - **简化了数据采集、可视化和报告过程。**通过与 Beats 和 Logstash 进行集成，用户能够在向 Elasticsearch 中索引数据之前轻松地处理数据。同时，Kibana 不仅可针对 Elasticsearch 数据提供实时可视化，同时还提供 UI 以便用户快速访问应用程序性能监测 (APM)、日志和基础设施指标等数据。



### 重要概念

- **索引**(Index):  一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识(必须全部是小写字母)，并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。
- **类型**(Type): 在一个索引中，你可以定义一种或多种类型。但是在6.0版本以后，ES只能支持一种 type，而在7.0版本以后默认不再支持自定义索引类型，只有默认的类型_doc。
- **文档**(Document): 一个文档是一个可被索引的基础信息单元，也就是一条数据。文档以 JSON(Javascript Object Notation)格式来表示。在一个 index/type 里面，你可以存储任意多的文档。
- **字段**(Field): 相当于是数据表的列，对文档数据根据不同属性进行的分类标识。
- Elasticsearch索引：Elasticsearch 使用一种称为**倒排索引**的结构，它适用于**快速的全文搜索**
  - **正向索引**(forward index): 搜索引擎会将待搜索的文件都对应一个文件 ID，搜索时将这个ID和搜索关键字进行对应，形成K-V对，然后对关键字进行统计计数。（作为搜索引擎，文档数量会巨大，正向索引无法满足实时返回结果）
  
    ![](https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/elasticsearch-index.png)
  
  - **反向索引/倒排索引**(inverted index): 把文件 ID 对应到关键词的映射转换为关键词到文件 ID 的映射，每个关键词都对应着一系列的文件。



### 下载与安装

- 官网下载地址：https://www.elastic.co/cn/downloads/elasticsearch

- ES支持Linux、MacOS和Windows。

  <img src="https://raw.githubusercontent.com/Xiongkai-Wang/photos/main/elasticsearch-dir.png" style="zoom:50%;" />

- 快速开始：

  ```shell
  # 直接解压下载好的压缩包, jdk.app为内置的java环境
  # 切换到elasticsearch目录路径
  cd ./elasticsearch-7.16.3
  # 直接运行 
  ./bin/elasticsearch # （windows系统为./bin\elasticsearch.bat）
  
  # 用浏览器打开 http://localhost:9200/ 成功
  # 9200 端口为浏览器访问的 http 协议 RESTful 端口。
  ```



### Elasticsearch访问

- Elasticsearch **使用的是标准的 RESTful 风格的 API** 。实际开发中，主要有三种方式可以作为es服务的客户端

  - es提供的**RESTful接口**直接访问
  - elasticsearch-**head插件**访问
  - es提供的**API**进行访问，支持Java、Python、Php、Go等多种语言

- **使用RESTful接口直接访问**

  - 如果直接通过浏览器向 Elasticsearch 服务器发请求，那么需要在发送的请求中包含HTTP标准的方法，而 HTTP 的大部分特性仅支持 GET 和 POST 方法。所以为了能方便地进行客户端的访问，可以使用 Postman 软件
  - Postman是一款强大的网页调试工具，提供功能强大的 Web API 和 HTTP 请求调试。
  - 下载安装地址：https://www.getpostman.com/apps。支持Linux、MacOS和Windows

- **什么是RESTful风格？**

  - REST，即Representational State Transfer的缩写，翻译为表现层状态转化

    - **省略的主语-资源（Resources）** ：它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。
    - **表现层（Representation）**；"资源"是一种信息实体，它可以有多种外在表现形式。"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。URI只代表资源的实体，不代表它的形式。
      - 有些网址最后的".html"后缀名是不必要的，因为这个后缀名表示格式，属于"表现层"范畴，而URI应该只代表"资源"的位置。在RESTful中，它的具体表现形式，应该在HTTP请求的头信息中用Accept和Content-Type字段指定。
    - **状态转化（State Transfer）**：访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。在RESTful中，利用HTTP协议里面表示操作方式的动词来进行基本操作：**GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。**对资源的操作与资源本身无关，操作是通过 HTTP动词来体现，所以REST ful通过URI暴露资源时，会强调不要在 URI 中出现动词。

  - **RESTful的实现
    **：REST 风格提倡 URI地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

  - 举例：

    ```shell
    GET /rest/api/getCats --> GET /rest/api/cats  # 获取所有小猫 
    GET /rest/api/addCats --> POST /rest/api/cats # 添加一个小猫
    GET /rest/api/updateCats?cat_id=1 --> PUT /rest/api/cats/1 # 修改一个小猫
    GET /rest/api/deleteCats?cat_id=1 --> DELETE /rest/api/cats/1 # 删除一个小猫
    ```



### HTTP操作-索引操作

- 创建索引

  ```shell
  PUT http://127.0.0.1:9200/shopping
  
  # 返回结果：
  {
    "acknowledged": true, # 【响应结果】true操作成功 
    "shards_acknowledged": true, # 【分片结果】 分片操作成功 
    "index": "shopping" # 【索引名称】
  }
  ```

- 查看索引

  ```shell
  # 查看所有索引
  GET http://127.0.0.1:9200/_cat/indices?v
  # _cat 表示查看的意思; indices 表示索引; ?v 表示显示详细信息
  
  # 查看单个索引
  GET http://127.0.0.1:9200/shopping
  # 查看索引的请求路径和创建索引是一致的。但是HTTP方法不一致。这里可以体现RESTful的意义，
  ```

- 删除索引

  ```shell
  DELETE http://127.0.0.1:9200/shopping
  ```



### HTTP操作-文档操作

- 创建文档

  ```shell
  POST http://127.0.0.1:9200/shopping/_doc # 会随机生成指定数据唯一性标识(ID)
  
  # 在request body中加入
  {
    "title":"mate40",
    "category":"phone", 
    "images":"https://consumer.huawei.com/cn/phones", 
    "price":4999.00
  }
  
  # 如果想要自定义唯一性标识，需要在创建时指定
  POST http://127.0.0.1:9200/shopping/_doc/1
  {
    "title":"iphone 13",
    "category":"phone", 
    "images":"https://www.apple.com.cn/iphone/", 
    "price":5999.00
  }
  
  # 返回的response body中
  "_version": 1, # 对数据的操作，都会更新版本
  "result": "created", #created表示创建成功
  ```

- 查看文档: 查看文档时，需要指明文档的唯一性标识

  ```shell
  GET http://127.0.0.1:9200/shopping/_doc/1
  ```

- 修改文档

  ```shell
  # 1-修改整个文档
  POST http://127.0.0.1:9200/shopping/_doc/1 # 会将原有的数据内容覆盖
  
  # 2-修改字段
  POST http://127.0.0.1:9200/shopping/_update/1
  	# 在request body中加入：
  {
    "doc": {
       "price":6299.00
      }
  }
  
  # 返回的response body中
  "result": "updated", # updated表示数据被更新
  ```

- 删除文档

  ```shell
  # 1--根据id删除
  DELETE http://127.0.0.1:9200/shopping/_doc/1
  
  # 返回的response body中
  "result": "deleted", # deleted表示数据被标记为删除
  
  # 2--根据条件删除
  POST http://127.0.0.1:9200/shopping/_delete_by_query
   # 在request body中加入：
  {
    "query":{
       "match":{
         "price":5000.00
    } }
  }
  ```

  

### HTTP操作-映射操作

- **What？**映射Mapping类似于数据库(database)中的表结构(schema)。创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)。

- 创建映射：

  ```shell
  PUT http://127.0.0.1:9200/student/_mapping
  
  # 在properties中添加映射规则
  {
    "properties": {
        "name":{
           "type": "text",
           "index": true
      		}, 
      	"sex":{
           "type": "text",
           "index": true
         	},
        "age":{
           "type": "long",
           "index": true
  				} 
  	}
  }
  
  # type:	String (包括text--可分词和keyword--不可分词)、Numerical（long、integer、short、byte、double、float、half_float、scaled_float）、Date、Array和Object
  # index:是否索引，默认为true，所有字段都会被索引。false:字段不会被索引，不能用来搜索
  
  ```

- 查看映射

  ```shell
  GET http://127.0.0.1:9200/student/_mapping
  ```

- 创建索引时关联映射

  ```shell
  PUT http://127.0.0.1:9200/student1
  
  # 添加映射内容
  {
    "settings": {},
    "mappings": {
        "properties": {
          "name":{
           "type": "text",
           "index": true
  				 }, 
  				 "sex":{
           "type": "text",
           "index": false
          },
          "age":{
           "type": "long",
  					"index": false
          }
         }
  		} 
  }
  ```

  

### HTTP操作-高级查询

- 查询所有文档

  ```shell
  GET http://127.0.0.1:9200/student/_search
  {
  	"query": {
    	"match_all": {}
   	 }
  }
  #  "query":这里的query代表一个查询对象，里面可以有不同的查询属性 
  #  "match_all":查询类型，例如:match_all(代表查询所有)， match，term ， range 等等 
  # 	{}:查询条件会根据类型的不同，写法也有差异
  
  # 结果在返回的hits字段中，total表示匹配总数
  ```

- 匹配查询

  ```shell
  # match会把查询条件进行分词，然后进行查询，多个词条之间是 or 的关系
  # 比如查询“北京奥运”，会分词为北京，奥运，北京奥运，只要搜索词为这三个之一，都可以将这篇文章搜索出来
  GET http://127.0.0.1:9200/student/_search 
  {
  	"query": {
        "match": {
          "name":"zhangsan"
        }
   	 }
  }
  ```

- 字段匹配查询

  ```shell
  # multi_match 与 match 类似，不同的是它是在多个字段中查询该值。
  GET http://127.0.0.1:9200/student/_search 
  {
    "query": {
       "multi_match": {
           "query": "zhangsan",
           "fields": ["name","nickname"]
    		} 
    }
  }
  ```

- 关键字精确查询

  ```shell
  GET http://127.0.0.1:9200/student/_search 
  # 1--单关键字：term查询，精确的关键词匹配查询，不对查询条件进行分词
  {
    "query": {
       "term": {
         "name": {
           "value": "zhangsan"
         }
    	} 
    }
  }
  
  # 2--多关键字：terms，允许你指定多值进行匹配。
  {
    "query": {
       "terms": {
         "name": ["zhangsan","lisi"]
    	} 
    }
  }
  ```

- 指定查询字段

  ```shell
  # 默认情况下，Elasticsearch在搜索的结果中，会把文档中所有字段都返回。 如果我们只想获取其中的部分字段，我们可以添加_source 的过滤
  GET http://127.0.0.1:9200/student/_search 
  {
  	"_source": ["name","nickname"],
    "query": {
       "terms": {
         "name": ["zhangsan","lisi"]
    	} 
    }
  }
  
  ## 用includes:来指定想要显示的字段
  ## 用excludes:来指定不想要显示的字段
  "_source": {
     "includes": ["name","nickname"]
    }
  ```

- 组合查询

  ```shell
  # `bool`把各种其它查询通过`must`(必须 )、`must_not`(必须不)、`should`(应该)的方式进行组合
  # must:如果有多个条件，这些条件都必须满足 and与;
  # should:如果有多个条件，满足一个或多个即可 or或
  # must_not:和must相反，必须都不满足条件才可以匹配到 ！非
  GET http://127.0.0.1:9200/student/_search 
  {
    "query": {
       "bool": {
         "must": [
              {
              "match": {
              	"name": "zhangsan"
              	}
           		} ],
         "must_not": [
             {
              "match": {
                "age": "40"
    						} 
    				} ],
         "should": [
             {
             "match": {
            		 "sex": "男" 
            		 }
             } ]
    		} 
    }
  }
  ```

- 范围查询

  ```shell
  # range 查询找出那些落在指定区间内的数字或者时间
  # 支持gt、gte、lt、lte四种
  GET http://127.0.0.1:9200/student/_search 
  {
    "query": {
         "range": {
           	"age": {
              "gte": 30,
              "lte": 35 
              }
    				} 
     }
  }
  ```

- 模糊查询

  ```shell
  # 返回包含与搜索字词相似的字词的文档。
  # 编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数: 更改字符(box → fox)，删除字符(black → lack)、插入字符(sic → sick)、转置两个相邻字符(act → cat)
  # 为了找到相似的术语，fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体 或扩展。然后查询返回每个扩展的完全匹配。
  GET http://127.0.0.1:9200/student/_search 
  {
      "query": {
         "fuzzy": {
           "name": {
             "value": "zhangsan",
             "fuzziness": 2
           	}
      		} 
      	}
  }
  ```

- 字段排序

  ```shell
  # sort 可以让我们按照不同的字段进行排序，并且通过 order 指定排序的方式。desc 降序，asc 升序。
  GET http://127.0.0.1:9200/student/_search 
  # 1--单字段排序
  {
    "query": {
       "match_all": {} 
     },
      "sort": [
      	{
           "age": {
               "order":"desc"
           }
    		}
    	] 
  }
  
  # 2--多字段排序
  {
    "query": {
       "match_all": {} 
     },
      "sort": [
      	{
         "age": {
             "order":"desc"
         	}
        }
        {
          "_score":{
                 "order": "desc"
              }
  				}
       ] 
  }
  ```

- 高亮查询

  ```shell
  # 在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮。
  GET http://127.0.0.1:9200/student/_search 
  {
     "query": {
         "match": {
           "name": "zhangsan"
      		} 
      },
    	"highlight": {
         "pre_tags": "<font color='red'>", # 前置标签
         "post_tags": "</font>", # 后置标签
         "fields": {
      				"name": {}  # 需要高亮的字段
      		}
  		} 
  }
  ```

- 分页查询

  ```shell
  # from:当前页的起始索引，默认从 0 开始。 from = (pageNum - 1) * size 
  # size:每页显示多少条
  GET http://127.0.0.1:9200/student/_search 
  {
    "query": {
       "match_all": {}
      },
    "sort": [ {
         "age": {
           "order": "desc"
    			} 
     } ],
     "from": 0,
     "size": 2
  }
  ```

- 聚合查询

  ```shell
  # 聚合允许使用者对 es 文档进行统计分析，例如取最大值、平均值等等。
  GET http://127.0.0.1:9200/student/_search 
  {
  	"aggs":{
       "max_age":{ # 对某个字段取最大值 max
         "max":{"field":"age"}
  		} },
  	"size":0 
  }
  
  # State 聚合对某个字段一次性返回 count，max，min，avg 和 sum 五个指标
  {
      "aggs":{
           "stats_age":{
             "stats":{"field":"age"}
      		} },
      "size":0 
  }
  ```

- 桶聚合查询

  ```shell
  # 桶聚和相当于 sql 中的 group by 语句
  {
    "aggs":{
         "age_groupby":{ 
             "terms":{"field":"age"}, # 先在terms分组
             "aggs": { # 在分组后再进行聚合
                "stats_age":{
                    "stats":{"field":"age"}
                  }
              }
    			} 
    },
    "size":0 
  }
  ```

  

### Java API

- 创建maven工程，引入依赖

  ```xml
  <dependency>
     <groupId>org.elasticsearch</groupId>
     <artifactId>elasticsearch</artifactId>
     <version>7.16.3</version>
  </dependency>
  <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.16.3</version>
  </dependency>
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
  </dependency>
  ```

- 建立client连接

  ```java
  // 1--Java Transport Client 
  TransportClient client = new PreBuiltTransportClient(Settings.EMPTY)
          .addTransportAddress(new TransportAddress(InetAddress.getByName("host1"), 9300))
          .addTransportAddress(new TransportAddress(InetAddress.getByName("host2"), 9300));
  
  // 2--Java REST Client
  RestHighLevelClient client = new RestHighLevelClient(
                  RestClient.builder(new HttpHost("localhost", 9200, "http"))
          );
  
  // 3--Java API 7.16.3
  RestClient restClient = RestClient.builder(
      new HttpHost("localhost", 9200)).build();
  ElasticsearchTransport transport = new RestClientTransport(
      restClient, new JacksonJsonpMapper());
  ElasticsearchClient client = new ElasticsearchClient(transport);
  ```

  

- 索引操作

  ```java
  // 1--创建索引
  CreateIndexRequest request1 = new CreateIndexRequest("user01");
  // 发送请求，获取响应
  CreateIndexResponse response1 = client.indices().create(request1, RequestOptions.DEFAULT);
  System.out.println("响应状态：" + response1.isAcknowledged());
  
  // 2--查询索引 
  GetIndexRequest request2 = new GetIndexRequest("user");
  GetIndexResponse response2 = client.indices().get(request2, RequestOptions.DEFAULT);
  System.out.println("aliases:"+response2.getAliases());
  System.out.println("settings:"+response2.getSettings());
  
  // 3--删除索引 - 请求对象
  DeleteIndexRequest request = new DeleteIndexRequest("user");
  AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
  System.out.println("操作结果 : " + response.isAcknowledged());
  ```

- 文档操作

  ```java
  // 1--新增文档 
  IndexRequest request = new IndexRequest();
  // 设置索引及唯一性标识
  request.index("user").id("1001");
  // 创建数据对象
  User user = new User();
  user.setName("zhangsan");
  user.setAge(30);
  user.setSex("male");
  ObjectMapper objectMapper = new ObjectMapper();
  String productJson = objectMapper.writeValueAsString(user);
  // 添加文档数据，数据格式为 JSON 格式
  request.source(productJson, XContentType.JSON);
  // 客户端发送请求，获取响应对象
  IndexResponse response = client.index(request, RequestOptions.DEFAULT);
  System.out.println("_index:" + response.getIndex());
  System.out.println("_id:" + response.getId());
  System.out.println("_result:" + response.getResult());
  
  // 2--修改文档 - 请求对象
  UpdateRequest request2 = new UpdateRequest();
  // 配置修改参数
  request2.index("user").id("1001");
  // 设置请求体，对数据进行修改
  request2.doc(XContentType.JSON, "sex", "女");
  UpdateResponse response2 = client.update(request2, RequestOptions.DEFAULT);
  System.out.println("_index:" + response2.getIndex());
  System.out.println("_id:" + response2.getId());
  System.out.println("_result:" + response2.getResult());
  System.out.println("--------------------");
  
  
  // 3--修改文档
  GetRequest request3 = new GetRequest().index("user").id("1001");
  GetResponse response3 = client.get(request3, RequestOptions.DEFAULT);
  System.out.println("_index:" + response3.getIndex());
  System.out.println("_type:" + response3.getType()); System.out.println("_id:" + response.getId());
  System.out.println("source:" + response3.getSourceAsString());
  System.out.println("--------------------");
  
  
  // 4--删除文档
  DeleteRequest request4 = new DeleteRequest().index("user").id("1"); 
  DeleteResponse response4 = client.delete(request4, RequestOptions.DEFAULT); 
  System.out.println(response.toString());
  System.out.println("--------------------");
  ```

  

- 查询操作

  ```java
  // 创建搜索请求对象
  SearchRequest request = new SearchRequest();
  request.indices("student");
  // 构建查询的请求体
  SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
  // 设置查询类型： 查询所有数据
  sourceBuilder.query(QueryBuilders.matchAllQuery());
  // sourceBuilder.query(QueryBuilders.termQuery("age", "30")); // term查询
  // sourceBuilder.sort("age", SortOrder.ASC); // 排序查询
  // RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age"); // 范围查询
  // rangeQuery.gte("30");
  // rangeQuery.lte("40");
  // sourceBuilder.query(QueryBuilders.fuzzyQuery("name","zhangsan").fuzziness(Fuzziness.ONE)); // 模糊查询
  // sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age")); // 聚合查询
  request.source(sourceBuilder);
  SearchResponse response = client.search(request, RequestOptions.DEFAULT);
  System.out.println("took:" + response.getTook());
  
  SearchHits hits = response.getHits();
  System.out.println("total:" + hits.getTotalHits());
  for (SearchHit hit : hits) {
    //输出每条查询的结果信息
    System.out.println(hit.getSourceAsString());
  }
  ```

  
