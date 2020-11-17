## INDEXING DATA
存储（索引）数据

## GOAL: Create, update and delete indices while satisfying a given set of requirements
目标：按照题目要求创建、更新、删除索引

## REQUIRED SETUP: 
建议docker-compose文件：`1e1k_base_cluster.yml`

按要求初始化集群
1. a running Elasticsearch cluster with at least one node and a Kibana instance,
   1. 初始化一个最少含有1个ES节点1个Kibana节点的ES集群
   2. the cluster has no index with name `hamlet`, 
      1. 保证这个集群里没有叫`hamlet`的索引
   3. the cluster has no template that applies to indices starting by `hamlet`
      1. 保证这个集群里，没有能匹配索引名字以`hamlet`开头的索引模板

删除叫`hamlet`的索引
```bash
DELETE hamlet*
```

寻找并删除能匹配名字以`hamlet`开头的索引模板
```bash
GET /_template/
```

```bash
DELETE _template/hamlet*
```

* [参考链接-delete-index](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-delete-index.html)
  * 页面路径：Indices APIs =》 Delete Index
* [参考链接-index-templates](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)
  * 页面路径：Indices APIs =》 Index Templates

## 第1题，按要求创建索引

1. Create the index `hamlet-raw` with 1 primary shard and 3 replicas
   1. 创建一个叫`hamlet-raw`的索引，它有1个分片和3个副本
2. Add a document to `hamlet-raw`, so that the document
   1. 添加一条数据到`hamlet-raw`，它满足以下要求
   2. has id "1"
      1. id是“1”
   3. has default type
      1. 默认type
   4. has one field named `line` with value "To be, or not to be: that is the question"
      1. 有一个叫`line`的字段，里面内容是"To be, or not to be: that is the question"
3.  Update the document with id "1" by adding a field named `line_number` with value "3.1.64"
    1.  更新这个id是“1”的文档，给它添加一个字段叫`line_number`，字段的值是“3.1.64”

## 第1题，题解

1. 创建索引
    ```bash
    PUT hamlet-raw
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 3
      }
    }
    ```
1. 添加数据
    ```bash
    POST hamlet-raw/_doc/1
    {
      "line": "To be, or not to be: that is the question"
    }
    ```
1. 更新数据，几种方式：
   1. 直接覆盖：
      ```bash
      POST hamlet-raw/_doc/1
      {
        "line": "To be, or not to be: that is the question",
        "line_number": "3.1.64"  
      }
      ```
   1. 通过更新接口：
      ```bash
      POST hamlet-raw/_update/1
      {
          "doc" : {
              "line_number": "3.1.64"
          }
      }
      ```
   1. 通过script：
      ```bash
      POST hamlet-raw/_update/1
      {
          "script" : {
              "source": "ctx._source.line_number = params.txt",
              "lang": "painless",
              "params" : {
                  "txt" : "3.1.64"
              }
          }
      }
      ```
    1. 或者简化版的script：
      ```bash
      POST hamlet-raw/_update/1
      {
          "script" : "ctx._source.line_number = '3.1.64'"
      }
      ```

## 第1题，题解说明

* 这题考察索引建立，数据插入及更新
  * 索引建立的时候，如果不指定的话，ES会设为默认分片数（`number_of_shards`）副本数（`number_of_replicas`），但是这题里面对分片和副本数有要求，所以直接在`settings`里面进行相应的设置就好，这里要注意的是这俩值都是复数形式，记得别拼错。
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html)
    2. 页面路径：Indices APIs =》 Create Index
  * 数据的插入和其他数据库类似，题目中指定了id就把id带着，不指定的话ES也会直接自己生成一个出来
    1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-index_.html)
    2. 页面路径：Document APIs =》 Index API
  * 数据的更新有很多方式，最简单直接的就全覆盖，但是需要先查一遍出来，手工merge之后再塞回去，ES也会根据id找到目标文档重新做一次merge，所以一般会比较推荐通过update接口
    1. [参考连接-index](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-index_.html)，[参考链接-update](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update.html)
    2. 页面路径-index：Document APIs =》 Index API
    2. 页面路径-index：Document APIs =》 Update API


