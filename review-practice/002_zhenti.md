# 1、集群部署篇
## 1.1 部署 3 节点的集群，需要同时满足以下要求
【铭毅天下 elastic.blog.csdn.net 答案】 
- 集群名为“geektime” 
- 将每个节点的名字设为和机器名一样，分别为 node1，node2，node3 
- node1 配置成dedicated master-eligable节点
- node2和node3配置成 ingest 和 data node
- 设置 jvm 为1g

```
1、elasticsearch.yml
cluster.name: geektime

node.name: node2
node.name: node3

2、node1
node.name: node1
node.master: true 
node.data: false 
node.ingest: false 
node.ml: false 

3、node2
node.name: node2
node.master: false 
node.data: false 
node.ingest: true 
node.ml: false

4、node3
node.name: node3
node.master: false 
node.data: true 
node.ingest: false 
node.ml: false

5、jvm.options
-Xms1g 
-Xmx1g

```

## 1.2 配置 3节点的集群，加上一个 Kibana 的实例，设定以下安全防护
- 为集群配置 basic authentication 
- 将 Kibana 连接到 Elasticsearch
- 创建一个名为 geektime 的用户
- 创建一个名为 orders 的索引
- geektime 用户只能读取和写入 oders 的索引，不能删除及修改 orders

【铭毅天下 elastic.blog.csdn.net 答案】

```
elasticsearch.yml配置：
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

这里有个问题?考试的时候，如下操作是允许kibana图形化吧？估计概率不大（已经向官方邮件求证，估计还得dsl实现）

```
PUT orders/_bulk
{"index":{"_id":1}}
{"name":"11111"}
{"index":{"_id":2}}
{"name":"22222"}

POST /_security/role/geektime_role
{
  "indices": [
    {
      "names": [ "orders" ],
      "privileges": ["read","write"]
    }
  ]
}

POST /_security/user/geektime
{
  "password" : "123456",
  "roles" : [ "geektime_role" ]
}

#会提示失败
DELETE orders

#以下，提示成功
GET orders/_search
PUT orders/_bulk
{"index":{"_id":3}}
{"name":"3333"}
{"index":{"_id":4}}
{"name":"4444"}
```

## 1.3 配置 3节点的集群，同时满足以下要求
- 确保索引 A 的分片全部落在在节点1
- 索引 B 分片全部落在 节点 2和3 
- 不允许删除数据的情况下，保证集群状态为 Green

【铭毅天下 elastic.blog.csdn.net 答案】
在elasticsearch.yml下添加如下配置（举例）：
```
node.attr.size: medium
node.attr.size: big
```

```
PUT a_index
{
  "settings": {
    "index.routing.allocation.include.size": "big"
  }
}

PUT b_index
{
  "settings": {
    "index.routing.allocation.include.size": "medium"
  }
}


POST a_index/_bulk
{"index":{"_id":1}}
{"name":"1111"}
{"index":{"_id":2}}
{"name":"2222"}

POST b_index/_bulk
{"index":{"_id":111}}
{"name":"33333"}
{"index":{"_id":2222}}
{"name":"4444"}

GET _cat/nodeattrs

GET _cat/shards?v
```

# 2、索引数据篇
## 2.1 为一个索引，按要求设置以下 dynamic Mapping
- 一切 text 类型的字段，类型全部映射成 keyword
- 一切以 int_开头命名的字段，类型都设置成 integer

【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT index001
{
  "mappings": {
    "dynamic_templates": [
      {
        "longs_as_strings": {
          "match_mapping_type": "string",
          "unmatch": "int_*",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "longs_as_strings": {
          "match":   "int_*",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ]
  }
}

POST index001/_bulk
{"index":{"_id":1}}
{"cont":"你好我好铭毅天下好", "int_value":35}
{"index":{"_id":2}}
{"cont":"铭毅天下", "int_value":35}
{"index":{"_id":3}}
{"cont":"铭毅好", "int_value":35}

GET index001/_mapping
```
## 2.2 设置一个Index Template，符合以下的要求
- 为 log 和log- 开头的索引。创建 3 个主分片，1 个副本分片
- 同时为索引创建一个相应的 alias
- 使用 bulk API，写入多条电影数据

【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT _template/template_001
{
  "index_patterns": ["log*", "log-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas":1
  },

"aliases" : {
        "alias_001" : {}}
}


PUT log-001

