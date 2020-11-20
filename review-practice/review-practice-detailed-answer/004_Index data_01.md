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

## 第2题，按要求添加和更新数据

1. Add a new document to `hamlet-raw`, so that the document
   1. 添加一个新文档到`hamlet-raw`满足以下要求
   2. has the id automatically assigned by Elasticsearch, 
      1. 它的id是ES自动指定的
   3. has default type, 
      1. type是默认的
   4. has a field named `text_entry` with value "Whether tis nobler in the mind to suffer", 
      1. 有一个字段叫`text_entry`，它的值为："Whether tis nobler in the mind to suffer"
   5. has a field named `line_number` with value "3.1.66"
      1. 有一个字段叫`line_number`，值为"3.1.66"
2. Update the last document by setting the value of `line_number` to "3.1.65"
   1. 更新最后一个文档，把`line_number`的值改为"3.1.65"
3. In one request, update all documents in `hamlet-raw` by adding a new field named `speaker` with value "Hamlet"
   1. 在一个请求中，更新所有`hamlet-raw`中的文档，给他们添加一个字段叫`speaker`，字段的值为"Hamlet"

## 第2题，题解

1. 添加数据
    ```bash
    POST hamlet-raw/_doc
    {
      "text_entry": "Whether tis nobler in the mind to suffer",
      "line_number": "3.1.66"
    }
    ```
    
    * 返回值
    ```json
    {
      "_index" : "hamlet-raw",
      "_type" : "_doc",
      "_id" : "Br_w03UBblECTPDi-EA9",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
        "total" : 4,
        "successful" : 1,
        "failed" : 0
      },
      "_seq_no" : 3,
      "_primary_term" : 1
    }
    ```

1. 更新指定数据
    ```bash
    POST hamlet-raw/_update/Br_w03UBblECTPDi-EA9
    {
      "script": "ctx._source.line_number = '3.1.65'"
    }
    ```
    
    * 返回值
    ```json
    {
      "_index" : "hamlet-raw",
      "_type" : "_doc",
      "_id" : "Br_w03UBblECTPDi-EA9",
      "_version" : 2,
      "result" : "updated",
      "_shards" : {
        "total" : 4,
        "successful" : 1,
        "failed" : 0
      },
      "_seq_no" : 4,
      "_primary_term" : 1
    }
    ```

1. 更新全部数据，几种方法
   1. `_update_by_query`接口 + `script`
      ```bash
      POST hamlet-raw/_update_by_query
      {
        "script": "ctx._source.speaker = 'Hamlet'",
        "query":{
          "match_all":{}
        }
      }
      ```
      
      * 返回值
      ```json
      {
        "took" : 10,
        "timed_out" : false,
        "total" : 2,
        "updated" : 2,
        "deleted" : 0,
        "batches" : 1,
        "version_conflicts" : 0,
        "noops" : 0,
        "retries" : {
          "bulk" : 0,
          "search" : 0
        },
        "throttled_millis" : 0,
        "requests_per_second" : -1.0,
        "throttled_until_millis" : 0,
        "failures" : [ ]
      }
      ```

   1. `_update_by_query`接口 + `pipeline`
      * 创建pipeline
      ```bash
      PUT _ingest/pipeline/set_speaker_pipeline
      {
        "description": "upset the speaker field and set it as 'Hamlet'",
        "processors": [
          {
            "set": {
              "field": "speaker",
              "value": "Hamlet1"
            }
          }
        ]
      }
      ```

      * 返回值
      ```json
      {
        "acknowledged" : true
      }
      ```

      * 进行更新
      ```bash
      POST hamlet-raw/_update_by_query?pipeline=set_speaker_pipeline
      ```

      * 返回值
      ```json
      {
        "took" : 11,
        "timed_out" : false,
        "total" : 2,
        "updated" : 2,
        "deleted" : 0,
        "batches" : 1,
        "version_conflicts" : 0,
        "noops" : 0,
        "retries" : {
          "bulk" : 0,
          "search" : 0
        },
        "throttled_millis" : 0,
        "requests_per_second" : -1.0,
        "throttled_until_millis" : 0,
        "failures" : [ ]
      }
      ```

1. 运行以下命令进行校验
    ```bash
    GET hamlet-raw/_search
    ```
    
    * 返回值
    ```json
    {
      "took" : 1,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 2,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "hamlet-raw",
            "_type" : "_doc",
            "_id" : "1",
            "_score" : 1.0,
            "_source" : {
              "line" : "To be, or not to be: that is the question",
              "line_number" : "3.1.65",
              "speaker" : "Hamlet"
            }
          },
          {
            "_index" : "hamlet-raw",
            "_type" : "_doc",
            "_id" : "Br_w03UBblECTPDi-EA9",
            "_score" : 1.0,
            "_source" : {
              "text_entry" : "Whether tis nobler in the mind to suffer",
              "line_number" : "3.1.65",
              "speaker" : "Hamlet"
            }
          }
        ]
      }
    }
    ```