# 5、 Add a new document to `hamlet-raw`, so that the document 
# (i) has the id automatically assigned by Elasticsearch, 
# (ii) has default type, 
# (iii) has a field named `text_entry` with value "Whether tis nobler in the mind to suffer", 
# (iv) has a field named `line_number` with value "3.1.66"
# Update the last document by setting the value of `line_number` to 
#    "3.1.65"
# In one request, update all documents in `hamlet-raw` by adding a 
#    new field named `speaker` with value "Hamlet"

POST hamlet-raw/_doc
{
  "text_entry":"Whether tis nobler in the mind to suffer",
  "line_number":"3.1.66"
}

POST hamlet-raw/_update/WSCUnG0BqJrkWJT5Gz0g
{
  "doc" : {
  "line_number":"3.1.65"
  }
}
GET hamlet-raw/_doc/WSCUnG0BqJrkWJT5Gz0g

#method 1
PUT _ingest/pipeline/speaker-Hamlet
{
  "description" : "speaker-Hamlet",
  "processors" : [ {
      "set" : {
        "field": "speaker",
        "value": "Hamlet"
      }
  } ]
}
POST hamlet-raw/_update_by_query?pipeline=speaker-Hamlet

GET hamlet-raw/_search

# method 2
POST hamlet-raw/_update_by_query
{
  "script": {
    "source" : "ctx._source.speaker_method2 = 'Hamlet'",
    "lang": "painless"
  },
  "query": {
   "match_all":{}
  }
}
GET hamlet-raw/_search

# 6
# Update the document with id "1" by renaming the field `line` into 
#    `text_entry`

PUT _ingest/pipeline/rename_pipeline
{
  "processors": [
    {
      "rename": {
        "field": "line",
        "target_field": "text_entry"
      }
    }
  ]
}

POST hamlet-raw/_update_by_query?pipeline=rename_pipeline
{
  "query":{
    "term":{
      "_id":1
    }
  }
}


GET hamlet-raw/_search


# 7
# Create the index `hamlet` and add some documents by running the 
#    following _bulk commandPUT 
 PUT hamlet/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green, and that it us befitted"}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"1.3.1","speaker":"LAERTES","text_entry":"My necessaries are embarkd: farewell:"}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"1.3.4","speaker":"LAERTES","text_entry":"But let me hear from you."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"1.3.5","speaker":"OPHELIA","text_entry":"Do you doubt that?"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
{"index":{"_index":"hamlet","_id":10}}
{"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
{"index":{"_index":"hamlet","_id":11}}
{"line_number":"1.5.2","speaker":"Ghost","text_entry":"Mark me."}
{"index":{"_index":"hamlet","_id":12}}
{"line_number":"1.5.3","speaker":"HAMLET","text_entry":"I will."}

# Create a script named `set_is_hamlet` and save it into the cluster 
#    state. The script (i) adds a field named `is_hamlet` to each 
#    document, (ii) sets the field to "true" if the document has 
#    `speaker` equals to "HAMLET", (iii) sets the field to "false" 
#    otherwise
# Update all documents in `hamlet` by running the `set_is_hamlet` 
#    script


GET hamlet-raw/_search

GET _scripts/set_is_hamlet
DELETE _scripts/set_is_hamlet

# this is ok
POST hamlet-raw/_update_by_query
{
   "script": {
    "lang": "painless",
    "source": """
     if (ctx._source.speaker == "HAMLET") { 
       ctx._source.is_hamlet = true;
     } else 
     {
       ctx._source.is_hamlet = false;
     } 
"""
  }
}


POST hamlet-raw/_doc
{
  "speaker":"3333"
}

GET hamlet-raw/_search

PUT _ingest/pipeline/set_is_hamlet
{
    "processors": [
      {
        "script": {
          "source": """
            if (ctx.speaker_method2 == 'Hamlet') { ctx.is_hamlet = true; } else  { ctx.is_hamlet = false;}
          """
        }
      }
    ]
}

POST hamlet-raw/_update_by_query?pipeline=set_is_hamlet
{
  "query":{
    "match_all":{}
  }
}




# 8 Remove from `hamlet` the documents that have either "KING 
#    CLAUDIUS" or "LAERTES" as the value of `speaker`

POST hamlet/_search

POST hamlet/_delete_by_query
{
  "query": {
    "bool":{
      "should": [
        {
          "term": {
            "speaker.keyword": {
              "value": "KING CLAUDIUS"
            }
          }
        },
         {
          "term": {
            "speaker.keyword": {
              "value": "LAERTES"
            }
          }
        }
      ]
    }
  }
}