POST log-001/_bulk
{"index":{"_id":1}}
{"name":"movice_001"}
{"index":{"_id":2}}
{"name":"movice_002"}
{"index":{"_id":3}}
{"name":"movice_003"}


GET log-001
GET alias_001/_search
```
## 2.3 为 movies index 设定一个 Index Alias，默认查询只返回评分大于3的电影

【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT movies_index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "score": {
        "type": "float"
      }
    }
  }
}

PUT movies_index/_bulk
{"index":{"_id":1}}
{"name":"001","score":1}
{"index":{"_id":2}}
{"name":"001","score":4}
{"index":{"_id":3}}
{"name":"001","score":3}
{"index":{"_id":4}}
{"name":"001","score":3.8}

GET movies_index/_mapping
GET movies_index/_search

POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "movies_index",
        "alias": "movies_alias",
        "filter": {
          "range": {
              "score":{
                "gt":3
              }
          }
        }
      }
    }
  ]
}

GET movies_alias/_search
```

## 2.4 给一个索引 A，要求创建索引 B，通过 Reindex API，将索引 A 中的文档写入索引 B，同时满足以下要求
- 增加一个整形字段，将索引 A中的一个字段的字符串长度，计算后写入
- 将 A 文档中的字符串以“；”分隔后，写入索引B中的数组字段中

【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT aindex
{
  "mappings": {
    "properties": {
      "name":{"type":"text"},
      "horry":{"type":"keyword"}
    }
  }
}

GET aindex/_mapping

POST aindex/_bulk
{"index":{"_id":1}}
{"name":"xiaozhang","horry":"pingpang;basketball;football"}
{"index":{"_id":2}}
{"name":"mingyi","horry":"glof;basketball;football"}
{"index":{"_id":3}}
{"name":"mytx","horry":"glof;basketball;ticket"}


PUT _ingest/pipeline/my_pipeline_id
{
  "description": "describe pipeline",
  "processors": [
    {
      "split": {
        "field": "horry",
        "separator": ";"
      }
    },
    {
      "script": {
        "source": """
            ctx._length = ctx.name.length();
"""
      }
    }
  ]
}

PUT bindex
GET bindex/_mapping

POST _reindex
{
  "source": {
    "index": "aindex"
  },
  "dest": {
    "index": "bindex",
    "pipeline": "my_pipeline_id"
  }
}

GET bindex/_search
```

## 2.5 定义一个 Pipeline，并且将 eathquakes 索引的文档进行更新
- pipeline的 ID 为 eathquakes_pipeline
- 将 magnitude_type 的字段值改为大写
- 如果文档不包含 “batch_number”, 增加这个字段，将数值设置为 1
- 如果已经包含 batch_number, 字段值➕1 

【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT earthquakes
{
  "mappings": {
    "properties": {
      "name":{"type":"text"},
      "level":{"type":"integer"},
      "magnitude_type":{"type":"keyword"},
      "batch_number":{"type":"integer"}
    }
  }
}

POST earthquakes/_bulk
{"index":{"_id":1}}
{"name":"111","level":1,"magnitude_type":"small","batch_number":22}
{"index":{"_id":2}}
{"name":"222","level":2,"magnitude_type":"big"}

PUT _ingest/pipeline/my_pipeline_002
{
  "processors": [
    {
      "uppercase": {
        "field": "magnitude_type"
      }
    },
    {
  "script": {
    "lang": "painless",
    "source": "if(!ctx.containsKey(\"batch_number\")) {ctx.batch_number = 1} else {ctx.batch_number+=1}"
    }
  }
  ]
}

POST earthquakes/_update_by_query?pipeline=my_pipeline_002

GET earthquakes/_search
```


## 2.6 为索引中的文档增加一个新的字段，字段值为 现有字段1+现有字段2+现有字段3
【铭毅天下 elastic.blog.csdn.net 答案】
```
DELETE test_index
PUT test_index
{
  "mappings":{
    "properties":{
      "value01":{"type":"integer"},
      "value02":{"type":"integer"},
      "value03":{"type":"integer"}
    }
  }
}

POST test_index/_bulk
{"index":{"_id":1}}
{"value01":1,"value02":2,"value03":3}
{"index":{"_id":2}}
{"value01":3,"value02":2,"value03":3}
{"index":{"_id":3}}
{"value01":5,"value02":2,"value03":3}


PUT _ingest/pipeline/newadd_pipeline
{
  "processors": [
    {
      "script": {
        "lang": "painless",
        "source": "ctx.newadd = (ctx.value01 + ctx.value02 + ctx.value03)",
        "params": {
          "param_c": 10
        }
      }
    }
  ]
}

POST test_index/_update_by_query?pipeline=newadd_pipeline

POST test_index/_search
```

