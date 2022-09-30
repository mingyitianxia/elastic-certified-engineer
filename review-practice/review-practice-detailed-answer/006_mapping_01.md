## Mapping & analysis
索引和分析（数据）

## GOAL: set the mapping and analyzer on data index against requirements
目标：按要求创建索引
建议docker-compose文件：`1e1k_base_cluster.yml`

## 第1题，按要求创建索引
1. Create the index `hamlet_1` with one primary shard and no replicas
   1. 创建一个叫`hamlet_1`的具有1分片0副本的索引
1. Define a mapping for the default type "_doc" of `hamlet_1`, so that
   1. 给默认type `_doc`设置索引，满足以下要求
   2. the type has three fields, named `speaker`, `line_number`, and `text_entry`, 
      1. 它有三个字段，分别叫`speaker`, `line_number` 和 `text_entry`
   3. `speaker` and `line_number` are unanalysed strings
      1. `speaker` 和 `line_number` 是不可分词的string型字段
1. Furthermore, I want you to disable aggregations on the “line_number” field, as I couldn’t think of any valuable statistic that we could get out of unique, progressive line numbers. Note that by disabling aggregations, you are going to save some resources.
   1. 下一步，我希望你能禁止对`line_number`字段的聚合操作，因为我不觉得任何从`line_number`上可以得到任何有价值的统计结果。但是注意，你在禁用聚合功能的同时，还得要能保留一些信息在这字段里。
1. Update the mapping of `hamlet_1` by disabling aggregations on `line_number`
   1. 更新`hamlet_1`的索引使得`line_number`的聚合功能被禁用
2. Update the mapping of `hamlet_` by ignore the items longer then 100 characters on `speaker`
   1. 更新`hamlet_1`使得`speaker`字段的索引忽略100个字符以外的部分
3. Add some documents to `hamlet_1` by running the following `_bulk` command
   1. 用下面的`_bulk`命令插入一些数据进去
      ```bash
      PUT hamlet_1/_doc/_bulk
      {"index":{"_index":"hamlet_1","_id":0}}
      {"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
      {"index":{"_index":"hamlet_1","_id":1}}
      {"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
      {"index":{"_index":"hamlet_1","_id":2}}
      {"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
      {"index":{"_index":"hamlet_1","_id":3}}
      {"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet     our dear brothers death"}
      {"index":{"_index":"hamlet_1","_id":4}}
      {"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green,    and that it us befitted"}
      ```

## 第1题，题解
1. 创建索引
   1. 清理数据`DELETE hamlet_1`
   2. 创建索引
      ```bash
      PUT hamlet_1
      {
        "settings": {
          "number_of_shards": 1,
          "number_of_replicas": 0
        }, 
        "mappings": {
          "properties": {
            "speaker": {
              "type": "keyword"
            },
            "line_number": {
              "type": "keyword"
            },
            "text_entry": {
              "type": "text"
            }
          }
        }
      }
      ```
1. 禁用聚合功能但是保留部分信息
   1. 重建索引
      1. 删除已有索引 `DELETE hamlet_1`
      2. 新建索引
          ```bash
          PUT hamlet_1
          {
            "settings": {
                "number_of_shards": 1,
                "number_of_replicas": 0
            },
            "mappings": {
                "properties": {
                  "text_entry": {
                      "type": "text"
                  },
                  "speaker": {
                      "type": "keyword"
                  },
                  "line_number": {
                      "type": "keyword",
                      "doc_values": false
                  }
                }
            }
          }
          ```
1. 忽略100个字符以外的部分
    ```bash
    PUT hamlet_1/mapping
    {
      "properties": {
        "speaker":{
          "type":"keyword",
          "ignore_above": 100
        }
      }
    }
    ```