## 第2题，题解说明

* 这题主要考察的是各种数据的更新，从单条到多条
  * 单条的数据更新和上题一样，就不赘述了
  * 多条的数据更新也是多种方式，但是有一点要记得，之前单条数据更新的接口不支持多条数据，因为它是对指定id进行更新的
    * 如果使用这个命令进行通配符匹配更新
      ```bash
      POST hamlet-raw/_update/*
      {
        "script": "ctx._source.line_number = '3.1.65'"
      }
      ```
      会报以下错误
      ```json
      {
        "error" : {
          "root_cause" : [
            {
              "type" : "document_missing_exception",
              "reason" : "[_doc][*]: document missing",
              "index_uuid" : "QLO4rZNgTSKnvPDJuRVLQA",
              "shard" : "0",
              "index" : "hamlet-raw"
            }
          ],
          "type" : "document_missing_exception",
          "reason" : "[_doc][*]: document missing",
          "index_uuid" : "QLO4rZNgTSKnvPDJuRVLQA",
          "shard" : "0",
          "index" : "hamlet-raw"
        },
        "status" : 404
      }
      ```
    1. [参考链接-update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html)，[参考链接-set-processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/set-processor.html)
    2. 页面路径-update-by-query：Document APIs =》 Update By Query API
    3. 页面路径-set-processor：Ingest node =》 Processors =》 Set Processor

## 第3题，按指定要求进行数据更新

1. Update the document with id "1" by renaming the field `line` into `text_entry`
   1. 更新id是“1”的文档，把它的`line`字段更名为`text_entry`

## 第3题，题解

1. 重命名某个字段也是俩操作
   1. 整个文档覆盖
      ```bash
      GET hamlet-raw/_doc/1
      ```
      返回值
      ```json
      {
        "_index" : "hamlet-raw",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 8,
        "_seq_no" : 13,
        "_primary_term" : 1,
        "found" : true,
        "_source" : {
          "line" : "To be, or not to be: that is the question",
          "line_number" : "3.1.65",
          "speaker" : "Hamlet"
        }
      }
      ```
      数据覆盖
      ```bash
      POST hamlet-raw/_doc/1
      {
        "text_entry" : "To be, or not to be: that is the question",
        "line_number" : "3.1.65",
        "speaker" : "Hamlet"
      }
      ```
   1. 通过`_update_by_query` 配合 `pipeline`
      * 创建pipeline
      ```bash
      PUT _ingest/pipeline/rename_pipeline
      {
        "description": "raname the field 'line' into 'text_entry'",
        "processors": [
          {
            "rename": {
              "field": "line",
              "target_field": "text_entry"
            }
          }
        ]
      }
      ```

      * 返回值
      ```json
      {
        "acknowledged" : true
      }
      ```

      * 调用 `_update_by_query`接口
      ```bash
      POST hamlet-raw/_update_by_query?pipeline=rename_pipeline
      {
        "query": {
          "term": {
            "_id": "1"
          }
        }
      }
      ```

      * 返回值
      ```json
      {
        "took" : 15,
        "timed_out" : false,
        "total" : 1,
        "updated" : 1,
        "deleted" : 0,
        "batches" : 1,
        "version_conflicts" : 0,
        "noops" : 0,
        "retries" : {
          "bulk" : 0,
          "search" : 0
        },
        "throttled_millis" : 0,
        "requests_per_second" : -1.0,
        "throttled_until_millis" : 0,
        "failures" : [ ]
      }
      ```

1. 调用接口 `GET hamlet-raw/_doc/1`校验操作结果
    ```json
    {
      "_index" : "hamlet-raw",
      "_type" : "_doc",
      "_id" : "1",
      "_version" : 11,
      "_seq_no" : 17,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "line_number" : "3.1.65",
        "text_entry" : "To be, or not to be: that is the question",
        "speaker" : "Hamlet"
      }
    }
    ```

## 第3题，题解说明

* 这题和上题类似，主要考察`_update_by_query`接口的使用，以及`pipeline`的使用
  1. [参考链接-update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html)，[参考链接-rename-processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/rename-processor.html)
  2. 页面路径-update-by-query：Document APIs =》 Update By Query API
  3. 页面路径-rename-processor：Ingest node =》 Processors =》 Rename Processor

## 第4题，通过script批量更新数据

1. Create the index `hamlet` and add some documents by running the following _bulk command
   1. 创建一个叫`hamlet`的索引，用下面的`_bulk`命令插一些数据进去
      ```bash
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
      ```
2. Create a script named `set_is_hamlet` and save it into the cluster state. The script:
   1. 在集群里创建一个叫`set_is_hamlet`的script，这个script有以下一些功能
   2. adds a field named `is_hamlet` to each document
      1. 给每个文档添加一个叫`is_hamlet`的字段
   3. sets the field to "true" if the document has `speaker` equals to "HAMLET"
      1. 如果文档的`speaker`的值是“HAMLET”那给`is_hamlet`设为“true”
   4. sets the field to "false" otherwise
      1. 否则设为“false”
