



## ES

它可以用于许多目的，但它的一个优势是索引半结构化数据流，例如日志或解码的网络数据包

### 命令示例

在Kibana的console上执行

```shell
GET /
GET /_cat/indices # 查看有哪些索引

# 创建一个indice
POST twitter/_doc/1
{
  "user":"GB",
  "uid":1,
  "city":"Beijing",
  "province":"Beijing",
  "country":"China"
}

# 查询记录
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


```





## Kibana

建议和Elasticsearch安装在一起，这样kibana启动时就会自动连接ES