1. 校验结果：`GET hamlet_1`
   * 返回值
   ```json
   {
     "hamlet_1" : {
       "aliases" : { },
       "mappings" : {
         "properties" : {
           "line_number" : {
             "type" : "keyword"
           },
           "speaker" : {
             "type" : "keyword",
             "ignore_above" : 100
           },
           "text_entry" : {
             "type" : "text"
           }
         }
       },
       "settings" : {
         "index" : {
           "creation_date" : "1606197654019",
           "number_of_shards" : "1",
           "number_of_replicas" : "0",
           "uuid" : "eRRLhWGESpe6td967QoJlA",
           "version" : {
             "created" : "7020199"
           },
           "provided_name" : "hamlet_1"
         }
       }
     }
   }
   ```
1. 通过`_bulk`接口插数据前几章讲过了，这里略

## 第1题，题解说明

* 本题主要考察数据索引的部分，包括了
  * `doc_values`，这个参数主要用于保存数据的元数据，用来支持对数据的排序、聚合以及script的操作
  * `ignore_abrove`，这个参数主要用于`keyword`型数据索引时，对xx字符以外的数据忽略（不索引），以免因为过长的数据带来的过多的存储消耗。（对于`keyword`型数据，在匹配/操作时多半是需要完全匹配的，过长的内容大概率是不会被命中的）
  * 索引`mapping`更新接口，用来修改已经存在的索引，添加、修改里面的字段参数等
  1. [参考链接-doc_values](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/doc-values.html)，[参考链接-ignore_abrove](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ignore-above.html)，[参考链接-indices-put-mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-put-mapping.html)
  2. 页面路径-doc_values： Mapping =》 Mapping parameters =》 doc_values
  3. 页面路径-ignore_abrove： Mapping =》 Mapping parameters =》 ignore_abrove
  4. 页面路径-indices-put-mapping： Indices APIs =》 Put Mapping

## 第2题，按要求设置索引

1. Create the index `hamlet_2` with one primary shard and no replicas 
   1. 创建一个有1个分片0个副本的索引`hamlet_2`
2. Copy the mapping of `hamlet_1` into `hamlet_2`, but also define a multi-field for `speaker`. The name of such multi-field is `tokens` and its data type is the (default) analysed string
   1. 把`hamlet_1`的索引结构赋值到`hamlet_2`里，但是给字段`speaker`设置成多合一字段，其中一个取名为`tokens`，并且保证它是（默认）分词的字符串字段
3. Reindex `hamlet_1` to `hamlet_2`
   1. 把`hamlet_1` reindex 到 `hamlet_2`里
4. Verify that full-text queries on "speaker.tokens" are enabled on `hamlet_2` by running the following command:
   1. 用下面的命令校验一下基于`hamlet_2`的`speaker.tokens`字段的全文检索语句有效（原文这里是搜"hamlet"但是由于第1题里通过`_bulk`接口插入的数据中，不存在`speaker`里包含"hamlet"的数据，所以改成了“KING”）
      ```bash
      GET hamlet_2/_search 
      {
        "query": {
          "match": { "speaker.tokens": "KING" }
      }}
      ```

## 第2题，题解

1. 创建索引
    ```bash
    PUT hamlet_2
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    ```
1. 把`hamlet_1`的索引结构拷过来，并给`speaker`字段做多合一字段设置
    ```bash
    PUT hamlet_2/_mapping
    {
      "properties": {
        "line_number" : {
          "type" : "keyword",
          "doc_values": false
        },
        "speaker" : {
          "type" : "keyword",
          "ignore_above" : 100,
          "fields": {
            "tokens": {
              "type": "text"
            }
          }
        },
        "text_entry" : {
          "type" : "text"
        }
      }
    }
    ```
1. 校验索引结构：`GET hamlet_2`
      * 返回值
      ```json
      {
        "hamlet_2" : {
          "aliases" : { },
          "mappings" : {
            "properties" : {
              "line_number" : {
                "type" : "keyword",
                "doc_values" : false
              },
              "speaker" : {
                "type" : "keyword",
                "fields" : {
                  "tokens" : {
                    "type" : "text"
                  }
                },
                "ignore_above" : 100
              },
              "text_entry" : {
                "type" : "text"
              }
            }
          },
          "settings" : {
            "index" : {
              "creation_date" : "1606219439262",
              "number_of_shards" : "1",
              "number_of_replicas" : "0",
              "uuid" : "YSlqYQ4uR8u5UqV37da53A",
              "version" : {
                "created" : "7020199"
              },
              "provided_name" : "hamlet_2"
            }
          }
        }
      }
      ```
