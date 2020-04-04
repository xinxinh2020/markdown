[TOC]

## Elasticsearch

它可以用于许多目的，但它的一个优势是索引半结构化数据流，例如日志或解码的网络数据包

ES中的数据以JSON格式保存

Index相当于数据库的表，document相当于记录，field相当于字段（键值对形式）

ES中一个index存放在多个物理分片（shard）中，每个物理分片会存放一部分document，不同的物理分片可能位于不同的节点上，即提供了容错，也提供了搜索效率。分片分为主分片和备份分片，主分片的数量在某个时间点是固定的，备份分片则是主分片的拷贝。主分片的大小和数量是需要 权衡的，分片数越多，维护的开销就越大，而分片的大小越大，在自动迁移时就需要花更多的时间。一般，分片的平均大小保持在几G和几十G之间。数量一般1G空间少于20G

ES会索引每个字段，并且每个字段都有一个优化的数据类型（text，numeric，geo）

ES会自动推断数据类型，但如果你对你的数据很了解的话，显式指定数据类型更好（mapping），另外，地理位置信息无法被自动推断出来

text类型用于全文搜索，keyword类型用于排序和聚合

ES支持结构化查询和全文搜索，以及两者的混合。结构化查询类似于SQL。全文搜索返回的记录是按相关度排序的。

聚合查询利用的是结构化数据，所以很快

CCR（Cross-Cluster Replication）可以自动将indices从主集群同步到另外的集群，这些集群可以在不同的数据中心，可以提高容错性，也可以提高就近访问的读请求

bulk可以执行批量操作，性能比一个一个执行效果好，最佳的数量为1000-5000条document以及5MB到15MB之间的负载



### Query DSL

Query DSL：Domain Specific Language，基于JSON来定义查询。

将查询DSL看作查询的AST(Abstract Syntax Tree，抽象语法树)，包含了两种类型的子句：

- Leaf query clauses(叶查询子句): 叶查询子句在特定字段中查找特定值，比如match，term或range查询。这些子句可以单独使用
- Compound query clauses(复合查询子句)：复合查询子句复合了其他叶查询子句或复合查询子句，以逻辑（bool,dis_max）的方式结合多个查询

#### Query Context

 Query Context用于回答“一个文档(document)和这个查询子句有多匹配？”，也就是，它除了判断一个文档是否匹配之外，还计算了它的匹配程度，保存在_score这个字段中

#### Filter context

在Filter context中，一个查询子句回答类似“这个文档匹配吗”的问题，回答为是或否。不计算匹配程度，Filter context一般用于过滤结构化数据。

总结：在应该影响匹配分数的情况下使用query context，其他情况下使用filter。

## Kibana

建议和Elasticsearch安装在一起，这样kibana启动时就会自动连接ES

Index Patterns通过通配符的方式匹配多个ES，*表示通配，逗号用于连接多个名字，-表示不包含

