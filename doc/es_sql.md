

[TOC]



## 元数据操作

```shell
GET / # 查看ES信息
GET /_cat/indices?v # 查看有哪些索引,后面的v表示可以看到indice上面显示的 
# 显式定义索引结构
PUT twitter/_mapping
{}

# 显式定义索引结构
PUT /shakespeare
{
  "mappings": {
    "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
    }
  }
}

# 删除索引
DELETE twitter

# 查看索引结构
GET twitter/_mapping # type:"text" 表示该字段可以用于全文索引
										 # keyword 表示可以用于聚合查询
										 # geo_point 表示地理信息
```



## 增删改

```shell
# 创建一个indice(自动推断索引结构各个字段的类型)，再执行一遍会把原来的indice给删掉
POST twitter/_doc/1
{
  "user":"GB",
  "uid":1,
  "city":"Beijing",
  "province":"Beijing",
  "country":"China"
}

# 往index中添加记录（如果index不存在会自动创建一个）
PUT /twitter/_doc/2
{
  "user":"xiaoming",
  "uid":2,
  "city":"Beijing",
  "province":"Beijing",
  "country":"China"
}

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



## Bulk批量操作

```shell
# 将accounts.json中的数据导入到es
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"


curl -O https://download.elastic.co/demos/kibana/gettingstarted/8.x/shakespeare.json
curl -O https://download.elastic.co/demos/kibana/gettingstarted/8.x/accounts.zip
curl -O https://download.elastic.co/demos/kibana/gettingstarted/8.x/logs.jsonl.gz

curl  -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl
```



## 查询操作

```shell
# 指定id查询记录(每条记录都有一个id)
# 记录的自定义字段都在_source里
GET twitter/_doc/1

# 查询某个索引所有记录
GET twitter/_search

# 查询bank中的所有记录，并按account_number排序，默认只显示前10条
GET /bank/_search?pretty
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}

# 查询bank中的所有记录，并按account_number排序，显示第十页
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}

# 模糊搜索address中包含mill和lane的document
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}

# 模糊搜索address中包含mill lane的document
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}

# 使用bool进行复合条件搜索
# mush:必须精确匹配  mush_not:必须不包含 should:应该包含（包含该条件的记录相关性更高）
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}

# 在结果中过滤掉某些条件
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}


```



## 聚合查询

```shell
# 按state聚合查询数量最多的10条记录
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}

# 嵌套聚合
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```



## Grok

```shell
# grok预定义匹配字段：https://www.jianshu.com/p/443f1ea7b640

# 查询名字为test-log的pipeline信息
GET _ingest/pipeline/test-log

# 添加一个pipeline
PUT _ingest/pipeline/demo
{
  "description" : "demo",
  "processors" : [ 
    {
      "grok": {
        "field": "msg",
        "patterns": ["接收访问请求：%{NOTSPACE:url_path}，标识：%{NOTSPACE:uuid}，请求方法：%{NOTSPACE:method}，处理函数：%{NOTSPACE:func}，来源IP：%{NOTSPACE:ip}，用户：%{NOTSPACE:user},请求数据：", 
        "响应访问请求：%{NOTSPACE:url_path}，标识：%{NOTSPACE:uuid}，请求方法：%{NOTSPACE:method}，处理函数：%{NOTSPACE:func}，来源IP：%{NOTSPACE:ip}，用户：%{NOTSPACE:user}，请求数据：%{GREEDYDATA:input}，请求结果：%{GREEDYDATA:output}，耗时：%{SECOND:duration}s，错误信息：\n%{MILTI_LINE_TEXT:err}",
        "%{MILTI_LINE_TEXT:msg}"  # 匹配不到上述规则的日志直接打印出来，不做格式化(默认匹配，如果不写这个规则该日志就无法在kibana上显示)
        ],
        "pattern_definitions" : {   # 自定义字段
          "MILTI_LINE_TEXT" : "(.*\n)*"
      	}
      }
    }
  ]
}
```



## 一些说明

### 请求结果

```shell
took:搜索花费了多少毫秒
time_out:搜索是否发生了超时
_shards:查询了多少分片，多少分片成功，多少失败，多少忽略了
max_score:结果中最相关的记录的分数
hits.total.value:匹配的记录总数
hits.hits.sort:该document所在的搜索位置
hits.hits._score:该document的相关性，当使用match_all的时候为null

# 聚合查询的结果
buckets.key:匹配聚合查询的term的值
buckets.doc_count:匹配的数量
```