# 3、查询篇
## 3.1 写一个查询，要求某个关键字在文档的 4 个字段中至少包含两个以上
 
- bool 查询，should / minimum_should_match
【铭毅天下 elastic.blog.csdn.net 答案】
```
GET search_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "cont1": "铭毅天下"
          }
        },
        {
          "match_phrase": {
            "cont2": "铭毅天下"
          }
        },
        {
          "match_phrase": {
            "cont3": "铭毅天下"
          }
        },
        {
          "match_phrase": {
            "cont4": "铭毅天下"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```
## 3.2 按照要求写一个 search template
- 写入 search template
- 根据 search template 写出相应的 query
【铭毅天下 elastic.blog.csdn.net 答案】

```
GET _search/template
{
    "source" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "cont2",
        "my_value" : "铭毅天下",
        "my_size" : 5
    }
}

GET _render/template
{
    "source" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "cont2",
        "my_value" : "铭毅天下",
        "my_size" : 5
    }
}

```

## 3.3 对一个文档的多个字段进行查询，要求最终的算分是几个字段上算分的总和，同时要求对特定字段设置 boosting 值
【铭毅天下 elastic.blog.csdn.net 答案】
```
GET search_index/_search
{
  "query": {
    "multi_match": {
      "type":"most_fields", 
      "query": "铭毅天下",
      "fields": ["cont2^3", "cont3"]
    }
  }
}
```
## 3.4 针对一个索引进行查询，当索引的文档中存在对象数组时，会搜索到了不期望的数据。需要重新定义 mapping，并提供改写后的 query 语句
 
- Nested Object
【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT my_index
{
  "mappings": {
    "properties": {
      "user": {
        "type": "nested" 
      }
    }
  }
}

PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "铭毅",
      "last" :  "天下"
    },
    {
      "first" : "铭毅",
      "last" :  "elastic"
    }
  ]
}

GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "铭毅" }},
            { "match": { "user.last":  "elastic" }} 
          ]
        }
      }
    }
  }
}

GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "铭毅" }},
            { "match": { "user.last":  "elastic" }} 
          ]
        }
      },
      "inner_hits": { 
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}
```
# 4、聚合篇
## 4.1 earthquakes索引中包含了过去11个月的地震信息，请通过一句查询，获取以下信息
 
- 过去11个月，每个月的平均 地震等级（magiitude）
- 过去11个月里，平均地震等级最高的一个月及其平均地震等级
- 搜索不能返回任何文档
【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT earthquakes_ext
{
  "mappings": {
    "properties": {
      "pt":{"type":"date"},
      "magiitude":{"type":"integer"}
    }
  }
}

POST earthquakes_ext/_bulk
{"index":{"_id":1}}
{"pt":"2019-01-01T17:00:00", "magiitude":1}
{"index":{"_id":2}}
{"pt":"2019-01-01T20:00:00", "magiitude":3}
{"index":{"_id":3}}
{"pt":"2019-02-01T17:00:00", "magiitude":4}
{"index":{"_id":3}}
{"pt":"2019-02-20T17:00:00", "magiitude":5}
{"index":{"_id":4}}
{"pt":"2019-11-01T17:00:00", "magiitude":7}
{"index":{"_id":5}}
{"pt":"2019-11-01T17:00:00", "magiitude":8}
{"index":{"_id":6}}
{"pt":"2019-11-01T17:00:00", "magiitude":9}

POST earthquakes_ext/_search
{
  "size": 0,
  "aggs": {
    "mag_over_time": {
      "date_histogram": {
        "field": "pt",
        "calendar_interval": "month"
      },
      "aggs": {
        "avg_mag": {
          "avg": {
            "field": "magiitude"
          }
        }
      }
    },
    "max_monthly_mags": {
      "max_bucket": {
        "buckets_path": "mag_over_time>avg_mag"
      }
    }
  }
}
```

## 4.2 Query Fileter Bucket Filter
 
## 4.3 Pipeline Aggregation -> Bucket Filter

# 5、映射与分词篇
 
