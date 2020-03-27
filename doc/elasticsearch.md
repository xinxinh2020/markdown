



## ES

它可以用于许多目的，但它的一个优势是索引半结构化数据流，例如日志或解码的网络数据包

### 命令示例

在Kibana的console上执行

```shell
GET /
GET /_cat/indices # 查看有哪些索引

# 创建一个indice(自动推断索引结构各个字段的类型)
POST twitter/_doc/1
{
  "user":"GB",
  "uid":1,
  "city":"Beijing",
  "province":"Beijing",
  "country":"China"
}

# 显示定义索引结构
PUT twitter/_mapping

# 查询某个索引所有记录
GET twitter/_search

# 指定id查询记录
GET twitter/_doc/1

# 更新记录
PUT twitter/_doc/1
{
  "user":"GB",
  "uid":1,
  "city":"Beijing",
  "province":"Beijing",
  "country":"China",
  "location":{
    "lat":"29.8",
    "lon":"111.3"
  }
}

# 删除索引
DELETE twitter

# 查看索引结构
GET twitter/_mapping # type:"text" 表示该字段可以用于全文索引
										 # keyword 表示可以用于聚合查询
										 # geo_point 表示地理信息
										 

```





## Kibana

建议和Elasticsearch安装在一起，这样kibana启动时就会自动连接ES