3. Update all documents in `hamlet` by running the `set_is_hamlet` script
   1. 运行更新命令来处理`hamlet`这个索引，并通过`set_is_hamlet`这个script来更新所有文档

## 第4题，题解

1. 创建索引+插数据参考之前的章节，这里不再赘述
1. 创建一个具有上述功能的script对数据进行处理，两个方法
   1. 直接通过script做`_update_by_query`
      ```bash
      POST hamlet/_update_by_query
      {
        "script": {
          "source": "ctx._source.is_hamlet = (ctx._source.speaker == 'HAMLET')",
          "lang": "painless"
        }
      }
      ```
      
   1. 通过script pipeline进行处理
      * 创建pipeline
      ```bash
      POST _ingest/pipeline/set_is_hamlet_pipeline
      {
        "description": "set the field 'is_hamlet' in doc, the value of it is whether the 'speaker' is 'HAMLET'",
        "processors": [
          {
            "script": {
              "lang": "painless",
              "source": "ctx.is_hamlet = (ctx.speaker == 'HAMLET')"
            }
          }
        ]
      }
      ```

      * 返回值
      ```json
      {
        "acknowledged" : true
      }
      ```

      * 调用`_update_by_query`
      ```bash
      POST _update_by_query?pipeline=set_is_hamlet_pipeline
      ```

      * 返回值
      ```json
      {
        "took" : 44,
        "timed_out" : false,
        "total" : 13,
        "updated" : 13,
        "deleted" : 0,
        "batches" : 1,
        "version_conflicts" : 0,
        "noops" : 0,
        "retries" : {
          "bulk" : 0,
          "search" : 0
        },
        "throttled_millis" : 0,
        "requests_per_second" : -1.0,
        "throttled_until_millis" : 0,
        "failures" : [ ]
      }
      ```
1. 调用 `POST hamlet/_search` 进行校验，结果略

## 第4题，题解说明

* 本题主要考察的是`_update_by_query`配合script进行数据的操作，需要注意的是script取值的时候在pipeline和直接写在query语句里面的写法是不一样的
  * 直接写在`_update_by_query`请求体里的取值是 `ctx._source.is_hamlet`，这里的`ctx`代表整个文档，`_source`代表文档中的数据部分
  * 写在`pipeline`里面的取值是 `ctx.is_hamlet`，这里的`ctx`直接代表文档中的数据部分
  1. [参考链接-script-processor](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/script-processor.html)，[参考链接-update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html)
  2. 页面路径-script-procesor：Ingest node =》 Processors =》 Script Processor
  3. 页面路径-update-by-query：Document APIs =》 Update By Query API

## 第5题，按要求删除数据

1. Remove from `hamlet` the documents that have either "KING CLAUDIUS" or "LAERTES" as the value of `speaker`
   1. 从`hamlet`里删除`speaker`字段为 "KING CLAUDIUS" 或 "LAERTES" 的数据

## 第5题，题解

1. 删除数据
    ```bash
    POST hamlet/_delete_by_query
    {
      "query": {
        "terms": {
          "speaker.keyword": [
            "KING CLAUDIUS",
            "LAERTES"
          ]
        }
      }
    }
    ```

    * 返回值
    ```json
    {
      "took" : 50,
      "timed_out" : false,
      "total" : 4,
      "deleted" : 4,
      "batches" : 1,
      "version_conflicts" : 0,
      "noops" : 0,
      "retries" : {
        "bulk" : 0,
        "search" : 0
      },
      "throttled_millis" : 0,
      "requests_per_second" : -1.0,
      "throttled_until_millis" : 0,
      "failures" : [ ]
    }
    ```
2. 用以下命令校验数据
    ```bash
    POST hamlet/_search
    {
      "query": {
        "terms": {
          "speaker.keyword": [
            "KING CLAUDIUS",
            "LAERTES"
          ]
        }
      }
    }
    ```

    * 返回值
    ```json
    {
      "took" : 0,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 0,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      }
    }
    ```

## 第5题，题解说明

* 这题主要考察`_update_by_query`的兄弟`_delete_by_query`，他们都需要通过query来限制处理的数据集，然后再对这个数据集里的数据进行后续的操作
  * 这里的query语句有很多写法，只要能满足匹配到 `speaker` 的值为 "KING CLAUDIUS" 或 "LAERTES" 的都可以，比如
    ```bash
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
    ```
  1. [参考链接-delete-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-delete-by-query.html)，[参考链接-terms-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-terms-query.html)，[参考链接-bool-query](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html)
  2. 页面路径-delete-by-query：Document APIs =》 Delete By Query API
  3. 页面路径-terms-query：Query DSL =》 Term-level queries =》 Terms
  4. 页面路径-bool-query：Query DSL =》 Compound queries =》 Boolean