## 5.1 一篇文档，字段内容包括了 “hello & world”，索引后，要求使用 match_phrase query, 
查询 hello & world 或者 hello and world 都能匹配
【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT my_index_002
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "whitespace"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            "& => and"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}


PUT my_index_002/_bulk
{"index":{"_id":1}}
{"title":"hello & world"}
{"index":{"_id":2}}
{"title":"hello and world"}


POST my_index_002/_search
{
  "query": {
    "match_phrase": {
      "title": "hello & world"
    }
  }
}

POST my_index_002/_search
{
  "query": {
    "match_phrase": {
      "title": "hello and world"
    }
  }
}
```

## 5.2 reindex 索引，同时确保给定的两个查询，都能搜索到相关的文档，并且文档的算分是一样的
 
- match 查询，分别查 “smith's” ，“smiths”
 
- 在不改变字段的属性，将数据索引到新的索引上
 
- 确保两个查询有一致的搜索结果和算分
【铭毅天下 elastic.blog.csdn.net 答案】
```
PUT _template/my_index_template
{
  "index_patterns": [
    "a_index_*"
  ],
  "settings": {
    "number_of_shards": 1,
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "whitespace"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            "' => "
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}

PUT a_index_001/_bulk
{"index":{"_id":1}}
{"title":"smith's"}
{"index":{"_id":2}}
{"title":"smiths"}

POST a_index_001/_analyze
{
  "text": "smith's",
  "analyzer": "my_custom_analyzer"
}
POST a_index_001/_analyze
{
  "text": "smiths",
  "analyzer": "my_custom_analyzer"
}


POST _reindex
{
  "source": {
    "index": "a_index_001"
  },
  "dest": {
    "index": "a_index_002"
  }
}

POST a_index_002/_search
{
  "query": {
    "match_phrase": {
      "title": "smith's"
    }
  }
}

POST a_index_002/_search
{
  "query": {
    "match_phrase": {
      "title": "smiths"
    }
  }
}
```

# 6 集群管理篇
## 6.1 安装并配置 一个 hot & warm 架构的集群
- 三个节点， node 1 为 hot ， node2 为 warm，node 3 为cold
- 三个节点均为 master-eligable 节点
- 新创建的索引，数据写入 hot 节点
- 通过一条命令，将数据从 hot 节点移动到 warm 节点

## 6.2 为两个集群配置跨集群搜索
 
- 两个集群都有 movies 的索引
- 创建跨集群搜索
- 创建一条查询，能够同时查到两个集群上的 movies 数据

【铭毅天下 elastic.blog.csdn.net 答案】

```
#cluster_one设置
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_two": {
          "seeds": [
            "172.17.0.17:9301"
          ]
        }
      }
    }
  }
}

#cluster_two设置
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_one": {
          "seeds": [
            "172.17.0.17:9300"
          ]
        }
      }
    }
  }
}

GET orders/_search

#在9301的集群添加如下数据
POST orders/_doc
{
  "name": "9301_11111"
}

POST orders/_doc
{"name":"9301_22222"}

GET /orders/_search
{
  "query": {
    "match_all": {}
  }
}


#跨集群检索
GET /orders,cluster_two:orders/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 6.3 解决集群变红或者变黄的问题
- 技能1：通过 explain API 查看
- 技能2：shard filtering API，查看 include
- 技能3： 更新一下 routing，确认 replica 可以分配（include 更加多的 rack）
- 解决方案
 （1）为集群配置延迟分配和一个节点上最多几个分片的配置
 （2）设置 Replica 为 0
 （3）删除 dangling index
 （4）使用了错误的 routing node attribute
 
## 6.4 备份一个集群中指定的几个索引
【铭毅天下 elastic.blog.csdn.net 答案】
配置：elasticsearch.yml 添加如下：
path.repo: ["/home/elasticsearch/elasticsearch-7.2.0/backup"]

```
PUT /_snapshot/my_fs_backup
{
    "type": "fs",
    "settings": {
        "location": "/home/elasticsearch/elasticsearch-7.2.0/backup",
        "compress": true
    }
}
GET /_cat/indices

PUT /_snapshot/my_fs_backup/snapshot_1?wait_for_completion=true
{
  "indices": "orders",
  "ignore_unavailable": true,
  "include_global_state": false
}

DELETE orders

POST /_snapshot/my_fs_backup/snapshot_1/_restore

GET orders/_search
```