1. 把`hamlet_1` reindex 到 `hamlet_2`里面
    ```bash
    POST _reindex
    {
      "source": {
        "index": "hamlet_1"
      },
      "dest": {
        "index": "hamlet_2"
      }
    }
    ```
    * 返回值
    ```json
    {
      "took" : 24,
      "timed_out" : false,
      "total" : 0,
      "updated" : 0,
      "created" : 0,
      "deleted" : 0,
      "batches" : 0,
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
1. 校验数据：`GET hamlet_2/_search`
      * 返回值
      ```json
      {
        "took" : 3,
        "timed_out" : false,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 5,
            "relation" : "eq"
          },
          "max_score" : 1.0,
          "hits" : [
            {
              "_index" : "hamlet_2",
              "_type" : "_doc",
              "_id" : "0",
              "_score" : 1.0,
              "_source" : {
                "line_number" : "1.1.1",
                "speaker" : "BERNARDO",
                "text_entry" : "Whos there?"
              }
            },
            {
              "_index" : "hamlet_2",
              "_type" : "_doc",
              "_id" : "1",
              "_score" : 1.0,
              "_source" : {
                "line_number" : "1.1.2",
                "speaker" : "FRANCISCO",
                "text_entry" : "Nay, answer me: stand, and unfold yourself."
              }
            },
            {
              "_index" : "hamlet_2",
              "_type" : "_doc",
              "_id" : "2",
              "_score" : 1.0,
              "_source" : {
                "line_number" : "1.1.3",
                "speaker" : "BERNARDO",
                "text_entry" : "Long live the king!"
              }
            },
            {
              "_index" : "hamlet_2",
              "_type" : "_doc",
              "_id" : "3",
              "_score" : 1.0,
              "_source" : {
                "line_number" : "1.2.1",
                "speaker" : "KING CLAUDIUS",
                "text_entry" : "Though yet of Hamlet     our dear brothers death"
              }
            },
            {
              "_index" : "hamlet_2",
              "_type" : "_doc",
              "_id" : "4",
              "_score" : 1.0,
              "_source" : {
                "line_number" : "1.2.2",
                "speaker" : "KING CLAUDIUS",
                "text_entry" : "The memory be green,    and that it us befitted"
              }
            }
          ]
        }
      }
      ```
1. 测试全文搜索语句
    ```bash
    GET hamlet_2/_search 
    {
      "query": {
        "match": { 
          "speaker.tokens": "KING"
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
          "value" : 2,
          "relation" : "eq"
        },
        "max_score" : 0.74487394,
        "hits" : [
          {
            "_index" : "hamlet_2",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 0.74487394,
            "_source" : {
              "line_number" : "1.2.1",
              "speaker" : "KING CLAUDIUS",
              "text_entry" : "Though yet of Hamlet     our dear brothers death"
            }
          },
          {
            "_index" : "hamlet_2",
            "_type" : "_doc",
            "_id" : "4",
            "_score" : 0.74487394,
            "_source" : {
              "line_number" : "1.2.2",
              "speaker" : "KING CLAUDIUS",
              "text_entry" : "The memory be green,    and that it us befitted"
            }
          }
        ]
      }
    }
    ```

## 第2题，题解说明

* 这题主要考察的是多合一字段（`multi-field`），`reindex`接口
  * 对于多合一字段，它的不同子字段里都可以单独访问/检索，同时，不指定子字段而直接对这种字段进行搜索时，算分默认按照所有子字段打分的总和进行计算
  * `reindex`接口其他章节已经讲过了，可以通过一些策略对集群内的索引进行数据的复制
  1. [参考链接-multi-field](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/multi-fields.html)，[参考链接-reindex](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html)
  2. 页面路径-multi-field：Mapping =》 Mapping parameters =》 fields
  3. 页面路径-reindex：Document APIs =》 Reindex API
