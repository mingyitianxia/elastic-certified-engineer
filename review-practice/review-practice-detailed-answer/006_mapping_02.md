
## MAPPINGS AND TEXT ANALYSIS
索引和文档的分析（分词）

## GOAL: Model relational data
目标：规整带关系的数据模型

## REQUIRED SETUP:
初始化步骤
建议docker-compose文件：`1e1k_base_cluster.yml`
1. a running Elasticsearch cluster with at least one node and a Kibana instance,
   1. 运行一个至少有1个节点的ES集群，以及1个kibana节点
2. the cluster has no index with name `hamlet`, 
   1. 保证这个集群里没有叫`hamlet`的索引
3. the cluster has no template that applies to indices starting by `hamlet
   1. 保证这个集群里没有能匹配以`hamlet`开头的索引模板
    ```bash
    DELETE hamlet_*
    DELETE _template/hamlet_*
    ```

## 第1题，对象（object）型数据

1. Create the index `hamlet_1` with one primary shard and no replicas
   1. 创建一个包含1分片0副本的索引`hamlet_1`
2. Add some documents to `hamlet_1` by running the following command 
   1. 用下面的命令给`hamlet_`插入一些数据
3. Verify that the items of the `relationship` array cannot be searched independently - e.g., searching for a friend named Gertrude will return 1 hit
   1. 校验一下`relationship`字段数组里的元素不能被独立搜索，比如搜索`"name": "Gertrude"`而且`"type": "friend"`的数据有一个返回
    ```bash
    PUT hamlet_1/_doc/_bulk
    {"index":{"_index":"hamlet_1","_id":"C0"}}
    {"name":"HAMLET","relationship":[{"name":"HORATIO","type":"friend"},{"name":"GERTRUDE","type":"mother"}]}
    {"index":{"_index":"hamlet_1","_id":"C1"}}
    {"name":"KING CLAUDIUS","relationship":[{"name":"HAMLET","type":"nephew"}]}
    ```

## 第1题，题解

1. 创建索引
    ```bash
    PUT hamlet_1
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    ```
1. 插数据，运行上面的命令，过程略。数据结构：`GET hamlet_1`
    ```json
    {
      "hamlet_1" : {
        "aliases" : { },
        "mappings" : {
          "properties" : {
            "name" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "relationship" : {
              "properties" : {
                "name" : {
                  "type" : "text",
                  "fields" : {
                    "keyword" : {
                      "type" : "keyword",
                      "ignore_above" : 256
                    }
                  }
                },
                "type" : {
                  "type" : "text",
                  "fields" : {
                    "keyword" : {
                      "type" : "keyword",
                      "ignore_above" : 256
                    }
                  }
                }
              }
            }
          }
        },
        "settings" : {
          "index" : {
            "creation_date" : "1606270886689",
            "number_of_shards" : "1",
            "number_of_replicas" : "0",
            "uuid" : "BaWwDy_eSaKPaynt8rWW3g",
            "version" : {
              "created" : "7020199"
            },
            "provided_name" : "hamlet_1"
          }
        }
      }
    }
    ```
2. 校验数据
    ```bash
    POST hamlet_1/_search
    {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "relationship.type": "friend"
              }
            },
            {
              "match": {
                "relationship.name": "Gertrude"
              }
            }
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
          "value" : 1,
          "relation" : "eq"
        },
        "max_score" : 1.2199391,
        "hits" : [
          {
            "_index" : "hamlet_1",
            "_type" : "_doc",
            "_id" : "C0",
            "_score" : 1.2199391,
            "_source" : {
              "name" : "HAMLET",
              "relationship" : [
                {
                  "name" : "HORATIO",
                  "type" : "friend"
                },
                {
                  "name" : "GERTRUDE",
                  "type" : "mother"
                }
              ]
            }
          }
        ]
      }
    }
    ```
## 第1题，题解说明

* 这题主要考察object型的数据，对ES来说所有的字段都支持数组，所以`relationship`这个数组里可以保存多个object型的数据。
  * 在没指定数据结构的时候，ES会尝试按数据的结构匹配合理的索引结构，像`relationship`这种带嵌套结构的数据会默认被解析成object型的数据
  * object型的数据是一个类似 map 结构的数据，可以通过里面的key进行检索，但是它和nested型数据的区别在于，列表中的所有对象会被当作一个整体来搜索，而nested型数据的每个对象中的字段可以分别进行搜索
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/object.html)
  2. 页面路径：Mapping =》 Field datatypes =》 Object

## 第2题，嵌套（nested）型数据

1. Create the index `hamlet_2` with one primary shard and no replicas
   1. 创建一个含有1分片0副本的索引`hamlet_2`
2. Define a mapping for the default type "_doc" of `hamlet_2`, so that the inner objects of the `relationship` field
   1. 给`hamlet_2`的type是默认的"_doc"，同时它的字段需要满足以下条件
   2. can be searched independently, 
      1. 字段可以被独立搜索
   3. have only unanalyzed fields
      1. 只有没分词的字段
3. Reindex `hamlet_1` to `hamlet_2`
      1. 把`hamlet_1` reindex 到 `hamlet_2`里面
4. Verify that the items of the `relationship` array can now be searched independently - e.g., searching for a friend named Gertrude will return no hits
   1. 校验一下`relationship`数组里的元素可以被独立搜索，比如，搜索`"type": "friend"` 而且 `"name":"Gertrude"`的数据没有返回

## 第2题，题解

1. 创建索引
    ```bash
    PUT hamlet_2
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      },
      "mappings": {
        "properties": {
          "relationship": {
            "type": "nested"
          }
        }
      }
    }
    ```
1. reindex
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
1. 校验数据
   1. 直接请求
      ```bash
      POST hamlet_2/_search
      {
        "query": {
          "bool": {
            "must": [
              {
                "match": {
                  "relationship.type": "friend"
                }
              },
              {
                "match": {
                  "relationship.name": "Gertrude"
                }
              }
            ]
          }
        }
      }
      ```
       * 返回值
       ```json
       {
         "took" : 7,
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

   2. 嵌套检索
      ```bash
      POST hamlet_2/_search
      {
        "query": {
          "nested": {
            "path": "relationship",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "relationship.type": "friend"
                    }
                  },
                  {
                    "match": {
                      "relationship.name": "Gertrude"
                    }
                  }
                ]
              }
            }
          }
        }
      }
      ```

      * 返回值
      ```json
      {
        "took" : 178,
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

## 第2题，题解说明

* 这题主要考察嵌套（`nested`）类型数据，它和对象（`object`）型数据的区别在于`nested`型数据可以通过指定路径（`path`）的方式对指定层/位置的数据进行分别的检索
  1. [参考链接-nested-datatype](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html)
  2. 页面路径：Mapping =》 Field datatypes =》 Nested

## 第3题，父子文档（`parent-join`)

1. Add more documents to `hamlet_2` by running the following command
   1. 用下面命令给`hamlet_2`多塞点数据
    ```bash
    POST _bulk
    {"index":{"_index":"hamlet_2", "_id":"LO"}}
    {"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
    {"index":{"_index":"hamlet_2","_id":"L1"}}
    {"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
    {"index":{"_index":"hamlet_2","_id":"L2"}}
    {"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
    ```
2. Create the index `hamlet_3` with only one primary shard and no replicas
   1. 创建一个1分片0副本的索引`hamlet_3`
3. Copy the mapping of `hamlet_2` into `hamlet_3`, but also add a join field to define a relation between a `character` (the parent) and a `line` (the child). The name of such field is "character_or_line" 
   1. 把`hamlet_2`的索引结构拷贝到`hamlet_3`里，同时添加一个名叫`character_or_line`的join字段来描述`character`（父文档）和`line`（子文档）的关系，
4. Reindex `hamlet_2` to `hamlet_3`
   1. 把`hamlet_2` reindex 到 `hamlet_3`里面
5. Create a script named `init_lines` and save it into the cluster state. The script:
   1. has a parameter named `characterId`,  
   2. adds the field `character_or_line` to the document,  
   3. sets the value of `character_or_line.name` to "line" ,  
   4. sets the value of `character_or_line.parent` to the value of the `characterId` parameter
6. Update the document with id `C0` (i.e., the character document of Hamlet) by adding the field `character_or_line` and setting its `character_or_line.name` value to "character" 
7. Update the documents in `hamlet_3` that have "HAMLET" as a `speaker`, by running the `init_lines` script with `characterId` set to "C0"

## 第3题，题解

1. 添加数据，略。
1. 创建索引
    ```bash
    PUT hamlet_3
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      },
      "mappings": {
        "properties": {
          "character_or_line": {
            "type": "join",
            "relations": {
              "character": "line"
            }
          }
        }
      }
    }
    ```
1. reindex
    ```bash
    POST _reindex
    {
      "source": {
        "index": "hamlet_2"
      },
      "dest": {
        "index": "hamlet_3"
      }
    }
    ```
1. 创建script
    ```bash
    PUT _ingest/pipeline/character_update_pipeline
    {
      "description": "set the 'character_or_linne', 'character_or_line.name', 'character_or_line.parent'",
      "processors": [
        {
          "script": {
            "lang": "painless",
            "source": """
              ctx.character_or_line = new HashMap(); 
              ctx.character_or_line.name = "line";
              ctx.character_or_line.parent = params.characterId;
              """,
              "params": {
                "characterId": "C0"
              }
          }
        }
      ]
    }
    ```
1. （由于join field需要routing配置）添加新数据
    ```bash
    POST hamlet_3/_doc/C2?routing=C0
    {
      "line_number": "1.2.1",
      "speaker": "KING CLAUDIUS",
      "text_entry": "Though yet of Hamlet our dear brothers death"
    }
    ```
1. 套用刚才的script定点更新
    ```bash
    POST hamlet_3/_update_by_query?routing=C0&pipeline=character_update_pipeline
    {
      "query":{
        "term":{
          "_id":"C2"
        }
      }
    }
    ```
    1. 这里如果不加`routing`的设置直接进行更新，可能会报这个错：大意是对于父子关联的字段，`routing`是必须存在的。
        ```json
        {
          "took": 10,
          "timed_out": false,
          "total": 1,
          "updated": 0,
          "deleted": 0,
          "batches": 1,
          "version_conflicts": 0,
          "noops": 0,
          "retries": {
            "bulk": 0,
            "search": 0
          },
          "throttled_millis": 0,
          "requests_per_second": -1,
          "throttled_until_millis": 0,
          "failures": [
            {
              "index": "hamlet_3",
              "type": "_doc",
              "id": "C2",
              "cause": {
                "type": "mapper_parsing_exception",
                "reason": "failed to parse",
                "caused_by": {
                  "type": "illegal_argument_exception",
                  "reason": "[routing] is missing for join field [character_or_line]"
                }
              },
              "status": 400
            }
          ]
        }
        ```
2. 校验数据：`GET hamlet_3/_doc/C2`
    * 返回值
    ```json
    {
      "_index" : "hamlet_3",
      "_type" : "_doc",
      "_id" : "C2",
      "_version" : 4,
      "_seq_no" : 5,
      "_primary_term" : 1,
      "_routing" : "C0",
      "found" : true,
      "_source" : {
        "character_or_line" : {
          "parent" : "C0",
          "name" : "line"
        },
        "line_number" : "1.2.1",
        "text_entry" : "Though yet of Hamlet our dear brothers death",
        "speaker" : "KING CLAUDIUS"
      }
    }
    ```

## 第3题，题解说明

* 这题主要考察的是父子关联数据（`parent join`），`reindex`和`_update_by_query`
  * 关联数据可以代替部分关系型数据库的联表查询，但是毕竟是文档型数据存储，ES这部分的处理做的有些差强人意。
  * 在校验结果的部分主要关注的是原始文档里不存在`character_or_line`和`_routing`字段，在处理完之后会添上
  * `reindex`和`_update_by_query`其他章节已经讲过，这里略。
  1. [参考链接](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/parent-join.html)
  2. 页面路径：Mapping =》 Field datatypes =》 Join

## 第3题，拓展
@老杨 还提供了另一种题解方式，但是会存在一些问题，比如子文档需要指定`routing`，但是用 `script` 做 `_update_by_query` 的时候又不能直接更新这个属性。

1. 创建script
    ```bash
    POST _scripts/character_update_script
    {
      "script": {
        "lang": "painless",
        "source": """
          Map map = new HashMap();
          map.name = "line";
          map.parent = params.characterId;
          ctx._source.character_or_line = map;
        """
      }
    }
    ```
1. 创建指定routing用的pipeline
    ```bash
    PUT _ingest/pipeline/set_routing
    {
      "description": "assign the routing attribute for doc",
      "processors": [
        {
          "script": {
            "lang": "painless",
            "source": "ctx._routing = 'C0'"
          }
        }
      ]
    }
    ```
1. 对文档进行定点更新
    ```bash
    POST hamlet_3/_update_by_query?pipeline=set_routing
    {
      "query":{
        "term":{
          "_id":"C2"
        }
      },
      "script": {
        "id": "character_update_script",
        "params": {
          "characterId": "C0"
        }
      }
    }
    ```
1. 校验数据同上，略。
